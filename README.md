# Jellyfin stack with Docker

## Table of Content

<!-- TOC -->
* [Docker Compose NAS](#docker-compose-nas)
  * [Table of Content](#table-of-content)
  * [Applications](#applications)
  * [Quick Start](#quick-start)
  * [Environment Variables](#environment-variables)
  * [PIA WireGuard VPN](#pia-wireguard-vpn)
  * [Sonarr & Radarr](#sonarr--radarr)
    * [File Structure](#file-structure)
    * [Download Client](#download-client)
  * [Prowlarr](#prowlarr)
  * [qBittorrent](#qbittorrent)
  * [Jellyfin](#jellyfin)
  * [Homepage](#homepage)
  * [Optional Services](#optional-services)
    * [FlareSolverr](#flaresolverr)
    * [SABnzbd](#sabnzbd)
    * [AdGuard Home](#adguard-home)
      * [Encryption](#encryption)
      * [DHCP](#dhcp)
  * [Static IP](#static-ip)
  * [Laptop Specific Configuration](#laptop-specific-configuration)
<!-- TOC -->

## Applications

| **Application**                                                      | **Description**                                                                                                                                      | **Image**                                                                                | **URL**      |
|----------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------|--------------|
| [Sonarr](https://sonarr.tv)                                          | PVR for newsgroup and bittorrent users                                                                                                               | [linuxserver/sonarr](https://hub.docker.com/r/linuxserver/sonarr)                        | /sonarr      |
| [Radarr](https://radarr.video)                                       | Movie collection manager for Usenet and BitTorrent users                                                                                             | [linuxserver/radarr](https://hub.docker.com/r/linuxserver/radarr)                        | /radarr      |
| [Prowlarr](https://github.com/Prowlarr/Prowlarr)                     | Indexer aggregator for Sonarr and Radarr                                                                                                             | [linuxserver/prowlarr:latest](https://hub.docker.com/r/linuxserver/prowlarr)             | /prowlarr    |
| [PIA WireGuard VPN](https://github.com/thrnz/docker-wireguard-pia)   | Encapsulate qBittorrent traffic in [PIA](https://www.privateinternetaccess.com/) using [WireGuard](https://www.wireguard.com/) with port forwarding. | [thrnz/docker-wireguard-pia](https://hub.docker.com/r/thrnz/docker-wireguard-pia)        |              |
| [qBittorrent](https://www.qbittorrent.org)                           | Bittorrent client with a complete web UI<br/>Uses VPN network<br/>Using Libtorrent 1.x                                                               | [linuxserver/qbittorrent:libtorrentv1](https://hub.docker.com/r/linuxserver/qbittorrent) | /qbittorrent |
| [Jellyfin](https://jellyfin.org)                                     | Media server designed to organize, manage, and share digital media files to networked devices                                                        | [linuxserver/jellyfin](https://hub.docker.com/r/linuxserver/jellyfin)                    | /jellyfin    |
| [Homepage](https://gethomepage.dev)                                  | Application dashboard                                                                                                                                | [benphelps/homepage](https://github.com/benphelps/homepage/pkgs/container/homepage)      | /            |
| [Watchtower](https://containrrr.dev/watchtower/)                     | Automated Docker images update                                                                                                                       | [containrrr/watchtower](https://hub.docker.com/r/containrrr/watchtower)                  |              |
| [SABnzbd](https://sabnzbd.org/)                                      | Optional - Free and easy binary newsreader                                                                                                           | [linuxserver/sabnzbd](https://hub.docker.com/r/linuxserver/sabnzbd)                      | /sabnzbd     |
| [FlareSolverr](https://github.com/FlareSolverr/FlareSolverr)         | Optional - Proxy server to bypass Cloudflare protection in Prowlarr                                                                                  | [flaresolverr/flaresolverr](https://hub.docker.com/r/flaresolverr/flaresolverr)          |              |
| [AdGuard Home](https://adguard.com/en/adguard-home/overview.html)    | Optional - Network-wide software for blocking ads & tracking                                                                                         | [adguard/adguardhome](https://hub.docker.com/r/adguard/adguardhome)                      |              |
| [DHCP Relay](https://github.com/modem7/DHCP-Relay)                   | Optional - Docker DHCP Relay                                                                                                                         | [modem7/dhcprelay](https://hub.docker.com/r/modem7/dhcprelay)                            |              |

Optional containers are not run by default, they need to be enabled, 
see [Optional Services](#optional-services) for more information.

## Quick Start

`cp .env.example .env`, edit to your needs then `sudo docker compose up -d`.

For the first time, run `./update-config.sh` to update the applications base URLs and set the API keys in `.env`.

If you want to show Jellyfin information in the homepage, create it in Jellyfin settings and fill `JELLYFIN_API_KEY`.

WARNING: this repo is customized for my personal use and could not be suited for your needs. Use the original source of this project for wider compatibility.

## Environment Variables

| Variable                       | Description                                                                                                                                                                                            | Default                                          |
|--------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------|
| `COMPOSE_FILE`                 | Docker compose files to load                                                                                                                                                                           | `docker-compose.yml`                             |
| `COMPOSE_PATH_SEPARATOR`       | Path separator between compose files to load                                                                                                                                                           | `:`                                              |
| `USER_ID`                      | ID of the user to use in Docker containers                                                                                                                                                             | `1000`                                           |
| `GROUP_ID`                     | ID of the user group to use in Docker containers                                                                                                                                                       | `1000`                                           |
| `TIMEZONE`                     | TimeZone used by the container.                                                                                                                                                                        | `Europe/Vilnius`                                 |
| `DATA_ROOT`                    | Host location of the data files                                                                                                                                                                        | `/mnt/data`                                      |
| `DOWNLOAD_ROOT`                | Host download location for qBittorrent, should be a subfolder of `DATA_ROOT`                                                                                                                           | `/mnt/data/torrents`                             |
| `PIA_LOCATION`                 | Servers to use for PIA                                                                                                                                                                                 | `pl` (Poland)                                    |
| `PIA_USER`                     | PIA username                                                                                                                                                                                           |                                                  |
| `PIA_PASS`                     | PIA password                                                                                                                                                                                           |                                                  |
| `PIA_LOCAL_NETWORK`            | PIA local network                                                                                                                                                                                      | `192.168.0.0/16`                                 |
| `HOSTNAME`                     | Hostname of the NAS, could be a local IP or a domain name                                                                                                                                              | `localhost`                                      |
| `ADGUARD_HOSTNAME`             | Optional - AdGuard Home hostname used, if enabled                                                                                                                                                      |                                                  |
| `ADGUARD_USERNAME`             | Optional - AdGuard Home username to show details in the homepage, if enabled                                                                                                                           |                                                  |
| `ADGUARD_PASSWORD`             | Optional - AdGuard Home password to show details in the homepage, if enabled                                                                                                                           |                                                  |
| `DNS_CHALLENGE`                | Enable/Disable DNS01 challenge, set to `false` to disable.                                                                                                                                             | `false`                                          |
| `DNS_CHALLENGE_PROVIDER`       | Provider for DNS01 challenge, [see list here](https://doc.traefik.io/traefik/https/acme/#providers).                                                                                                   | `cloudflare`                                     |
| `LETS_ENCRYPT_CA_SERVER`       | Let's Encrypt CA Server used to generate certificates, set to production by default.<br/>Set to `https://acme-staging-v02.api.letsencrypt.org/directory` to test your changes with the staging server. | `https://acme-v02.api.letsencrypt.org/directory` |
| `LETS_ENCRYPT_EMAIL`           | E-mail address used to send expiration notifications                                                                                                                                                   |                                                  |
| `CLOUDFLARE_EMAIL`             | CloudFlare Account email                                                                                                                                                                               |                                                  |
| `CLOUDFLARE_DNS_API_TOKEN`     | API token with `DNS:Edit` permission                                                                                                                                                                   |                                                  |
| `CLOUDFLARE_ZONE_API_TOKEN`    | API token with `Zone:Read` permission                                                                                                                                                                  |                                                  |
| `SONARR_API_KEY`               | Sonarr API key to show information in the homepage                                                                                                                                                     |                                                  |
| `RADARR_API_KEY`               | Radarr API key to show information in the homepage                                                                                                                                                     |                                                  |
| `PROWLARR_API_KEY`             | Prowlarr API key to show information in the homepage                                                                                                                                                   |                                                  |
| `JELLYFIN_API_KEY`             | Jellyfin API key to show information in the homepage                                                                                                                                                   |                                                  |
| `HOMEPAGE_VAR_TITLE`           | Title of the homepage                                                                                                                                                                                  | `Fuji server landing page`                       |
| `HOMEPAGE_VAR_SEARCH_PROVIDER` | Homepage search provider, [see list here](https://gethomepage.dev/en/widgets/search/)                                                                                                                  | `google`                                         |
| `HOMEPAGE_VAR_HEADER_STYLE`    | Homepage header style, [see list here](https://gethomepage.dev/en/configs/settings/#header-style)                                                                                                      | `boxed`                                          |
| `HOMEPAGE_VAR_WEATHER_CITY`    | Homepage weather city name                                                                                                                                                                             |                                                  |
| `HOMEPAGE_VAR_WEATHER_LAT`     | Homepage weather city latitude                                                                                                                                                                         |                                                  |
| `HOMEPAGE_VAR_WEATHER_LONG`    | Homepage weather city longitude                                                                                                                                                                        |                                                  |
| `HOMEPAGE_VAR_WEATHER_UNIT`    | Homepage weather unit, either `metric` or `imperial`                                                                                                                                                   | `metric`                                         |

## PIA WireGuard VPN

This example uses PrivateInternetAccess VPN.

Fill `.env` and fill it with your PIA credentials.

The location of the server it will connect to is set by `LOC=pl`, defaulting to Poland.

You need to fill the credentials in the `PIA_*` environment variable, 
otherwise the VPN container will exit and qBittorrent will not start.

## Sonarr & Radarr

### File Structure

Sonarr and Radarr must be configured to support hardlinks, to allow instant moves and prevent using twice the storage
(Bittorrent downloads and final file). The trick is to use a single volume shared by the Bittorrent client and the *arrs.
Subfolders are used to separate the TV shows from the movies.

The configuration is well explained by [this guide](https://trash-guides.info/Hardlinks/How-to-setup-for/Docker/).

In summary, the final structure of the shared volume will be as follows:

```
data
├── torrents = shared folder qBittorrent downloads
│  ├── movies = movies downloads tagged by Radarr
│  └── tv = movies downloads tagged by Sonarr
└── media = shared folder for Sonarr and Radarr files
   ├── movies = Radarr
   └── tv = Sonarr
```

Go to Settings > Management.
In Sonarr, set the Root folder to `/data/media/tv`.
In Radar, set the Root folder to `/data/media/movies`.

### Download Client

Then qBittorrent can be configured at Settings > Download Clients. Because all the networking for qBittorrent takes
place in the VPN container, the hostname for qBittorrent is the hostname of the VPN container, ie `vpn`, and the port is `8080`:

## Prowlarr

The indexers are configured through Prowlarr. They synchronize automatically to Radarr and Sonarr.

Radarr and Sonarr may then be added via Settings > Apps. The Prowlarr server is `http://prowlarr:9696/prowlarr`, the Radarr server
is `http://radarr:7878/radarr` and Sonarr `http://sonarr:8989/sonarr`:

Their API keys can be found in Settings > Security > API Key.

## qBittorrent

Set the default save path to `/data/torrents` in Settings, and restrict the network interface to WireGuard (`wg0`).

The web UI login page can be disabled on for the local network in Settings > Web UI > Bypass authentication for clients

```
192.168.0.0/16
127.0.0.0/8
172.17.0.0/16
```

## Jellyfin

To enable [hardware transcoding](https://jellyfin.org/docs/general/administration/hardware-acceleration/),
depending on your system, you may need to update the following block:

```    
devices:
  - /dev/dri/renderD128:/dev/dri/renderD128
  - /dev/dri/card0:/dev/dri/card0
```

Generally, running Docker on Linux you will want to use VA-API, but the exact mount paths may differ depending on your
hardware.

## Homepage

The homepage comes with sensible defaults; some settings can ben controlled via environment variables in `.env`.

If you to customize further, you can modify the files in `/homepage/*.yaml` according to the [documentation](https://gethomepage.dev). 
Due to how the Docker socket is configured for the Docker integration, files must be edited as root.

The files in `/homepage/tpl/*.yaml` only serve as a base to set up the homepage configuration on first run.

## Optional Services

As their name would suggest, optional services are not launched by default. They have their own `docker-compose.yml` file
in their subfolders. To enable a service, append it to the `COMPOSE_FILE` environment variable.

Say you want to enable FlareSolverr, you should have `COMPOSE_FILE=docker-compose.yml:flaresolverr/docker-compose.yml`

### FlareSolverr

In Prowlarr, add the FlareSolverr indexer with the URL http://flaresolverr:8191/

### SABnzbd

Enable SABnzbd by setting `COMPOSE_FILE=docker-compose.yml:sabnzbd/docker-compose.yml`. It will be accessible at `/sabnzbd`.

If that is not the case, the `url_base` parameter in `sabnzbd.ini` should be set to `/sabnzbd`.

### AdGuard Home

Set the `ADGUARD_HOSTNAME`, I chose a different subdomain to use secure DNS without the folder.

On first run, specify the port 3000 and enable listen on all interfaces to make it work with Tailscale.

If after running `docker compose up -d`, you're getting `network docker-compose-nas declared as external, but could not be found`,
run `docker network create docker-compose-nas` first.

#### Encryption

In Settings > Encryption Settings, set the certificates path to `/opt/adguardhome/certs/certs/<YOUR_HOSTNAME>.crt`
and the private key to `/opt/adguardhome/certs/private/<YOUR_HOSTNAME>.key`, those files are created by Traefik cert dumper
from the ACME certificates Traefik generates in JSON.

#### DHCP

If you want to use the AdGuard Home DHCP server, for example because your router does not allow changing its DNS server,
you will need to select the `eth0` DHCP interface matching `10.0.0.10`, then specify the 
Gateway IP to match your router address (`192.168.0.1` for example) and set a range of IP addresses assigned to local
devices.

In `adguardhome/docker-compose.yml`, set the network interface `dhcp-relay` should listen to. By default, it is set to
`enp2s0`, but you may need to change it to your host's network interface, verify it with `ip a`.

In the configuration (`adguardhome/conf/AdGuardHome.yaml`), set the DHCP options 6th key to your NAS internal IP address:
```yml
dhcp:
  dhcpv4:
    options:
      - 6 ips 192.168.0.10,192.168.0.10
```

## Static IP

Set a static IP, assuming `192.168.0.10` and using Google DNS servers: `sudo nano /etc/netplan/00-installer-config.yaml`

```yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp2s0:
      dhcp4: no
      addresses:
        - 192.168.0.10/24
      gateway4: 192.168.0.1
      nameservers:
          addresses: [8.8.8.8, 8.8.4.4]
  version: 2
```

Apply the plan: `sudo netplan apply`. You can check the server uses the right IP with `ip a`.

## Laptop Specific Configuration

If the server is installed on a laptop, you may want to disable the suspension when the lid is closed:
`sudo nano /etc/systemd/logind.conf`

Replace:
- `#HandleLidSwitch=suspend` by `HandleLidSwitch=ignore`
- `#LidSwitchIgnoreInhibited=yes` by `LidSwitchIgnoreInhibited=no`

Then restart: `sudo service systemd-logind restart`
