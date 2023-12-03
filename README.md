# docker-arrs-with-nordvpn
This is a Docker Stack to deploy Sonarr, Radarr, Jackett, and qBittorrent. With Jacket and qBittorrent connecting to NordVPN.

All apps connect to a shared volume that connects to a NAS SMB share using CIFS.

## Sources/Referances
- ~~NordVPN: https://github.com/bubuntux/nordvpn~~
- NordLynx: https://github.com/bubuntux/nordlynx
  - https://github.com/bubuntux/nordlynx/issues/39
  - https://github.com/bubuntux/nordlynx/discussions/28
  - https://forum.openwrt.org/t/instruction-config-nordvpn-wireguard-nordlynx-on-openwrt/89976
  - https://sleeplessbeastie.eu/2019/02/18/how-to-use-public-nordvpn-api/
- qBittorrent: https://hub.docker.com/r/linuxserver/qbittorrent
- ~~Jackett: https://hub.docker.com/r/linuxserver/jackett/~~
- Prowlarr: https://wiki.servarr.com/prowlarr
- Sonarr: https://hub.docker.com/r/linuxserver/sonarr
- Radarr: https://hub.docker.com/r/linuxserver/radarr
- Servarr Wiki: https://wiki.servarr.com/
- autoheal: https://github.com/willfarrell/docker-autoheal
  - https://github.com/bubuntux/nordlynx/issues/78#issuecomment-1133950952

## Docker Environment Variables:
| Variable | Notes |
| ----------- | ----------- |
| nordlynx_pkey | Generated via NordVPN Container |
| nasUser | The service username on NAS host |
| nasPass | The service password on NAS host |
| nasMediaPath | Network path to your SMB share Example: //192.168.1.18/Media |
| timezone | Whatever you timezone is, Example: America/New_York |

## Docker Compose File
```yaml:docker-compose.yml
version: "3.4"
networks:
  arr-network:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: "172.30.172.0/24"
    enable_ipv6: false
services:
  nordlynx:
    image: ghcr.io/bubuntux/nordlynx
    container_name: nordlynx
    cap_add:
      - NET_ADMIN
      - SYS_MODULE  # maybe; May resolve curl ipv6 issue where host cannot resolve
    environment:
      - PRIVATE_KEY=$nordlynx_pkey    # required
      - QUERY=filters\[country_id\]=228&filters\[servers_groups\]\[identifier\]=legacy_p2p&filters\[servers_technologies\]\[identifier\]=wireguard_udp #https://api.nordvpn.com/v1/servers/recommendations?filters\[country_id\]=228&filters\[servers_groups\]\[identifier\]=legacy_p2p&filters\[servers_technologies\]\[identifier\]=wireguard_udp
      - NET_LOCAL=192.168.1.0/24      # required; Set your LAN subnet mask
      - DNS=1.1.1.1,1.0.0.1,8.8.8.8
      - RECONNECT=3600                # Optional, force reconnect after 3600s (1hr) 
      - TZ=$timezone
    ports:
      - 9080:9080     # qbittorrent Web GUI
      # - 9117:9117   # jackett Web GUI
      - 9696:9696     # prowlarr Web GUI
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1   # maybe; May resolve curl ipv6 issue where host cannot resolve
      - net.ipv4.conf.all.rp_filter=2        # maybe; set reverse path filter to loose mode; May resolve curl ipv6 issue where host cannot resolve
      - net.ipv6.conf.all.disable_ipv6=1     # disable ipv6; recommended if using ipv4 only
    networks:
      - arr-network
    restart: always
    labels:
      - autoheal=true   # Required for willfarrell/docker-autoheal
  qbittorrent:
    image: ghcr.io/linuxserver/qbittorrent
    container_name: qbittorrent
    cap_add:
      - NET_ADMIN
    network_mode: service:nordlynx
    depends_on:
      nordlynx:
        condition: service_started
    environment:
      - PUID=1000
      - PGID=997
      - TZ=$timezone
      - WEBUI_PORT=9080
      - DOCKER_MODS=arafatamim/linuxserver-io-mod-vuetorrent
    volumes:
      - qbit_config:/config
      - media:/media
    restart: always
    labels:
      - autoheal=true    # Required for willfarrell/docker-autoheal
    healthcheck:    # Required for willfarrell/docker-autoheal
      test: curl --fail --ipv4 https://google.com || exit 1
      interval: 30s
      timeout: 5s
      retries: 3
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    cap_add:
      - NET_ADMIN
    network_mode: service:nordlynx
    depends_on:
      nordlynx:
        condition: service_started
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=$timezone
    volumes:
      - prowlarr_config:/config
    restart: always
    labels:
      - autoheal=true    # Required for willfarrell/docker-autoheal
    healthcheck:    # Required for willfarrell/docker-autoheal
      test: curl --fail --ipv4 https://google.com || exit 1
      interval: 10s
      timeout: 5s
      retries: 3
  sonarr:
    image: ghcr.io/linuxserver/sonarr
    container_name: sonarr
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1000
      - PGID=997
      - TZ=$timezone
    volumes:
      - sonarr_config:/config
      - media:/media
    ports:
      - 8989:8989
    networks:
      - arr-network
    restart: always
    labels:
      - autoheal=true    # Required for willfarrell/docker-autoheal
    healthcheck:    # Required for willfarrell/docker-autoheal
      test: curl --fail --ipv4 https://google.com || exit 1
      interval: 60s
      timeout: 5s
      retries: 3
  radarr:
    image: ghcr.io/linuxserver/radarr
    container_name: radarr
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1000
      - PGID=997
      - TZ=$timezone
    volumes:
      - radarr_config:/config
      - media:/media
    ports:
      - 7878:7878
    networks:
      - arr-network
    restart: always
    labels:
      - autoheal=true    # Required for willfarrell/docker-autoheal
    healthcheck:    # Required for willfarrell/docker-autoheal
      test: curl --fail --ipv4 https://google.com || exit 1
      interval: 60s
      timeout: 5s
      retries: 3
  readarr:
    image: lscr.io/linuxserver/readarr:nightly
    container_name: readarr
    cap_add:
      - NET_ADMIN
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=$timezone
    volumes:
      - readarr_config:/config
      - media:/media
    ports:
      - 8787:8787
    networks:
      - arr-network
    restart: always
    labels:
      - autoheal=true    # Required for willfarrell/docker-autoheal
    healthcheck:    # Required for willfarrell/docker-autoheal
      test: curl --fail --ipv4 https://google.com || exit 1
      interval: 60s
      timeout: 5s
      retries: 3
volumes:
  qbit_config:
  prowlarr_config:
  sonarr_config:
  radarr_config:
  readarr_config:
  media:
    driver_opts:
      type: cifs
      o: username=$nasUser,password=$nasPass,file_mode=0777,dir_mode=0777,noperm
      device: $nasMediaPath
```
