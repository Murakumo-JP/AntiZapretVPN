# AntiZapret VPN in Docker

Antizapret created to redirect only blocked domains to VPN tunnel. Its called split tunneling.
This repo is based on idea from original [AntiZapret LXD image](https://bitbucket.org/anticensority/antizapret-vpn-container/src/master/)

# Support and discussions group:
https://t.me/antizapret_support

# How  works?

1) List of blocked domains downloaded from open registry.
2) List parsed and rules for dns resolver (adguardhome) created.
3) Adguardhome resend requests for blocked domains to python script dnsmap.py.
4) Python script:
   a) resolve real address for domain
   b) create fake address from 10.244.0.0/15 subnet
   c) create iptables rule to forward all packets from fake ip to real ip.
5) Fake IP is sent in DNS response to client
6) All vpn tunnels configured with split tunneling. Only traffic to 10.244.0.0/15 subnet is routed through VPN.

# Features

- [openvpn-dco](https://openvpn.net/as-docs/tutorials/tutorial--turn-on-openvpn-dco.html) - a kernel extension for improving performance of OpenVPN
- Multiple VPN transports: Wireguard, OpenVPN, IPsec/XAuth ("Cisco IPsec")
- Adguard as main DNS resolver
- filebrowser as web viewer & editor for `*-custom.txt` files
- Unified dashboard
- Optional built-in reverse proxy based on caddy


# Installation

0. Install [Docker Engine](https://docs.docker.com/engine/install/):
   ```bash
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh
   ```
1. Clone repository and start container:
   ```bash
   git clone https://github.com/Murakumo-JP/AntiZapretVPN.git antizapret
   cd antizapret
   ```
2. Create docker-compose.override.yml with services you need. Minimal example with only wireguard:
```yml
services:
  antizapret:
    environment:
      - ADGUARDHOME_PASSWORD=somestrongpassword
  wireguard:
     environment:
        - WIREGUARD_PASSWORD=somestrongpassword
     extends:
        file: services/wireguard/docker-compose.yml
        service: wireguard
     depends_on:
        - antizapret
```
Find full example in [docker-compose.override.sample.yml](./docker-compose.override.sample.yml)

3. Start services:
```shell
   docker compose pull
   docker compose build
   docker compose up -d
   docker system prune -f
```

## Access admin panels:

### HTTP: 
By default panels have following http ports exposed to internet:
- dashboard: no exposed port
- adguard: 3000
- filebrowser: 2000
- openvpn: 8080
- wireguard: 51821
- wireguard-amnezia: 51831

If you do not wish to expose ports to internet override them in `docker-compose.override.yml`.
In this example adguard and wireguard admin panels are removed from internet, and wireguard udp server is exposed: 
```yml
services:
   antizapret:
      environment:
         - ADGUARDHOME_USERNAME=admin
         - ADGUARDHOME_PASSWORD=password
      ports: !reset []

   wireguard:
      extends:
         file: services/wireguard/docker-compose.yml
         service: wireguard
      environment:
         - WIREGUARD_PASSWORD=password
      ports: !override
         - 51820:51820/udp
```

### HTTPS
To enable https server and create self-signed certificates - add `proxy` container to `docker-compose.override.yml`
When `proxy` container is started, access services with https at following ports at your host ip:
- dashboard: 443
- adguard: 1443
- filebrowser: 2443
- openvpn: 3443
- wireguard: 4443
- wireguard-amnezia: 5443

`proxy` container is optional.

### Local network
   When you connected to VPN, you can access containers without exposing ports to internet:
- http://core.antizapret:3000
- http://dashboard.antizapret:80
- http://wireguard-amnezia.antizapret:51821
- http://wireguard.antizapret:51821
- http://openvpn-ui.antizapret:8080
- http://filebrowser.antizapret:80

## Update

```shell
git pull
docker compose pull
docker compose build
docker compose down --remove-orphans && docker compose up -d --remove-orphans
```

## Reset:
Remove all settings, vpn configs and return initial state of service:
```shell
docker compose down
rm -rf config/*
docker compose up -d
```

# [Documentation](https://github.com/xtrime-ru/antizapret-vpn-docker?tab=readme-ov-file#documentation)

# Credits
- [ProstoVPN](https://antizapret.prostovpn.org) — the original project
- [AntiZapret VPN Container](https://bitbucket.org/anticensority/antizapret-vpn-container/src/master/) — source code of the LXD-based container
- [AntiZapret PAC Generator](https://bitbucket.org/anticensority/antizapret-pac-generator-light/src/master/) — proxy auto-configuration generator to bypass censorship of Russian Federation
- [Amnezia WireGuard VPN](https://github.com/w0rng/amnezia-wg-easy) — used for Amnezia Wireguard integration
- [WireGuard VPN](https://github.com/wg-easy/wg-easy) — used for Wireguard integration
- [OpenVPN](https://github.com/d3vilh/openvpn-ui) - used for OpenVPN integration
- [IPsec VPN](https://github.com/hwdsl2/docker-ipsec-vpn-server) — used for IPsec integration
- [AdGuardHome](https://github.com/AdguardTeam/AdGuardHome) - DNS resolver
- [filebrowser](https://github.com/filebrowser/filebrowser) - web file browser & editor
- [lighttpd](https://github.com/lighttpd/lighttpd1.4) - web server for unified dashboard
- [caddy](https://github.com/caddyserver/caddy) - reverse proxy
- [No Thought Is a Crime](https://ntc.party) — a forum about technical, political and economical aspects of internet censorship in different countries
