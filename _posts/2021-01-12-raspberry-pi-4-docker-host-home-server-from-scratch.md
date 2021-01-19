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

This post is mostly for my own notes, but others might find it helpful. Here I'll document the process of buying, building, booting, and configuring a Raspberry Pi 4 (8GB) as a home server that runs various applications in Docker. Of course, you don't necessarily *need* to do all of the steps below to get a functional server, this is just what I did to get my Pis up and running how I wanted them.

### Assumptions

- You already have a functioning home network.
- You know the basics of linux, networking, docker, hardware, etc.
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
 - The process to make it a fully functioning home server
- Housekeeping and Monitoring
 - Setting up some backups scripts, and using Telegraf to monitor stats about the Pi itself and Docker


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

##### Pics and stuff

### Booting - Getting it running

##### Create the SD card using the latest Raspberry Pi OS

##### Mount the SD card in your computer to enable headless/ssh
[Follow this guide](https://www.shellhacks.com/raspberry-pi-enable-ssh-headless-without-monitor/)

### Configuring - The bulk of the work is in here...

#### Initial Stuff

##### Change your password
If you intend to use the default *pi* user, change the password. If you don't, either change it anyway or remove the user account.
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
systemctl disable apt-daily.timer          # More on this later
systemctl disable apt-daily-upgrade.timer  # More on this later
systemctl disable man-db.timer             # More on this later
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
Now let's format it. I like *ext4*. If you prefer something else, use it here.
``` 
mkfs.ext4 /dev/sda1   - /dev/sda1 because sda1 refers to the new partition we created above
```
Now we'll mount it.
```
mkdir /storage              - You can use whatever you want here, I like /storage
mount /dev/sda1 /storage
```
Add the following to */etc/fstab* to make sure the SSD gets mounted at boot.
```
# Mount the SSD drive.
/dev/sda1	/storage	ext4	defaults 	0	0
```

##### Adjust tmpfs sizes, add some mount points
While we're mucking with */etc/fstab*, we might as well make some adjustments here we can use later.
Add the following to */etc/fstab*
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

##### Reboot
Reboot so you can take advantage of your newly cleaned and updated system before you do the rest of the work.

#### Install some packages
Install a few handy packages that will make your life easier.
```
apt install ntp dc telnet screen ntpdate busybox-syslogd
```
Yes, telnet. The telnet client, not the server, is useful for quickly checking to see if you can access open ports on another system, or locally. There's probably a better tool, sue me.

Busybox-syslogd replaces rsyslog that we removed earlier. There are two ways to read logs going forward.
1. *logread*      - ringlog that shows you a rolling list of the last N (I think it's 128 bytes) worth of logs.
2. *journalctl*   - All of the system logs since the system booted.
NOTE: I need to understand a bit better about which logs go where and how to manipulate that. It's on my TODO list and I'll update here when I better understand it.

##### Telegraf
Even if you plan to run Docker (like I am, and will explain later), it's a good idea to run Telegraf as a system install.


