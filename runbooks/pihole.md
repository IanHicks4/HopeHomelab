# 🧾 Raspberry Pi DNS Node (Pi-hole + Unbound Runbook)

## 📌 Overview

**Purpose:**  
Provide network-wide DNS filtering and private recursive DNS resolution.

**Host:** Raspberry Pi  
**Services:**
- Pi-hole (Docker)
- Unbound (systemd)
- Tailscale (remote access)

---

# 🏗️ Architecture


Clients (PC / Phone / TV)
↓
Router (DHCP → DNS = Pi-hole)
↓
Pi-hole (Docker, port 53)
↓
Unbound (127.0.0.1:5335)
↓
Root DNS Servers


---

# 🔧 Prerequisites (Fresh Build)

## Install packages

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io docker-compose-plugin unbound
Enable Docker
sudo systemctl enable docker
sudo systemctl start docker
###Step 1 — Install Tailscale
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

Verify:

tailscale ip
###Step 2 — Configure Unbound
Create config
sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf

Paste:

server:
  interface: 127.0.0.1
  port: 5335

  root-hints: "/etc/unbound/root.hints"
  auto-trust-anchor-file: "/var/lib/unbound/root.key"

  access-control: 127.0.0.0/8 allow
Download root hints
sudo curl -o /etc/unbound/root.hints https://www.internic.net/domain/named.root
Restart Unbound
sudo systemctl restart unbound
Verify Unbound
dig @127.0.0.1 -p 5335 google.com

Expected: NOERROR

###Step 3 — Deploy Pi-hole
Create directory
sudo mkdir -p /srv/docker/pihole/compose
cd /srv/docker/pihole/compose
Create docker-compose.yml
nano docker-compose.yml

Paste:

services:
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    network_mode: host
    restart: unless-stopped

    environment:
      TZ: America/New_York
      WEBPASSWORD: changeme-now
      DNSMASQ_USER: pihole
      FTL_CMD: no-daemon

    volumes:
      - pihole_etc:/etc/pihole
      - pihole_dnsmasq:/etc/dnsmasq.d

volumes:
  pihole_etc:
  pihole_dnsmasq:
Start Pi-hole
sudo docker compose up -d
Verify container
sudo docker ps
###Step 4 — Configure Pi-hole
Access UI
http://<PI_IP>/admin
Set upstream DNS

Settings → DNS

Remove all default providers
Add:
127.0.0.1#5335
Fix listening mode (CRITICAL)
sudo nano /var/lib/docker/volumes/pihole_etc/_data/pihole.toml

Set:

listeningMode = "ALL"

Restart:

sudo docker restart pihole
Verify DNS
dig @127.0.0.1 google.com
dig @<PI_IP> google.com
###Step 5 — Router Configuration

Set DHCP DNS:

Primary DNS: 192.168.1.56

(Optional temporary fallback)

Secondary DNS: 1.1.1.1
##Operations
Update blocklists
sudo docker exec pihole pihole -g
Restart services
sudo docker restart pihole
sudo systemctl restart unbound
Test DNS
dig @192.168.1.56 google.com
##Data Locations
Pi-hole
/var/lib/docker/volumes/pihole_etc/_data
/var/lib/docker/volumes/pihole_dnsmasq/_data
Unbound
/etc/unbound/
##Troubleshooting
No internet on network

Cause: Pi-hole down

Fix:
Set router DNS:
1.1.1.1
Restart Pi-hole:
sudo docker restart pihole
Unbound not resolving
dig @127.0.0.1 -p 5335 google.com

If broken:

sudo systemctl restart unbound
Pi-hole not reachable

Check:

sudo grep listeningMode /var/lib/docker/volumes/pihole_etc/_data/pihole.toml

Fix:

listeningMode = "ALL"
