# docker-arrs-with-nordvpn
This is a Docker Stack to deploy Sonarr, Radarr, Jackett, and qBittorrent. With Jacket and qBittorrent connecting to NordVPN.

All apps connect to a shared volume that connects to a NAS SMB share using CIFS.

## Sources/Referances
NordVPN: https://github.com/bubuntux/nordvpn#environment-variables
qBittorrent: https://hub.docker.com/r/linuxserver/qbittorrent
Jackett: https://hub.docker.com/r/linuxserver/jackett/
Sonarr: https://hub.docker.com/r/linuxserver/sonarr
Radarr: https://hub.docker.com/r/linuxserver/radarr
Servarr Wiki: https://wiki.servarr.com/

Docker Environment Variables:
| Variable | Notes |
| ----------- | ----------- |
| nordUser | NordVPN username/email |
| nordPass | NordVPN password |
| nasUser | The service username on NAS host |
| nasPass | The service password on NAS host |
