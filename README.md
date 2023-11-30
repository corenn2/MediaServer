

# Set up Automatic Jellyfin Media Server on Ubuntu With few commands via Docker-Compose

## Ports
1. **Jellyfin**:  Port `8096` need to be exposed.

2. **Radarr**: Port `7878` need to be exposed.

3. **Sonarr**:  Port `8989` need to be exposed.

4. **Prowlarr**:  Port `9696` need to be exposed.

5. **qBittorrent**: Port `8080` need to be exposed.

Remember to configure your firewall or any network security groups (especially if you're on AWS) to allow traffic on these ports.



## Update System and Install Docker
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install docker.io -y
sudo apt install docker-compose -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ubuntu
```

## Create Required Directories
```bash
mkdir -p config/jellyfin config/radarr config/sonarr config/prowlarr
mkdir Jellyfin_media
mkdir -p ~/Jellyfin_media/movies ~/Jellyfin_media/tvshows ~/Jellyfin_media/downloads
sudo chown -R 1000:1000 ~/Jellyfin_media
sudo chmod -R 755 ~/Jellyfin_media
```

## Set Up Docker Compose
Create `docker-compose.yml` with your configuration:
```bash
nano docker-compose.yml
```

-The content:
```
version: '3'
services:
  jellyfin:
    image: jellyfin/jellyfin
    container_name: jellyfin
    network_mode: "host"
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - /home/ubuntu/Jellyfin_media:/home/ubuntu/Jellyfin_media
      - ./config/jellyfin:/config
    restart: unless-stopped

  radarr:
    image: linuxserver/radarr
    container_name: radarr
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - ./config/radarr:/config
      - /home/ubuntu/Jellyfin_media/movies:/home/ubuntu/Jellyfin_media/movies
      - /home/ubuntu/Jellyfin_media/downloads:/home/ubuntu/Jellyfin_media/downloads
    ports:
      - 7878:7878
    restart: unless-stopped

  sonarr:
    image: linuxserver/sonarr
    container_name: sonarr
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - ./config/sonarr:/config
      - ~/Jellyfin_media/tvshows:/tv
      - /home/ubuntu/Jellyfin_media/downloads:/home/ubuntu/Jellyfin_media/downloads
    ports:
      - 8989:8989
    restart: unless-stopped

  prowlarr:
    image: linuxserver/prowlarr
    container_name: prowlarr
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - ./config/prowlarr:/config
    ports:
      - 9696:9696
    restart: unless-stopped
```


This Docker Compose file sets up a complete media server stack including Jellyfin, Radarr, Sonarr, and Prowlarr. You can use the default configuration or customize it as needed.

**Dynamic Variables to Consider:**

1. **Volumes**: Adjust the volume paths (`/home/ubuntu/Jellyfin_media`) to match your local directories.
2. **Environment Variables**: `PUID` and `PGID` should match the user ID and group ID of your host system for proper file permissions.
3. **Ports**: Default ports are set (e.g., `7878` for Radarr). Change them if they conflict with other services on your system.

**Default Setup**:

You can use the file as-is if the default configuration suits your environment. Ensure the specified directories exist on your host.

**Starting the Stack**:

Run in the directory with your `docker-compose.yml` to start all services in detached mode.
```
docker-compose up -d
``` 


---


## Install and Configure qBittorrent-nox (if you want to use docker at the button there is diffrent option)
```bash
sudo apt install qbittorrent-nox
qbittorrent-nox
sudo nano /etc/systemd/system/qbittorrent-nox.service
```

Include the following in the qBittorrent-nox service file:
```ini
[Unit]
Description=qBittorrent Command Line Client
After=network.target

[Service]
Type=simple
User=ubuntu
ExecStart=/usr/bin/qbittorrent-nox
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

### Enable and Start qBittorrent-nox Service
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now qbittorrent-nox
```

---





### 1. Configuring Jellyfin

1. **Access Jellyfin**: Go to `http://your-server-ip:8096`.
2. **Initial Setup**:
   - **Create Admin Account**: Create an admin user with a username and password.
   - **Media Libraries**: Set up your media libraries. Point Jellyfin to the media directories (`~/Jellyfin_media/movies` for movies, `~/Jellyfin_media/tvshows` for TV shows) that you have set in your Docker compose file.
   - **Transcoding Settings**: Configure transcoding settings based on your server's capabilities.
    - **Configure Subtitles**: Enable automatic subtitle downloading. , Set preferred subtitle languages to English and Hebrew


### 2. Configuring Radarr

1. **Access Radarr**: Go to `http://your-server-ip:7878`.
2. **Initial Setup**:
   - **Media Management**: Set your movie path to `~/Jellyfin_media/movies`. This should match the volume you set in Docker.
   - **Quality Profiles**: Define your desired quality profiles for movies.
   - **Connect DownloadClient**: Connect your torrent downloading client via Radarr UI
### 3. Configuring Sonarr

1. **Access Sonarr**: Go to `http://your-server-ip:8989`.
2. **Initial Setup**:
   - **Media Management**: Set the TV shows path to `/Jellyfin_media/tvshows`.
   - **Quality Profiles**: Set up quality profiles for the type of TV shows you want.
   - **Connect DownloadClient**: Connect your torrent downloading client via Sonarr UI

### 4. Configuring Prowlarr

1. **Access Prowlarr**: Go to `http://your-server-ip:9696`.
2. **Initial Setup**:
   - **Add Indexers and Trackers**: Here, you'll configure the indexers and trackers. Prowlarr supports a variety of indexers and trackers; add those you have access to and plan to use.
   - **Integration with Radarr and Sonarr**: After adding indexers, integrate Prowlarr with Radarr and Sonarr via Radarr UI using API Key and IP which is fetched from both Sonarr and Radarr settings. This allows them to use the indexers configured in Prowlarr efficiently.






## Auto remove downlaoded torrents


### Sonarr or Raddar - Enable Completed Download Handling

1. **Access Sonarr & Radarr**: Open the Sonarr & Radarr web interface.
2. **Navigate to Settings**: Go to `Settings` in the Sonarr & Radarr interface.
3. **Download Client Settings**: Select the `Download Client` tab.
4. **Enable Completed Download Handling**: Make sure that `Completed Download Handling` is enabled. This feature allows Sonarr & Radarr to import completed downloads and then remove the torrent from the download client once it's done.

### qBittorrent - Set Pause Rules 

1. **Access qBittorrent**: Open the qBittorrent web interface.
2. **Go to Options/Settings**: Find the Options or Settings menu.
3. **Configure Removal Rules**:
   - Look for settings related to seeding goals . qBittorrent allows you to set rules for when to Pause seeding. 
   - You can set it to Pause seeding after a certain ratio is reached (e.g., `1.0`) or after a certain amount of time of seeding.
   

Basiclly after you set it up you are all done with this step.





## Add Live channels

Live channels Database : [iptv-org/iptv on GitHub](https://github.com/iptv-org/iptv)

Create an M3U (`example.m3u`) file for IPTV or Use existing URL in Jellyfin involves compiling a playlist with links to various streaming sources. 

- If you are using existing URL for Live Channels skip steps  (`1 , 2 , 3`)

1. **Create a New Text File**: Start by creating a new text file using a text editor (like Notepad).

2. **Format Your M3U File**:
   - Begin the file with `#EXTM3U`.
   - For each stream, first, add `#EXTINF:-1, Channel Name`.
   - On the next line, put the direct URL to the stream.
   - Repeat this format for each channel you want to add.
   - Example:
     ```
     #EXTM3U
     #EXTINF:-1, Channel 1
     http://example.com/stream1.m3u8
     #EXTINF:-1, Channel 2
     http://example.com/stream2.m3u8
     ```

3. **Save the File**: Save the file with an `.m3u` extension.

4. **Add to Jellyfin**:
   - In Jellyfin, go to the Live TV section and add a new tuner, selecting "M3U Tuner" as the type.
   - Provide the path or URL to your M3U file.



## No-IP Dynamic DNS Setup on Ubuntu

- To set up DNS that maps to your public ip each time you start the instance automatically

### Update and Install Required Packages
```
sudo apt update
sudo apt install build-essential make
sudo apt install make
```
## Download and Install No-IP DUC
```
cd /usr/local/src
sudo wget http://www.no-ip.com/client/linux/noip-duc-linux.tar.gz
sudo tar xzf noip-duc-linux.tar.gz
cd noip-2.1.9-1
sudo make
sudo make install
```

### To Configure the Client

- As root again (or with sudo) issue the below command:
```
/usr/local/bin/noip2 -C  
```
(dash capital C, this will create the default config file)
You will then be prompted for your username and password for No-IP, as well as which hostnames you wish to update.  Be careful, one of the questions is “Do you wish to update ALL hosts”.  If answered incorrectly this could affect hostnames in your account that are pointing at other locations.

Now that the client is installed and configured, you just need to launch it.  Simply issue this final command to launch the client in the background:

```
/usr/local/bin/noip2
```


### Create a service for No-IP's Dynamic Update Client (DUC) on Ubuntu

1. **Create a New Service File**:
   - Open a terminal and use a text editor to create a new service file:
     ```bash
     sudo nano /etc/systemd/system/noip2.service
     ```

2. **Add the Following Content to the File**:
   ```ini
   [Unit]
   Description=No-IP Dynamic Update Client
   After=network.target

   [Service]
   Type=forking
   ExecStart=/usr/local/bin/noip2
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```

3. **Enable and Start the Service**:
   - Enable the service so it starts on boot:
     ```bash
     sudo systemctl enable noip2.service
     ```
   - Start the service:
     ```bash
     sudo systemctl start noip2.service
     ```

4. **Check the Service Status**:
   - To ensure the service is running properly, check its status:
     ```bash
     sudo systemctl status noip2.service
     ```


---


# Mount Google drive as storage 

To modify your Docker Compose file to use the Rclone Docker Volume Plugin with Google Drive, you'll need to create named volumes that interface with Google Drive via Rclone. This setup will allow your services like Jellyfin, Radarr, Sonarr, and Prowlarr to use data stored on Google Drive.

[install rclone and configure it](https://ostechnix.com/mount-google-drive-using-rclone-in-linux/)

[create api for google drive](https://rclone.org/drive/#making-your-own-client-id)

#### Note Remember to first mount rclone on a specfic folder before moving to next steps

1. **Install FUSE:** - as Rclone depends on FUSE for mounting.

```
sudo apt-get -y install fuse
```


2. **Create Plugin Directories:** Create the necessary directories for the Rclone Docker plugin:
   ```bash
   sudo mkdir -p /var/lib/docker-plugins/rclone/config
   sudo mkdir -p /var/lib/docker-plugins/rclone/cache
   ```

3. **Install Rclone Plugin:** Install the Rclone Docker plugin:
   ```bash
   docker plugin install rclone/docker-volume-rclone:amd64 --alias rclone --grant-all-permissions
   ```

4. **Configure Google Drive in Rclone:**
   - Run `rclone config` on a machine with a GUI and web browser.
   - Set up a Google Drive remote and obtain the `rclone.conf` file.
   - Transfer this `rclone.conf` to your Docker host and place it in `/var/lib/docker-plugins/rclone/config`.

5. **Create Rclone Volumes:**
   - Use `docker volume create` to create volumes for your Google Drive paths. For example, for TV shows and movies:
     ```bash
     docker volume create gdrive_tvshows -d rclone -o remote="gdrive:tvshows" -o vfs-cache-mode=full -o allow_other=true
     docker volume create gdrive_movies -d rclone -o remote="gdrive:movies" -o vfs-cache-mode=full -o allow_other=true
     ```

6. **Modify Docker Compose File:** Replace the direct mount points in your Docker Compose file with the named Rclone volumes. Here's an example for the `tvshows` and `movies` directories:

   ```yaml
   version: '3'
   services:
     jellyfin:
       image: jellyfin/jellyfin
       container_name: jellyfin
       network_mode: "host"
       environment:
         - PUID=1000
         - PGID=1000
       volumes:
         - gdrive_tvshows:/tvshows
         - gdrive_movies:/movies
         - ./config/jellyfin:/config
       restart: unless-stopped

     # Similar changes for radarr, sonarr, prowlarr services
     # Use the gdrive_tvshows and gdrive_movies volumes as needed

   volumes:
     gdrive_tvshows:
       external: true
     gdrive_movies:
       external: true
   ```

7. **Deploy Your Stack:** Use `docker-compose up -d` to deploy your stack with the new volume configuration.

Remember to replace `gdrive:tvshows` and `gdrive:movies` with the correct paths to your Google Drive directories. The paths should match the structure in your Google Drive. Ensure all steps are correctly followed and that the `rclone.conf` file is correctly placed.


---

# Add a qBittorrent service to Docker Compose

And we will also configure it to use a Google Drive volume via Rclone

Here's how to integrate qBittorrent

1. **Create Rclone Volume for qBittorrent Downloads:**
   If you want qBittorrent to use a separate download directory on Google Drive, create a named volume specifically for it. 
   ```bash
   docker volume create gdrive_qbittorrent -d rclone -o remote="gdrive:qbittorrent" -o vfs-cache-mode=full -o allow_other=true
   ```

2. **Modify Docker Compose File:**
   Add the qBittorrent service to your `docker-compose.yml` file and configure it to use the created Rclone volume. Ensure that you set the appropriate user and group IDs (`PUID` and `PGID`) for file permission consistency.


   ```yaml
   version: '3'
   services:
     # ... other services ...

      qbittorrent:
    image: linuxserver/qbittorrent
    container_name: qbittorrent
    environment:
      - PUID=1000  
      - PGID=1000  
      - TZ=Europe/London  
    volumes:
      - ./config/qbittorrent:/config
      - gdrive_qbittorrent:/downloads  
    ports:
      - 8080:8080  
    restart: unless-stopped

   volumes:
     gdrive_tvshows:
       external: true
     gdrive_movies:
       external: true
     gdrive_qbittorrent:
       external: true
   ```

   In this configuration, qBittorrent will store its configuration files in `./config/qbittorrent` on the host and use the `gdrive_qbittorrent` volume for downloads.

3. **Deploy Your Updated Stack:**
   Run `docker-compose up -d` to deploy your stack with qBittorrent and the updated volume configurations.

