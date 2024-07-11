+++
title = 'Nvidia GPU Passthrough for LXC Containers in Proxmox VE'
date = 2024-05-30T14:54:42-04:00
draft = true

description = "Allow Proxmox containers to interact with host's GPU with this simple passthrough method. Currently working on PVE version 8 with a Jellyfin container using the GPU for stream transcoding."
tags = ["Homelab", "Proxmox", "LXC", "Container"]
categories = ["Homelab", "Proxmox"]

#image = "/image.jpg" #featured image
comments = false
math = false
toc = true
readingTime = true
+++


### Introduction

This method of installing Nvidia drivers for passthrough should work on most versions of Proxmox with most modern GPUs,
as we are manually finding and installing the correct version for your device. I am currently using it for transcoding Jellyfin streams 
without issue, but more testing should be done for other purposes.

The thread where I came up with the solution [can be found here](https://forum.proxmox.com/threads/jellyfin-lxc-with-nvidia-gpu-transcoding-and-network-storage.138873/). It was extremely helpful in 
figuring out how to set this up and lead to me creating this simpler method.

### Environment

> Note: Passthrough should work with container's running most software,
> but additional steps may be required required for services running on Docker.

    - Proxmox Cluster running PVE 8.2.2 
    - One host node with RTX 3050 running Nvidia drivers 550.78
    - Jellyfin running on Debian container with 4gb RAM and 4 cores allocated