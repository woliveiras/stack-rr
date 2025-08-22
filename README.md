# Stack*RR

> Fast way for running for Radarr, Lidarr, Readarr, and Bazarr

This repository contains a Docker Compose setup for managing various media applications, including Radarr, Lidarr, Readarr, and Bazarr. The setup is designed to run on a Linux server with Docker and Docker Compose installed.

In this guide, you’ll learn how to set up a fully automated media server that, when you add a movie or TV show, will:

- Search for torrents/magnet links using Jackett/Prowlarr
- Manage content with Sonarr (TV shows), Radarr (movies), Readarr (books), and Lidarr (music)
- Download subtitles with Bazarr
- Download files via qBittorrent
- Stream your media to your TV with Plex or Jellyfin

All you need is my `docker-compose.yml` file and a few environment variables!

## Prerequisites

- [Docker and Docker Compose](https://docs.docker.com/compose/install/#docker-desktop-recommended) installed on your server;

## Setup Instructions

1. **Clone the Repository**: Clone this repository to your server using the following command:

   ```bash
   git clone git@github.com:woliveiras/stack-rr.git
   ```

   If you prefer using HTTPS, use this command instead:

   ```bash
   git clone https://github.com/woliveiras/stack-rr.git
   ```

1. **Navigate to the Directory**: Change into the cloned directory:

   ```bash
   cd stack-rr
   ```

1. **Configure Environment Variables**: You need to set up your environment variables. Change the name of the file `.env.example` to `.env`. Set the following variables:

   ```env
   ARRPATH=/arr/      # Path to your media library, keep the final slash
                      # Example: ARRPATH=/path/to/your/media/ 
                      # /Users/<User>/Downloads/RADARR/
   TZ=Europe/Madrid   # Set your timezone https://www.timeanddate.com/time/map/

   # You don't need to change the following ones unless you have a specific reason
   PUID=1000
   PGID=1000
   UMASK=002
   ```

## Running the Stack

1. **Start the Stack**: Use Docker Compose to start the stack. Run the following command in the directory where your `docker-compose.yml` file is located:

   ```bash
   docker-compose up -d
   ```

   This command will pull the necessary images and start the containers in detached mode.

   The first time you run this command, it may take a while to download all the images and set up the containers.

2. **Access the Applications**: Once the containers are running, you can access the applications via your web browser:

   - Radarr: `http://<your-server-ip>:7878`
   - Lidarr: `http://<your-server-ip>:8686`
   - Readarr: `http://<your-server-ip>:8787`
   - Bazarr: `http://<your-server-ip>:6767`

   Ex.: Radarr: `http://localhost:7878/`, qBittorrent: `http://localhost:8081/`, Jellyfin: `http://localhost:8096/`

3. **Stopping the Stack**: To stop the stack, run:

   ```bash
   docker-compose down
   ```

## Initial Configuration of the *arr Suite

After accessing each application for the first time, you will need to go through the initial setup process. This typically involves setting up your media library paths, configuring download clients, and adding indexers.

###  qBittorrent

Access `http://<YOUR_SERVER>:8081` (default user: `admin`, password: run `docker logs qbittorrent` in your terminal).

You can change the credentials in **Settings** > **Web UI** > **Authentication** and click Save.

Set the download path to `/downloads` in Tools > Options > Downloads and click Save.

You don't need to paste magnet links in qBittorrent. The *arr applications will do that for you.

### Radarr, Lidarr, Readarr

Access `http://<YOUR_SERVER>:7878` (Radarr) or `8989` (Sonarr).

Go to **Settings** > **Media Management**, click on **Add Root Folder** and add /data/movies for Radarr or /data/tvshows for Sonarr.

Go to **Download Clients** and click on the plus button (+): add *qBittorrent* (host: `qbittorrent`, port: `8081`, your username and password).

Copy the API Key from **General** > **Security** > API Key.

### Prowlarr

Access `http://<YOUR_SERVER>:9696`

Go to **Settings** > **Download Clients** and click on the plus button (+): add *qBittorrent* (host: `qbittorrent`, port: `8081`, your username and password).

Go to **Settings** > **Apps**: add Radarr, Lidarr, Readarr and Sonarr using their URLs and API Keys you copied earlier. In the Prowlarr Server, use `http://Prowlarr:9696`, and in the Radarr/Lidarr/Readarr/Sonarr Server, use `http://Lidarr:8686`, `http://Radarr:7878`, `http://Readarr:8787` or `http://Sonarr:8989`.

### Bazarr

Access `http://<YOUR_SERVER>:6767`

Go to **Settings** > **Providers**: add the subtitle providers you want to use.

Go to **Settings** > **Radarr** and add Radarr (address: `radarr`, port: `7878`, base URL: `/`, your API Key).

Go to **Settings** > **Sonarr** and add Radarr (address: `sonarr`, port: `8989`, base URL: `/`, your API Key).

For Multi-language Search: In Settings > Profiles, edit profiles and change Language to `<Language You Want>` to search for `<Language>` content.

### Jellyfin

Access `http://<YOUR_SERVER>:8096`

Go through the initial setup process, creating an admin user and adding your media libraries.

You can install the Jellyfin app on your smart TV, phone, or tablet to stream your media. When asked for the server address, use `http://<YOUR_SERVER>:8096`. The username and password are the ones you created during the initial setup.

## VPN Setup (Optional)

For Global VPN: Go to the `docker-compose.yml` file and add the following lines:

```yaml
  gluetun:
    image: ghcr.io/qdm12/gluetun
    container_name: gluetun
    restart: always
    cap_add:
      - NET_ADMIN
    devices:
      - /dev/net/tun:/dev/net/tun
    # ports:
    #   - 8888:8888/tcp # HTTP proxy
    #   - 8388:8388/tcp # Shadowsocks
    #   - 8388:8388/udp # Shadowsocks
    volumes:
      - ./data:/gluetun
      - ./wireguard.conf:/gluetun/wireguard/wg0.conf
    environment:
      # See https://github.com/qdm12/gluetun-wiki/tree/main/setup#setup
      - VPN_SERVICE_PROVIDER=ivpn
      - VPN_TYPE=openvpn
      # OpenVPN:
      - OPENVPN_USER=
      - OPENVPN_PASSWORD=
      # Wireguard:
      # - WIREGUARD_PRIVATE_KEY=wOEI9rqqbDwnN8/Bpp22sVz48T71vJ4fYmFWujulwUU=
      # - WIREGUARD_ADDRESSES=10.64.222.21/32
      # Timezone for accurate log times
      - TZ=
      # Server list updater
      # See https://github.com/qdm12/gluetun-wiki/blob/main/setup/servers.md#update-the-vpn-servers-list
      - UPDATER_PERIOD=
```

## Demo: Downloading Public Domain Content

In Radarr, try downloading public domain movies:

Click on **Add New Movie** and search for `Night of the Living Dead (1968)`.

Watch as it changes to Downloading and appears in qBittorrent.

Repeat with `Nosferatu (1922)` and `The City of the Dead (1960)`.

In Jellyfin/Plex, you’ll see the movies with covers, synopsis, and actors ready to play.

You can find more public domain movies at [JustWatch](https://www.justwatch.com/us/provider/public-domain-movies?sort_by=trending_7_day).

For public domain books, you can use [goodreads](https://www.goodreads.com/shelf/show/public-domain).
