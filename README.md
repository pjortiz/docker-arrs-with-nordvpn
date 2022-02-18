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

##Docker Environment Variables:
| Variable | Notes |
| ----------- | ----------- |
| nordlynx_pkey | Generated via NordVPN Container |
| nasUser | The service username on NAS host |
| nasPass | The service password on NAS host |
| LAN | Whitelist your local area network so the apps connect via NordVPN can be accessed. Example: 192.168.1.0/24 |
| smbSharePath | Network path to your SMB share Example: //192.168.1.18/Media |

##Docker Compose File
```yaml:docker-compose.yml
```
