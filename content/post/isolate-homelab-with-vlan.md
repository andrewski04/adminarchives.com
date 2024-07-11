+++
title = 'Isolate your homelab using VLANs, Proxmox, and OPNsense'
date = 2023-09-19T12:20:06-04:00
draft = false

description = "Create a secure and efficient isolated homelab on a non-VLAN aware home network using VLAN-aware switches and Proxmox bridges. When paired with Wireguard, you can securely access this network from anywhere. No longer takes down your entire houses network when you mess something up!"
tags = ["Homelab", "VLAN", "Networking"]
categories = ["Homelab", "Proxmox"]

#image = "/image.jpg" #featured image
comments = false
math = false
toc = true
readingTime = true

+++

### Current network map

```less
Internet
    |
Home Router (192.168.1.1/24)
    |------------------- Other devices on Home Network
    |
    | (VLAN 10 Uplink - Untagged)
Switch 
    |
Proxmox Host ------ OPNsense VM (Uplink on VLAN 10, Internal on VLAN 5)
|        |                  |
|  (VLAN 5 & 10)            | (VLAN 5 Internal)
|                           |
|  VMs & Proxmox Host       | VMs on Internal Network
|  (10.0.x.x/16)            | (10.0.x.x/16)
|                           | OPNsense IP: 10.0.0.1
|

```

VLAN 5 acts as the internal network isolated from your home, while VLAN 10 is used as an 'uplink' for OPNsense and any hosts and is untagged for the router.

OPNsense has a network bridge to both VLAN 5 and 10, routing between the VLAN 5 'LAN' network and VLAN 10 'WAN'. 


### Further isolations (3 or more vlans)
```less
Internet
    |
Home Router (192.168.1.1/24)
    |------------------- Other devices on Home Network
    |
    | (VLAN 10 Uplink - Untagged)
Switch 
    |
Proxmox Host ------ OPNsense VM (Uplink on VLAN 10, Internal on VLAN 15)
    |                       |
    | (VLAN 5)              | (VLAN 15 Internal)
    VMs on Management       VMs on Internal Network
    Network (10.0.0.x/24)   (10.15.0.x/24)
```

OPNsense allows you to add optional network adapters as well, giving you the ability to further isolate the setup with further VLANs.
In this example, VLAN 15 will be isolated from your management network. Isolation like this should be used for publicly exposed services.

Using OPNsense firewall rules, you could block VLAN 15 from connecting to devices with IP ranges 10.0.x.x and 192.168.1.x, giving it only access to the internet and needed internal services.
