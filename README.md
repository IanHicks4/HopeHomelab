Homelab

Host: OptiPlex Path: /srv/docker

Services:

-caddy
-qbittorrent/gluetun
-Sonarr
-Radarr
-Prowlarr
-Overseerr
-Flaresolverr
-Jellyfin

Domains: 
-jellyfin.kai.coach
-seer.kai.coach

Notes: *arr stack all in one compose file Jellyfin is network_mode host and accessible remotely via Tailscale qBittorrent and gluetun in one compose file Caddy will be used once I move over Actual and Vaultwarden from Pi
