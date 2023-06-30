---
published: true
layout: posts
comments: false
title: Switch from FSTAB to Autofs for NFS mounts.
categories:
  - blog
tags:
  - Docker
  - Ubuntu
  - NFS
---
## Introduction

This post will very briefly explain why you might want to switch from using `/etc/fstab` to `autofs` for your NFS mounts.

Reason #1: NFS causing boot to hang.

During a recent power outage, I took my NAS devices offline to save on UPS load.
But the servers were up and running and hung while trying to mount the NFS share.

AutoFS lets you define mounts that don't get mounted until you try to access them. Like when your docker containers start up.

## Installation and Config

Installing is pretty simple.

`apt install -y autofs`

Configuration is also pretty easy once you figure it out.

You'll create two files. Replace the `<something>` with a meaningful tag, like the name of your NAS.

`/etc/auto.master.d/<something>.autofs`

and

`/etc/auto.<something>`

First we'll set up the master file.

```bash
vi /etc/auto.master.d/<something>.autofs`
<edit/paste line below>

/-   /etc/auto.<something> --timeout 600 --browse
```
* `/-`  indicates that the paths in the map file are non-relative.
* `/etc/auto.<something>` is the path to the map file.
* `--timeout 600` how long to wait to auto-unmount the share after last access
* `--browse` if the NFS share is unavailable, create an empty directory so things don't hang forver

Next we'll set up the map file. (You might need to create `/mnt/nfs` first)
```bash
vi /etc/auto.<something>
<edit/paste line below>

/mnt/nfs/<share> -rw,fstype=nfs,soft,rsize=8192,wsize=8192 <nfs-server.example.com>:</path/to/share>
```
* `/mnt/nfs/<share>` where you want the share mounted locally
* `-rw,fstype=nfs,soft,rsize=8192,wsize=8192` mounted it read-write, NFS, and some packet sizes for performance
* `<nfs-server.example.com>:</path/to/share>` the FQDN to your NFS server and the path to the share.

## Running

I'm running Ubuntu, so you'll set it up just like any other service.
```bash
systemctl enable autofs
systemctl start autofs
systemctl status autofs
```

Then you just `cd` to what you set in `/mnt/nfs/<share>` and after a second or two, your remote NFS data should be there.

Make sure to remove or comment out the previous entry in `/etc/fstab`.

## Notes

I kept running into issues when I was trying to use relative paths in the map file, which is supposed to be supported. But I couldn't get it working, seeing the error(s) below.
```
automount[68274]: key "-" not found in map source(s).
```

Switch to that `/-` in the master file and then using a full path in the map file was the key to success.