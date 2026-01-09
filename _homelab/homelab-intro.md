---
title:  "Homelab Intro"
excerpt: "How I host useful services and own my own data with my home server."
date: 2026-01-04
last_modified_at: 2026-01-08
order: 1
---

I administrate and run my own home server with some useful self hosted services. These services include Immich to store my photo library, SMB and NFS servers providing network storage to all my devices, and much more.

## Why Self-Host?

To start, life already has too many subscriptions. OneDrive, Google One, AI Subscriptions, you name it, it probably has a subscription. With my own server, I can host my own storage. This lets me avoid cloud storage costs and put my data into my own hands.

Some things are just better than cloud subscriptions. For example, I can't just mount my cloud storage as a network drive (not in a performant way that is). Can you edit videos or run virtual machines off of a cloud storage provider? Didn't think so.

Also, self hosting is just fun and you end up learning a lot throughout the process. I do genuinely enjoy spending time making improvements, adding new useful services, and just managing my server in general. I've learned so much about many things like what a reverse proxy is, DNS records, networking and firewall concepts, docker, etc. It's a good time.

### Pitfalls

All of that being said, it wouldn't be fair to say that there aren't any downsides to doing things this way. There are numerous pitfalls to be aware of and if you don't prepare, you may have to learn them the hard way. Just to start, like I said earlier, your data is in your own hands. Proper backups (preferably of the 3-2-1 variety) are <ins>**critical**</ins> or you will lose data at some point or another. Check out my post on backups to see how I manage them in my servers. Security is also paramount. If you don't lock things down properly, you might wake up to someone mining bitcoin on your poor server (or worse, maybe snooping through your files).

## My Setup

From the outside, my server certainly doesn't look the part. When you hear the word "server", you may think of a rack-mounted affair meant to live inside a server rack. For my use case and budget, this was unfortunately not practical (though I would love to have that some day). My server is simply a desktop PC with an Intel® Core™ i5-12400, 32GB of RAM, a 512GB boot SSD, and several hard drives for mass storage. The main storage pool consists of 2 16TB Seagate IronWolf Pros mirrored in RAID-1 for redundancy.

## Software

My server runs on Proxmox as its main OS. I chose Proxmox for its relative simplicity and ease of use for virtualization. Virtualization allows a lot of benefits compared to running on bare metal. For example, backing up an entire VM with its data is a breeze with snapshotting and live incremental backups. As well, creating VMs is just fun and I have the ability to mess around in non production VMs without any fear of me breaking my own services that I rely on.

I currently run two virtual machines to host all my production services. The first is a TrueNAS Scale VM which handles the hard drives pools with ZFS. TrueNAS has a handy web UI that makes it easy to setup automatic snapshots, ZFS replication (for backups), and SMB + NFS shares. As well, ACLs are properly supported which makes permissions just a tad less painful with multiple users.

The second VM runs Debian which handles anything not related to storage management. Most services run in Docker, though I have a small few that are not such as my Wireguard VPN. NFS is setup so that large files can be stored on the storage pool managed by TrueNAS. Currently, I run the following services:

- Authelia (Single Sign-On)
- LLDAP (User and Group Management)
- Forgejo (Like my own GitHub)
- Immich (Photo Library)
- Mealie (Recipes)
- Navidrome (Music Library)
- Trilium (Notes)
- Traefik (Reverse Proxy)

I aim to make articles going further into detail about how I setup these services. Look out for articles about setting up SSO with Authelia, proper SSL with Traefik, and Wireguard for secure remote access!
