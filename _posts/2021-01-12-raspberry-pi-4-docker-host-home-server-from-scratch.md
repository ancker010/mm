---
published: false
layout: posts
comments: false
title: Raspberry Pi 4 Docker Host/Home Server - From Scratch
categories:
  - blog
tags:
  - Raspberry Pi
  - Docker
  - SSD
  - PiHole
  - InfluxDB
  - Grafana
  - Telegraf
  - Python
  - Traefik
  - ZoneMinder
---
## Introduction

This post is mostly for my own notes, but others might find it helpful. Here I'll document the process of buying, building, booting, and configuring a Raspberry Pi 4 (8GB) as a home server that runs various applications in Docker. Of course, you don't necessarily **need** to do all of the steps below to get a functional server, this is just what I did to get my Pis up and running how I wanted them.

### Assumptions

- You already have a functioning home network.
- You know the basics of linux, networking, docker, hardware, etc.
- All of the commands below assume you are operating as root, or understand that you need to append **sudo** if using an underprivileged user
- You can work your way around basic applications, like the Raspberry Pi Imager app
- other stuff I remember later

## Outline
- Buying
 - Actual hardware you need to purchase
- Building
 - Any gotchas around connecting any of the hardware itself
- Booting
 - Creating a bootable SD card and editing it so it boots correctly
- Configuring
 - Initial set up
 - Remove unnecessary packages
 - Set up the SSD
 - Set up Read-Only Mode
 - Install some packages
  - Set up some monitoring stuff
- Docker
 - Pulling and running some containers with docker-compose
- Backups
 - Setting up some backups scripts


### Buying - Open your wallet

First off, what do you need to buy to do even do this? 

1. A Raspberry Pi 4B Kit (I went with the 8GB RAM version, because more is better)
2. A MicroSD Card (I went with 32GB because I had a few spares)
3. An SSD/mSATA/M.2 Add-on Card (For adding external storage)
4. An SSD/mSATA drive (Dedicated USB Storage)
5. RPi4 Heat Sinks (maybe not required, but really helps!)
6. RPi4 Active Cooling Fan (also not required, but really helps)
7. Short USB-A to USB-A Cable (don't use the U shaped USB thing included with the add-on cards)
8. Power cables. USB-C cable/brick that can deliver up to 3 Amps at 5V-DC. And a second 5.5x2.5mm cable that can deliver the same specs.
9. USB Power Bank. Any power bank that can deliver 3 Amps per port will work.

#### Links to Stuff
1. [RPi4 Kit - via eBay](https://www.ebay.com/itm/Vilros-Raspberry-Pi-4-with-USB-C-Micro-HDMI-Adapters/313125561738?ssPageName=STRK%3AMEBIDX%3AIT&var=612153551075&_trksid=p2057872.m2749.l2649)
2. [32GB MicroSD Card](https://www.amazon.com/SanDisk-Ultra-microSDXC-Memory-Adapter/dp/B073JWXGNT/ref=sxin_9?ascsubtag=amzn1.osa.e5e0b91d-64d7-4ef7-a6e5-3682102bf716.ATVPDKIKX0DER.en_US&creativeASIN=B073JWXGNT&cv_ct_cx=microsd&cv_ct_id=amzn1.osa.e5e0b91d-64d7-4ef7-a6e5-3682102bf716.ATVPDKIKX0DER.en_US&cv_ct_pg=search&cv_ct_we=asin&cv_ct_wn=osp-single-source-gl-ranking&dchild=1&keywords=microsd&linkCode=oas&pd_rd_i=B073JWXGNT&pd_rd_r=46fdaf5d-1fa2-44c3-bff3-38105add0cbc&pd_rd_w=K7SmA&pd_rd_wg=rzRHb&pf_rd_p=53f37bb1-bef6-4b9e-be3a-0696c5f5ad01&pf_rd_r=1CNX2C931HSXNR27KM9A&qid=1610469915&s=electronics&sr=1-1-d9dc7690-f7e1-44eb-ad06-aebbef559a37&tag=androidcentralosp-20)
3. [mSATA](https://www.amazon.com/gp/product/B085HG7ZLJ/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1) or [M.2](https://www.amazon.com/gp/product/B088HCY4TH/ref=ppx_yo_dt_b_asin_title_o06_s00?ie=UTF8&psc=1) Add-on Card
4. [mSATA](https://www.amazon.com/gp/product/B084Q137YJ/ref=ppx_yo_dt_b_asin_title_o00_s00?ie=UTF8&psc=1) or [M.2](https://www.amazon.com/gp/product/B07V1KHPKK/ref=ppx_yo_dt_b_asin_title_o06_s01?ie=UTF8&psc=1) SSD Drive
5. Included in RPi4 Kit Linked above. There's also a handy mini-HDMI to HDMI dongle.
6. [Active Cooling Fan](https://www.amazon.com/gp/product/B07Y9LFP1J/ref=ppx_yo_dt_b_asin_title_o03_s00?ie=UTF8&psc=1)
7. [USB Cable](https://www.amazon.com/gp/product/B0891X229X/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1)
8. [USB-C for Power](https://www.amazon.com/gp/product/B0765CFXBM/ref=ppx_yo_dt_b_asin_title_o04_s01?ie=UTF8&psc=1) and [USB-to-5.5x2.5mm for power](https://www.amazon.com/gp/product/B00ZUE4WBE/ref=ppx_yo_dt_b_asin_title_o04_s01?ie=UTF8&psc=1)
9. [USB Power Bank](https://www.amazon.com/gp/product/B08HN6JK7N/ref=ppx_yo_dt_b_asin_title_o04_s02?ie=UTF8&psc=1) (Yeah-yeah, this is only 2.4A per port, but that's probably going to be fine for my uses.)

### Building - Putting it all together
Assembling the various pieces should be fairly self-explanatory. Hopefully if you're reading this guide, you can figure it out. Nevertheless, there's some pics below that might be helpful.

##### Pics and stuff
First, the Raspberry Pi4 and the heat sinks that came with the kit. 

[<img src="https://github.com/ancker010/rpi-home-server/raw/main/pics/pi_sinks.heic" width="250" />](https://github.com/ancker010/rpi-home-server/raw/main/pics/pi_sinks.heic)[<img src="https://github.com/ancker010/rpi-home-server/raw/main/pics/pi_sinks_on.heic" width="250" />](https://github.com/ancker010/rpi-home-server/raw/main/pics/pi_sinks_on.heic)

Second, all of the pieces. The Pi4, the fan hat, the mSATA (or M.2) SSD, and the SSD adapter.

[<img src="https://github.com/ancker010/rpi-home-server/raw/main/pics/full_parts.heic" width="250" />](https://github.com/ancker010/rpi-home-server/raw/main/pics/full_parts.heic)

A gotcha. The Argon fan hat has a bolt that holds the fan on the board. This bolt interferes with one of the heat sinks. I removed it so there was adequate clearance. Three out of four screws to hold the fan in place should be adequate. Here's a pic of the one to remove.

[<img src="https://github.com/ancker010/rpi-home-server/raw/main/pics/fan_bolt.jpg" width="250" />](https://github.com/ancker010/rpi-home-server/raw/main/pics/fan_bolt.jpg)

Last, fully assembled. Minus the 1 foot USB cable between the SSD adapter and the Pi4.
**DO NOT, I REPEAT, DO NOT** use the little U shaped USB thingy that comes with the Geekworm SSD adapter. It is NOT electronically shielded and **WILL** interfere with Wifi/bluetooth/anything nearby.

[<img src="https://github.com/ancker010/rpi-home-server/raw/main/pics/assembled_side.heic" width="250" />](https://github.com/ancker010/rpi-home-server/raw/main/pics/assembled_side.heic)[<img src="https://github.com/ancker010/rpi-home-server/raw/main/pics/assembled_top.heic" width="250" />](https://github.com/ancker010/rpi-home-server/raw/main/pics/assembled_top.heic)


### Booting - Getting it running

##### Create the SD card using the latest Raspberry Pi OS

This step is pretty easy. Go [here](https://www.raspberrypi.org/software/) and download the imager for your operating system. Then insert an SD card. Then use the imager app to write the image.

I didn't use any of the images included by default in the app since I wanted to run a 64-bit OS. Running 64-bit is required to take advantage of the full 8GB of RAM in the RPi4-8GB. You can downlaod it [here](https://downloads.raspberrypi.org/raspios_arm64/images/). Pick the directory with the latest date. (I used 2020-08-24.)

##### Mount the SD card in your computer to enable headless/ssh
[Follow this guide](https://www.shellhacks.com/raspberry-pi-enable-ssh-headless-without-monitor/)

### Configuring - The bulk of the work is in here...

#### Initial Stuff

##### Change your password
If you intend to use the default **pi** user, change the password. If you don't, either change it anyway or remove the user account.
```
passwd
```

##### Remove a bunch of crap you don't need
NOTE: This can probably be avoided by selecting a more light weight OS or image, but I went with the Raspberry Pi (buster) OS that had 64bit support so I could use all 8GB of the RAM.

```
sudo apt remove "x11-*" triggerhappy logrotate dphys-swapfile rsyslog
sudo apt autoremove
```

##### Do an OS Update
Just a good idea. The image you downloaded to your SD is likely outdated, get the latest versions of everything. This will also update you to the latest version of the RPi bootloader, if needed.
```
apt update
apt full-upgrade
```

##### Disable a bunch of services you don't need
Turn off some services that we won't need as a headless home server.
```
systemctl disable avahi-daemon
systemctl disable bluetooth
systemctl disable wpa_supplicant
systemctl disable apt-daily.timer           # More on this later
systemctl disable apt-daily.service         # More on this later
systemctl disable apt-daily-upgrade.timer   # More on this later
systemctl disable apt-daily-upgrade.service # More on this later
systemctl disable man-db.timer              # More on this later
```

##### Partition, Format, Mount your fancy SSD drive.
Typical Linux process here.
Run *dmesg* to see what device id your SSD was given, then use fdisk to partition it, them mount it and add it to /etc/fstab.
```
dmesg | grep sd
[    2.111554] sd 0:0:0:0: [sda] 500118192 512-byte logical blocks: (256 GB/238 GiB)
[    2.111752] sd 0:0:0:0: [sda] Write Protect is off
[    2.111781] sd 0:0:0:0: [sda] Mode Sense: 43 00 00 00
[    2.112116] sd 0:0:0:0: [sda] Write cache: enabled, read cache: enabled, doesn't support DPO or FUA
[    2.112862] sd 0:0:0:0: [sda] Optimal transfer size 33553920 bytes
[    2.129604]  sda:
```
We see that it's */dev/sda*.
```
fdisk /dev/sda
Enter p         - To ensure you're on the right disk, check the reported size, etc...
Enter n         - To create a new partition on the disk
Enter p         - To specifiy a primary partition type
Enter 1         - Since this is the first (and only) partition we're creating
Enter <enter>   - To select the first sector as the start of your new partition
Enter <enter>   - To select the last sector as the end of your new partition (Your new partition is th full size of the disk)
Enter w         - To write the new changest to the disk
```
Now let's format it. I like **ext4**. If you prefer something else, use it here.
``` 
mkfs.ext4 /dev/sda1   - /dev/sda1 because sda1 refers to the new partition we created above
```
Now we'll mount it.
```
mkdir /storage              - You can use whatever you want here, I like /storage
mount /dev/sda1 /storage
```
Add the following to **/etc/fstab** to make sure the SSD gets mounted at boot.
```
# Mount the SSD drive.
/dev/sda1	/storage	ext4	defaults 	0	0
```

##### Adjust tmpfs sizes, add some mount points
While we're mucking with **/etc/fstab**, we might as well make some adjustments here we can use later.
Add the following to **/etc/fstab**
```
### Mount a few paths as tmpfs so they exist in RAM instead of on disk (SD card)
tmpfs        /tmp            tmpfs   nosuid,nodev,size=256M         0       0
tmpfs        /var/log        tmpfs   nosuid,nodev,size=100M         0       0
tmpfs        /var/tmp        tmpfs   nosuid,nodev,size=100M         0       0

# And this so docker will work in Read-only mode
tmpfs        /var/lib/containerd tmpfs   nosuid,nodev,size=512M         0       0

### Make some default stuff smaller to save RAM
tmpfs      /dev/shm	tmpfs		nosuid,nodev,size=256M 		0	0
tmpfs      /run		tmpfs		nosuid,nodev,size=256M 		0	0
```

##### Set your pi's hostname
Edit **/etc/hostname**
```
vi /etc/hostname
```

##### Reboot
Reboot so you can take advantage of your newly cleaned and updated system before you do the rest of the work.

We'll reboot a few times throughout this guide. Luckily these things boot super quickly.

#### Make your SD Card and root filesystem read-only
Why? Because SD cards aren't designed to be constantly written to. If you do, they'll eventually (months) degrade and fail. And that's no fun. They will still eventually fail, but this should let you run them for much much longer.

I used [this guide](https://medium.com/swlh/make-your-raspberry-pi-file-system-read-only-raspbian-buster-c558694de79) as the basis for most of this. Credit to Andreas Schallwig for the excellent write up.

##### Remove stuff that likes to write files constantly
Oh wait, we already did this above. Check!

##### Adjust your kernel parameters to turn off swap, disable filesytem checking, and operate in read-only mode
```
vi /boot/cmdline.txt
# Add the following three parameters to the only line that should be there.
fastboot noswap ro
```

##### Make your filesystem read-only
Your **/boot** and **/** filesystems need a flag added to make them read-only. You do this via **/etc/fstab** again.
**DO NOT** copy/paste this. Just add the **ro** to each already existing line.

```
PARTUUID=fb0d460e-01  /boot     vfat    defaults,ro          0     2
PARTUUID=fb0d460e-02  /         ext4    defaults,noatime,ro  0     1
```

##### Move some paths to those tmpfs locations you created above
```
rm -rf /var/lib/dhcp /var/lib/dhcpcd5 /var/spool /etc/resolv.conf
ln -s /tmp /var/lib/dhcp
ln -s /tmp /var/lib/dhcpcd5
ln -s /tmp /var/spool
touch /tmp/dhcpcd.resolv.conf
ln -s /tmp/dhcpcd.resolv.conf /etc/resolv.conf
```

##### Update random-seed

```
rm /var/lib/systemd/random-seed
ln -s /tmp/random-seed /var/lib/systemd/random-seed
vi /lib/systemd/system/systemd-random-seed.service
# Add this line above ExecStart=
ExecStartPre=/bin/echo "" >/tmp/random-seed
```

##### Add some aliases to make switching between read-write and read-only easier
```
vi /etc/bash.bashrc
# Paste the below at the end of the file.
set_bash_prompt() {
    fs_mode=$(mount | sed -n -e "s/^\/dev\/.* on \/ .*(\(r[w|o]\).*/\1/p")
    PS1='\[\033[01;32m\]\u@\h${fs_mode:+($fs_mode)}\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
}
alias ro='sudo mount -o remount,ro / ; sudo mount -o remount,ro /boot'
alias rw='sudo mount -o remount,rw / ; sudo mount -o remount,rw /boot ; chmod 1777 /tmp'

PROMPT_COMMAND=set_bash_prompt
```

##### Create a bash_logout file to go back into ro mode when you log out.
```
vi /etc/bash.bash_logout
# Paste the following, and save
mount -o remount,ro /
mount -o remount,ro /boot
```

#### Disable a bunch of services that try to write to disk often
Oh wait, we already did that above.
Remember those `systemctl disable` commands?
We turned off **apt-daily** and **apt-daily-upgrade**.

**NOTE** This means your system is no longer keeping itself up to date automatically. I will talk about a way to do this later in this post, but you can also just do it manually if you want.

##### Reboot
Let's reboot to make sure everything is working properly.

**NOTE** From here on out, you might be presented with an error about the read-only filesystem. It will happen when installing packages, or editing any files that don't live on your SSH. In most cases you just need to run the `rw` command to remount as read-write.


#### Install some packages
Install a few handy packages that will make your life easier.
```
rw # Make you filesystem read-write temporarily.
apt update
apt install ntp dc telnet screen ntpdate busybox-syslogd docker-compose
```
Yes, telnet. The telnet client, not the server, is useful for quickly checking to see if you can access open ports on another system, or locally. There's probably a better tool, sue me.

Busybox-syslogd replaces rsyslog that we removed earlier. There are two ways to read logs going forward.
1. **logread**      - ringlog that shows you a rolling list of the last N (I think it's 128 bytes) worth of logs.
2. **journalctl**   - All of the system logs since the system booted.
NOTE: I need to understand a bit better about which logs go where and how to manipulate that. It's on my TODO list and I'll update here when I better understand it.

##### SystemD Watchdog
This is a process that will automatically safely reboot your Pi4 if it hangs for some reason.
You need to set it up a bit after installing it above.

```
vi /etc/systemd/system.conf
### uncomment (and edit) the following life
RuntimeWatchdogSec=1m
systemctl daemon-reload
```


##### Telegraf
Even if you plan to run Docker (like I am, and will explain later), it's a good idea to run Telegraf as a system install. The main reason is that I want to gather system stats even if docker is broken somehow. 

```
echo "deb https://repos.influxdata.com/debian buster stable" | tee /etc/apt/sources.list.d/influxdb.list
apt update
apt install telegraf
```

Let's set it up to send some metrics to InfluxDB. You don't have an InfluxDB running yet, but you will later. If you don't want or plan to run InfluxDB, or use some other DB, I guess just skip this part.

First we need to edit the main telegraf config file to enable an InfluxDB output and tell it to talk to the local Docker socket.

```
vi /etc/telegraf/telegraf.conf
## Look for "outputs.influxdb"
## Uncomment the [[outputs.influxdb]] line, and edit/replace the urls field to something like the below.
urls = ["http://influxdb.home.example.com:8086"]
## Next enable the Docker section.
## Look for inputs.docker, and either uncomment the bits you need, or just paste this at the bottom:
# # Read metrics about docker containers
[[inputs.docker]]
  ## Docker Endpoint
  ## To use TCP, set endpoint = "tcp://[ip]:[port]"
  ## To use environment variables (ie, docker-machine), set endpoint = "ENV"
  endpoint = "unix:///var/run/docker.sock"
  ## Only collect metrics for these containers, collect all if empty
  container_names = []
  ## Timeout for docker list, info, and stats commands
  timeout = "5s"
  ## Whether to report for each container per-device blkio (8:0, 8:1...) and
  ## network (eth0, eth1, ...) stats or not perdevice = true
  perdevice = true
  ## Whether to report for each container total blkio and network stats or not
  total = false
```

Next we add the stuff to gather a bunch of stats from the RPi.
I am using a version you can find [here](https://github.com/ancker010/rpi-home-server/raw/main/telegraf/rpi-to-influx.conf). You might need to edit it.
I got most of it from [this guide](http://oostens.me/projects/raspberrypiserver/system-monitoring/), so it might be helpful to follow that guide. I of course, skipped the InfluxDB/Telegraf/etc stuff since I already did that stuff.

```
vi /etc/telegraf/telegraf.d
### Paste the file from the github repo above, and save it.
### Then restart telegraf
systemctl restart telegraf
```

##### Docker
Finding a guide to install docker is easy. I'll include my steps and the couple tweaks I made here.

```
curl -sSL https://get.docker.com | sh
```
Edit /etc/docker/daemon.json to place the Docker root directory on your SSD instead of the default.
```
vi /etc/docker/daemon.json
### add
{
  "data-root": "/storage/docker-root"
}
```
Turn on TCP access to the Docker socket for monitoring purposes
```
vi /lib/systemd/system/docker-tcp.socket
### Paste the following:
[Unit]
Description=Docker Socket for the API
PartOf=docker.service

[Socket]
ListenStream=2375

BindIPv6Only=both
Service=docker.service

[Install]
WantedBy=sockets.target
```
Enable the new docker-tcp.socket and restart docker.
```
systemctl daemon-reload
systemctl stop docker.service
systemctl enable docker-tcp.socket
systemctl start docker-tcp.socket
systemctl start docker.service
```

Now you're ready to start running containers with docker.

#### Containers
A few of the containers I run, and how I set them up. From here on out, you can pretty much mix and match the things you want to set up or not. So skip sections for things you don't want.

##### Docker Secrets

Docker allows you to store credentials and other secrets in a file, then reference them in your docker-compose files. Here's how you set it up, using AWS credentials as an example.

```
mkdir /storage/secrets
vi /storage/secrets/aws.creds
###
[default]
aws_access_key_id = <key here>
aws_secret_access_key = <key here>
###
chmod 600 /storage/secrets/aws.creds
```

##### Traefik
Traefik is a reverse proxy for docker. It's a lot more than I use it for. I'm using it as a fancy way to automate running containers that expose port 80/443 (web) and generate Let's Encrypt certificates for them. If none of this interest you, or you already know how to do all of this, skip ahead.

Create your docker-compose.yml file. [This is what I use](https://github.com/ancker010/rpi-home-server/raw/main/traefik/docker-compose.yml), I won't explain what everything means or all of the options are. You can use this as a starting point, or find another guide that explains it.

```
mkdir /storage/traefik
vi /storage/traefik/docker-compose.yml
```

Create your config files. You need both **[api.toml](https://github.com/ancker010/rpi-home-server/raw/main/traefik/api.toml)** and **[traefik.toml](https://github.com/ancker010/rpi-home-server/raw/main/traefik/traefik.toml)**.

The docker-compose.yml file sets up the container to mount **/storage/traefik/conf** as **/etc/traefik** inside the container, so make sure you place them in the right place on your system. Also, create a directory for traefik to store the certificate file.

```
mkdir /storage/traefik/conf
vi /storage/traefik/conf/api.toml
### paste stuff
vi /storage/traefik/conf/traefik.toml
### paste stuff
mkdir /storage/letsencrypt
```

**NOTE**: There's a very important part of this I'm not covering in great detail. I'm using AWS Route53 to host the zone (And do DNS-01 Challenges) where I want to create Let's Encrypt certificates. You'll need to set that up, or something similar with another service. There are lots of guides on how to set that up. 

Now that you're ready to go (and edited those files to match your environment), let's start it up.
```
cd /storage/traefik
docker network create traefik-ext
docker-compose up -d
```

Once that is up and running, and you wait a few minutes. You should be able to browse to whatever container + domainname you configured in your docker-compose.yml file. So from the example it is: **traefik-docker1.home.example.com**. You should be presented with the traefik dashboard, and it should have a valid Let's Encrypt certificate. Hurray!

##### InfluxDB
You may want to skip this section (and the following one for Grafana) if you aren't interested in setting up dashboards for various apps and services you run on your network. But I do, and I wanted to document how I do it.

Create your docker-compose.yml file. Again, [here's an example of mine](https://github.com/ancker010/rpi-home-server/raw/main/influxdb/docker-compose.yml). Same as above, you should be able to figure out what the options mean. :) 

A few things you want to tweak:
1. The influxdb admin username and password. (You'll need it later for Grafana)
2. The hostname in the Traefik config, so it generates the correct certificate. (You'll note that the `certresolver=le` line is commented out in the example. I'm not using SSL, but you might want to for your applications.)

```
mkdir /storage/influxdb
vi /storage/influxdb/docker-compose.yml
```

Next you just need to bring it up.
```
cd /storage/influxdb
docker-compose up -d
```

Most applications (like Telegraf) that can use InfluxDB, will automatically create the databases they need the first time you run them. So running InfluxDB is really as simple as just running the container.

##### Grafana
Same as above, this section can be skipped if you want plan to build dashboards. I have dashboards for everything I can think of.

1. General Service Health
2. Docker Stats for each Server
3. Traffic/Health of my Mikrotik Router (it's also my wireless controller for 3 Mikrotik APs)
4. Cable Modem Health and Stats
5. Zoneminder Event Stats
6. Hubitat Information (Temperatures, doors open/close status, Thermostat settings, etc)
7. Synology NAS Health and Stats

You can find dashboard JSON files for some of these dashboards [here](https://github.com/ancker010/rpi-home-server/tree/main/grafana/dashboards). Most are tweaked versions I found on the Grafana Dashboard site. So YMMV if you try to run them in your environment.

Anyway, let's get it running.

Create your docker-compose.yml file. [Here's mine](https://github.com/ancker010/rpi-home-server/raw/main/grafana/docker-compose.yml). You'll need to tweak the hostname stuff again, and set the admin user password. You'll need it to log into the interface.

```
mkdir /storage/grafana
vi /storage/grafana/docker-compose.yml
```
And run it.
```
cd /storage/grafana
docker-compose up -d
```

If you wait a few minutes, you should be able to browse to the hostname you configured. Like the traefik dashboard above, you should have a valid Let's Encrypt certificate. Log in with `admin` and the password you set the docker-compose file.

Next we need to add InfluxDB as a datasource. Rather than me take a bunch of screenshots, I'll just [link to the official guide from Grafana](https://grafana.com/docs/grafana/latest/datasources/influxdb/). You'll enter something like `http://influxdb.home.example.com:8086"` as the URL, and use the username and password you set in the InfluxDB docker-compose file. **NOTE** Best practice is to create an influxdb user just for grafana rather than use an admin account. You should totally do this. Don't say I didn't warn you. Your risk is probably not that high if you aren't exposing InfluxDB or Grafana outside of your home network, but still.

I won't get into how to create Dashboards. It's fairly easy to find a guide that will help you with that. But since I provided some dashboards in my github repo, we can at least import the basic Server Stats and Docker Stats ones.

To import, [follow this guide](https://grafana.com/docs/grafana/latest/dashboards/export-import/#importing-a-dashboard), copy/paste or upload the json files, select the Datasource you created above, and you should be set.

Because InfluxDB has been running for a bit, and telegraf has been running for a bit, you should almost immediately see metrics in both of the Dashboards. If you don't, there might be some fun detective work to be done. 

##### PiHole
If you haven't heard of PiHole, you should [look it up](https://pi-hole.net/). It's a local DNS server that will use meticulously curated lists of AD, tracking, and other malicious (or not) domains and block them on your network. It can also provide DHCP services, but my router does that.

It is designed to operate on a standalone Raspberry Pi, but I choose to run mine in docker so it's a bit more portable. Here's how I set it up.

Create your docker-compose.yml file. [Example here](https://github.com/ancker010/rpi-home-server/raw/main/pihole/docker-compose.yml). Tweak the file to your environment, make sure to edit the hostname for the certificate, the IP/IPv6 address, and your web admin password.

```
mkdir /storage/pihole
vi /storage/pihole/docker-compose.yml
```

Set up DNSMASQ. This is necessary if you are running a private local zone. I have my router configured with a bunch of local-only DNS entries for all of my home network devices.
Here's an [example config file](https://github.com/ancker010/rpi-home-server/raw/main/pihole/01-pihole.conf).
```
mkdir /storage/pihole/dnsmasq
vi /storage/pihole/dnsmasq/01-pihole.conf
```

Create the log file, it complains if it doesn't already exist for some reason.
```
touch /storage/pihole/logs/pihole.log
```
And run it.
```
cd /storage/pihole
docker-compose up -d
```

#### Backups - In case something goes wrong
I'm not going to cover this in great detail. You should probably already know how to back up important files. The bulk of your critical config is on the SSD drive. It's less likely to fail than the SD card, but you should probably set something up to zip/tar and compress it on a regular basis. Just don't forget to exclude the docker-root directory, you don't need to backup local versions if your docker containers.

Your command will look something like this. It will tar (and bzip2 compress) everything under **/storage** but exclude the **docker-root** directory, any **.log** files, and your backup directory. You may also want to exclude the InfluxDB database files. They can get pretty big, and unless you really really need it, it's not a big deal to lose historical data about your cable modem or interface stats. If you want to exclude them, just add `--exclude="/storage/dockerinfluxdb/data"`.

The command will write the backup file to the SSD, but be sure to remember to copy it off to a USB thumb drive, or another system frequently. Storing your backups locally doesn't help much if the drive fails, does it...

```
tar -cjvf /storage/backups/home-server-backup-ssh-`date +%m-%d-%Y`.bz2 --exclude="/storage/docker-root" --exclude="*.log" --exclude="storage/backups" /storage
```

The easiest, but downtime causing, backup method of the SD card is to just use `dd` or an app to copy the whole SD card to an image file.

I was going to type up a guide for this, but realized there are a ton out there that do a better job and have screenshots. So just use something [like this](https://medium.com/@ccarnino/backup-raspberry-pi-sd-card-on-macos-the-2019-simple-way-to-clone-1517af972ca5). Or [this](https://pimylifeup.com/backup-raspberry-pi/).



### Conlusion
There you have it. You should have a fully functioning home server on your fancy new Raspberry Pi4. This post is mainly for my own purposes, documenting almost everything I did to set up my RPis. I hope it's helpful to you. I will probably update this post as I figure out new ways to do things, so hopefully it'll be consistently accurate over the years. Famous last words, I know, but hey, at least I documented this much. It's a lot more than I usually do for home projects...

I don't enable comments on my posts, and odds are I don't really have time to help through problems. If you do want to reach out, use the Twitter link in the sidebar. My DMs are closed, but just tag me in a tweet and I'll see it.
