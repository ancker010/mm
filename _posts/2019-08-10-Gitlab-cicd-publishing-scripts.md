---
layout: post
title: "Use GitLab CI/CD to Publish Script RPM/DEB to your servers."
date: 2019-08-10 08:00:00 -0600
comments: false
hidden: false
published: false
categories:
  - Blog
tags:
  - CICD
  - gitlab
  - python
  - automation
  - networking
  - rpm
---

Introduction
============

At the dayjob, we're in the process of evaluating everything we do with an eye towards automation and process efficiency. 
As we're doing this we're experimenting with various tools to get some ideas on how we can incorporate them into our workflows.
I've personally been intrigued by CI/CD for network tasks. IMO, that's the holy-grail for automation. 
Commit some changes and it fires off a bunch of different processes that verify, test, stage, then deploy those changes.

This post will outline the steps required to automate the process of bundling a script (and possible dependency files) 
as an RPM or DEB (or both), test them, then publish them to a staging repo for manual testing, then when you're ready,
publish to your production package repository. (At the moment this is limited to a Yum repo since I don't have a apt-get)
repo to test against.

Prerequisites
============

* Linux skills. (Probably obvious.)
* Some script you want to package. (Or you can use my sample script.)
* A basic understanding of Git.
* A private/local installation of GitLab. (This article assumes you have disk access.)
* Basic understanding of [GitLab's CI/CD Pipeline](https://docs.gitlab.com/ee/ci/)
* Have already installed `gitlab-runner` and have (or can google how to) [register a GitLab Runner](https://docs.gitlab.com/runner/) for your specific project.

The Script
===============

First we need to start with a script that you want to package up. We'll use some simple python here to make it small and
easy to comprehend. Obviously, replace with your own script, or adjust to do something useful.

Filename: hello-world.py
```python
#/usr/bin/python

print("Hello, World!")
```

Directory Structure
==============
The first thing we need to put together is a directory structure for your scripts.
This is important because you'll want to make it easy to just bundle your script files
without having to copy or move them around in the GitLab CI/CD process.

We store everything in a `/neteng` directory on our servers. So that's our root. Then on
top of that, we have `/neteng/bin` for scripts "people" will run, and `/neteng/sbin` for 
scripts that run in cron, or things that "people" won't usually run directly. We also have
`/neteng/etc/<project>` and `/neteng/templates/<project>` for any project specific config 
files or templates. Then finally, any templates that might be used by multiple projects
can be stashed in `/neteng/templates` itself.

To recap:
```bash
/neteng
/neteng/bin
/neteng/sbin
/neteng/templates
/neteng/templates/<project>
/neteng/etc/<project>
```
You don't need to create all of these, but it makes sense to standardize this in each of your 
project repositories so you don't have to create custom GitLab CI/CD processes. 


Yum Repository
===============
For this post, we'll create a Yum repo. You can replace this process with whatever repo
system you prefer. If you're building DEB packages, you'll want to use that style of repository.

For the purposes of this post, we're going to create a Yum repository on the server where GitLab is 
installed. As I mentioned before, this probably isn't the best long-term option, but it certainly
makes things easier for an example post. If you want to copy the RPM/DEB packages to a remote Yum repo
you're already running, you'll need to figure out how to [import ssh keys into the GitLab-Runner container](https://docs.gitlab.com/ee/ci/ssh_keys/)
and adjust it to work with your scenario. There's lots of examples out there, so I won't get into how
to do that here. 

First you need to `yum install createrepo` then change to the directory where you want to create a repo
and run `createrepo .`. Yes. It's that simple.

For this example:
```bash
mkdir /neteng/rpm-repos
mkdir /neteng/rpm-repos/staging /neteng/rpm-repos/production
yum install createrepo
cd /neteng/rpm-repos/staging
createrepo .
cd /neteng/rpm-repos/production
createrepo .
```


GitLab Runner Tweaks
=============
This step is only required if you're running your repository on your GitLab server like we are for this example.
This tweak is required to allow the GitLab Runner, which is just a docker container, access to the local filesystem
so it can easily copy the RPM/DEB and then tell the repo to update.

```bash
cd /etc/gitlab-runner
vi config.toml
# locate the stanza for the runner you registered to this project
# Locate the line with "volumes" and replace it with:
volumes = ["/neteng/rpm-repos:/neteng/rpm-repos:rw"]
# Save the file then:
systemctl restart gitlab-runner
```

This tweak will make the `/neteng/rpm-repos` directory available inside your runner.



.gitlab-ci.yml
===============



Testing
========================



Versioning
=======


