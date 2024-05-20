---
title: "Nvidia GPU Passthrough for LXC Containers in Proxmox VE"
date: 2024-05-19T12:20:06-04:00
draft: true

# weight: 1
# aliases: ["/first"]
tags: ["Homelab", "Proxmox", "LXC", "Container"]
author: "Andrew"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
hidemeta: false
comments: true
description: "Allow Proxmox containers to interact with host's GPU with this simple passthrough method. Currently working on PVE version 8 with a Jellyfin container using the GPU for stream transcoding."
canonicalURL: "https://adminarchives.com/posts/proxmox-lxc-gpu-passthrough/"
disableHLJS: false # to disable highlightjs
disableShare: true
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
#cover:
#    image: "<image path/url>" # image path/url
#   alt: "<alt text>" # alt text
#    caption: "<text>" # display caption under cover
#    relative: false # when using page bundles set this to true
#    hidden: true # only hide on current single page
#editPost:
#    URL: "https://github.com/<path_to_repo>/content"
#    Text: "Suggest Changes" # edit text
#    appendFilePath: true # to append file path to Edit link
#
---


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

