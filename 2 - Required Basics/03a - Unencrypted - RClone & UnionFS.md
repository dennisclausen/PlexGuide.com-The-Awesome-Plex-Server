# Unencrypted - RClone, UnionFS & Move
WARNING: Chose Either 03A or 03B

- RClone:  Mounts your Google Drive (not used as primary due to API Bans)
- UnionFS: Moves multiple drives
- Move:    Made to sync files from your local drive to your google drive

## Setting Up & Installing RClone

```sh
### Creating Folders
sudo mkdir /mnt/rclone-union
sudo mkdir /mnt/rclone-move
sudo mkdir /mnt/rclone-move/tv
sudo mkdir /mnt/rclone-move/movies

### Chaning Permissions
sudo chmod 755 /mnt/rclone-move && sudo chmod 755 /mnt/rclone-union

## To Install Fuse
sudo apt-get install unionfs-fuse

## Installing RClone (Can Copy Entire and Execute Entire Mini Block Below)

cd /tmp
sudo curl -O https://downloads.rclone.org/rclone-current-linux-amd64.zip
sudo unzip rclone-current-linux-amd64.zip
cd rclone-*-linux-amd64
sudo cp rclone /usr/bin/
sudo chown root:root /usr/bin/rclone
sudo chmod 755 /usr/bin/rclone
sudo mkdir -p /usr/local/share/man/man1
sudo cp rclone.1 /usr/local/share/man/man1/
sudo mandb
cd .. && sudo rm -r rclone*
sudo mkdir /mnt/rclone
sudo chmod 755 /mnt/rclone
sudo chown root /mnt/rclone
```

Configure RClone config as Root 

```sh
sudo su
rclone config
```

## Configuring RClone
May change due to version | Currently for version 1.38

- N < For New remote 
- gdrive < for the name
- 9 < For Google Drive (double check the number select incase)
- Enter Your Google ID
- Enter Your Goole Secret
- N < for headless machine #### NOTE: if your on a VM or the actual machine with an interface (GUI), select Y.  
- Enter Your Verification Code
 - Note: If you copy and paste by SELECTING and CLICK the RIGHT Mouse button, it will work; but then you will see it repeat
 -Note: Hold the DEL button to del the extra crap and then paste into a browser to get the verfication code
- N < Configure this as a team drive?
- Y < If asking all is ok?
- N < For New remote
- local < for the name
- 11 < For a Local Drive
- /mnt/rclone-move
- Y < Is asking all is ok?
- Q < to quit

### Back to your username

```sh
su [YOURUSERNAME]
```

### Allow multiple connections for fuse

```sh
sudo nano /etc/fuse.conf
```
- remove the (#) symbol before user_allow_other; then press CTRL+X and then save

### Create RClone Service

```sh
sudo nano /etc/systemd/system/rclone.service
```

- Copy the following below into the Nano Edit for rclone.service

```sh
[Unit]
Description=RClone Daemon
After=multi-user.target

[Service]
Type=simple
User=root
Group=root
ExecStart=/usr/bin/rclone --allow-non-empty --allow-other mount gdrive: /mnt/rclone --bwlimit 8650k --size-only
TimeoutStopSec=20
KillMode=process
RemainAfterExit=yes
 
[Install]
WantedBy=multi-user.target
```

- Press CTRL+X and then Yes to save

### Start and enable the service

```sh
sudo systemctl daemon-reload
sudo systemctl enable rclone.service
sudo systemctl start rclone.service
sudo systemctl status rclone.service
```

- Press CTRL + C to exit the status message


### Create UnionFS Service ### Thanks @Alasano for fixing the startup script
sudo nano /etc/systemd/system/unionfs.service
 
##### START COPY #####

[Unit]
Description=UnionFS Daemon
After=multi-user.target

[Service]
Type=simple
User=root
Group=root
ExecStart=/usr/bin/unionfs -o cow,allow_other,nonempty /mnt/rclone-move=RW:/mnt/plexdrive4=RO /mnt/rclone-union
TimeoutStopSec=20
KillMode=process
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target


##### END COPY #####
# Press CTRL+X and then Yes to save

### Start and enable the service
sudo systemctl daemon-reload
sudo systemctl enable unionfs.service
sudo systemctl start unionfs.service
sudo systemctl status unionfs.service
################# UNIONFS ################# END

################# Move Service ################# START
### Create Script ### Inspired by aj1252 - rclone forum
# Create the SCRIPT the CMD
sudo nano /opt/rclone-move.sh

######## Copy ####### START
#!/bin/bash

while true
do
# Purpose of sleep starting is so rclone has time to startup and kick in
sleep 30
# Anything above 8M will result in a google ban if uploading above 8M for 24 hours
rclone move --bwlimit 8M --tpslimit 4 --max-size 99G --log-level INFO --stats 15s local:/mnt/rclone-move gdrive:/
sleep 270
done

######## Copy ####### END
# Press CTRL+X and ENTER 

# Script permissions
sudo chmod 755 /opt/rclone-move.sh

### Create UnionFS Service
sudo nano /etc/systemd/system/move.service

##### START COPY #####

[Unit]
Description=Move Service Daemon
After=multi-user.target

[Service]
Type=simple
User=root
Group=root
ExecStart=/bin/bash /opt/rclone-move.sh
TimeoutStopSec=20
KillMode=process
RemainAfterExit=yes
Restart=always

[Install]
WantedBy=multi-user.target

##### END COPY #####
# Press CTRL+X and then Yes to save

### Start and enable the service
sudo systemctl daemon-reload
sudo systemctl enable move.service
sudo systemctl start move.service
sudo systemctl status move.service
# Press CTRL + C to exit the status message
################# RClone Move Service ################# END