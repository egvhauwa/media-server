# media-server

- https://github.com/AdrienPoupa/docker-compose-nas
- https://github.com/Morzomb/All-jellyfin-media-server

# Prerequisites

### Docker

https://docs.docker.com/engine/install/ubuntu/

Add your user to the Docker group

```
sudo usermod -aG docker $USER
```

# Accessing Applications

Once the applications are deployed, you can access them using the following addresses :

> [!IMPORTANT]  
> Replace `localhost` with the IP address of your machine or remote server if needed.

- Jellyfin : http://localhost:8096
- Jellyseer : http://localhost:5055
- Sonarr : http://localhost:8989
- Radarr : http://localhost:7878
- Prowlarr : http://localhost:9696
- qBittorrent : http://localhost:8080

# Configuration

## Jellyfin

Go to the Jellyfin webui and configure Jellyfin.
Enable `Automatically merge series that are spread across multiple folder` when adding a Shows Library.

### hardware acceleration

1. Download jellyfin-ffmpeg

2. Go to `Dashboard -> Playback -> Transcoding`

- Enable hardware acceleration
- Set the transcoding folder to: /transcode

## qBittorrent

qBittorrent Web UI will generate a temporary password when the container is started. To view this password, check the logs for this container with the command:

```
docker logs qbittorrent
```

To use the VueTorrent WebUI just go to qBittorrent, Options, Web UI, Use Alternative WebUI, and enter `/vuetorrent`.

### options

1. Once logged in, click the gear icon to go to **Options**.
2. Under the **WebUi** tab, update the password.
3. Under the **Downloads** tab, configure the backup settings as follows:
   - **Default Torrent Management Mode**: `Automatic` (required for category-based save paths to work)
   - **When Torrent Category changed**: `Relocate torrent`
   - **When Default Save Path changed**: `Relocate affected torrents`
   - **When Category Save Path changed**: `Relocate affected torrents`
   - **Default Save Path**: `/downloads`
4. Click **SAVE**.

<div style="text-align: center">
    <img src="images/qbit-options.png" style="margin: 15px 10px;">
</div>

### category configuration

1. In the WebUI, expand **CATEGORIES** in the left menu. Right-click on **All** and select **Add category...**.
2. In the **New Category** window, configure as follows:
   - **Category**: `movies` (this corresponds to the category you will later configure in Radarr)
   - **Save path**: `/downloads/movies`
3. Click **Add**.
4. Right-click on **All** again, select **Add category...**.
5. Configure as follows:
   - **Category**: `tvshows` (this should match the category configured later in Sonarr, by default `sonarr-tv`, but this guide uses `tvshows`)
   - **Save path**: `/downloads/tvshows`
6. Click **Add**.

## Radarr

### media management

1. Open the WebUI and go to **Settings** > **Media Management**.
2. Click **Add Root Folder**, add the path `/movies`, and click **OK**.
3. Click **Show Advanced** at the top, scroll down to **Importing**, and make sure **Use Hardlinks instead of Copy** is enabled.

### download clients

1. In the WebUI, go to **Settings** > **Download Clients**.
2. Click **+** under **Download Clients**, then select **qBittorrent** from the **Add Download Client** window.
3. Fill in the fields as follows:
   - **Name**: `qBittorrent` (or another name of your choice)
   - **Host**: `localhost`
   - **Username**: `admin`
   - **Password**: (password chosen in qBittorrent)
   - **Category**: `movies` (this should match the category set in qBittorrent)
4. Click **Test**. If you see a checkmark, it means the connection is working; if not, there is an error.
5. Click **Save**.

_Note: if entering `localhost` as the Host does not work, try entering the IP address instead (ex: `192.168.x.x`)_

## Sonarr

### media management

1. Open the WebUI and go to **Settings** > **Media Management**.
2. Click **Add Root Folder**, add the path `/COMMON_PATH/sonarr/tv`, and click **OK**.
3. Click **Show Advanced**, scroll down to **Importing**, and enable **Use Hardlinks instead of Copy**.

_Note: if entering `qbittorrent` as the Host does not work, try entering the IP address instead (ex: `192.168.x.x`)_

### download clients

1. In the WebUI, go to **Settings** > **Download Clients**.
2. Click **+** under **Download Clients**, then select **qBittorrent**.
3. Fill in the fields as follows:
   - **Name**: `qBittorrent` (or another name of your choice)
   - **Host**: `qbittorrent`
   - **Username**: `admin`
   - **Password**: (password chosen in qBittorrent)
   - **Category**: `tvshows` (this should match the category set in qBittorrent)
4. Click **Test**. If you see a checkmark, it means the connection is working.
5. Click **Save**.

## Prowlarr

### configure torrent indexers

1. Open the WebUI and go to **Indexers** > **Add New Indexer**.
2. Select **1337x** (or another tracker of your choice).
   - You can modify the settings as per your preference, but the default values generally work well.
   - Sorting by **Seeders** can be useful for faster downloads.
3. Click **Test**. If you see a checkmark, the connection is functional; otherwise, there's an error.
4. Click **Save**.

### configure FlareSolverr

1. Go to **Settings** and click **+** under **Indexer**.
2. Select **FlareSolverr** and fill in the information as follows:
   - **Name**: `FlareSolverr`
   - **Tags**: `flaresolverr`
   - **Host**: `http://flaresolverr:8191/`
3. Click **Test** to check the connection.
4. Click **Save**.

### configure Radarr

1. Go to **Settings** and click **Apps**.
2. Select **Radarr** and fill in the information as follows:
   - **Sync Level**: `Full Sync`
   - **Prowlarr Server**: `http://prowlarr:9696`
   - **Radarr Server**: `http://radarr:7878`
   - **ApiKey**: Find the API key in the Radarr interface under **Settings** > **General** > **API Key**.
3. Click **Test** to check the connection.
4. Click **Save**.

### configure Sonarr

1. Go to **Settings** and click **Apps**.
2. Select **Sonarr** and fill in the information as follows:
   - **Sync Level**: `Full Sync`
   - **Prowlarr Server**: `http://prowlarr:9696`
   - **Sonarr Server**: `http://sonarr:8989`
   - **ApiKey**: Find the API key in the Sonarr interface under **Settings** > **General** > **API Key**.
3. Click **Test** to check the connection.
4. Click **Save**.

## Jellyseerr

### sign in / configuration

1. Open the WebUI and in the **Welcome to Jellyseerr** screen, select **Use your Jellyfin account**.
2. Fill in the information as follows:
   - **Jellyfin URL**: `http://jellyfin:8096/`
   - **Email Address**: `<your email address>`
   - **Username**: `<your Jellyfin username>`
   - **Password**: `<your Jellyfin password>`
3. Select **Sign In**.
4. Go to **Sync Libraries** under **Jellyfin Libraries**, select your Jellyfin libraries, then click **Continue**.

### **Integrating with Radarr**

1. Go to **Radarr Settings**, then click **Add Radarr Server**.
2. Fill in the information as follows:
   - **Default Server**: Check this box
   - **Server Name**: `Radarr`
   - **Name or IP Address**: `http://radarr`
   - **Port**: `7878`
   - **API Key**: Find the API key in the Radarr interface under **Settings** > **General** > **API Key**.
3. Click **Test** to check the connection.
4. Click **Save Changes**.

### **Integrating with Sonarr**

1. Go to **Sonarr Settings**, then click **Add Sonarr Server**.
2. Fill in the information as follows:
   - **Default Server**: Check this box
   - **Server Name**: `Sonarr`
   - **Name or IP Address**: `http://sonarr`
   - **Port**: `8989`
   - **API Key**: Find the API key in the Sonarr interface under **Settings** > **General** > **API Key**.
3. Click **Test** to check the connection.
4. Click **Save Changes**.

# Possible improvements

- proxmox
- truenas
- tailscale
