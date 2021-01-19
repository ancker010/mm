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

### Booting - Getting it running

#### Create the SD card using the latest Raspberry Pi OS

#### Mount the SD card in your computer to enable headless/ssh
[Follow this guide](https://www.shellhacks.com/raspberry-pi-enable-ssh-headless-without-monitor/)

