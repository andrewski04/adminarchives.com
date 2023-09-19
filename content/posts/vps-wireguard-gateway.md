---
title: "Wireguard VPS as Reverse Proxy and Gateway to Home Network"
date: 2023-08-23T12:20:06-04:00
draft: true

# weight: 1
# aliases: ["/first"]
tags: ["Homelab", "Wireguard", "VPS", "Reverse Proxy", "Networking"]
author: "Andrew Houser"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Connect to home network and forward public services through VPS with Wireguard. Bypasses CGNAT and avoids directly exposing network."
canonicalURL: "https://andrew-houser.com/posts/vps-wireguard-gateway/"
disableHLJS: false # to disable highlightjs
disableShare: true
hideSummary: false
searchHidden: true
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

> **NOTE:**  I soon plan to put more time into these articles including additional details and specific configs, as well as explaining how I set everything up. If you have any questions or would like to contribute to an article, please contact me at _andrew@andrew-houser.com_.

## Network Diagram

![diagram](/vps-wg-diagram.png)

## Requirments
 - Device or VM on home network with Wireguard
 - VPS with public ip, Wireguard, and reverse proxy of choice.  
   (Nginx Proxy Manager is great for most basic uses.)

## Wireguard Configs

> **NOTE:  The Wireguard network and your home network must be on different subnets!** Custom routing and masquarading rules will allow you to connect to home network through Wireguard. These rules were adapted from adapted from [Multi Hop Wireguard](https://www.procustodibus.com/blog/2022/06/multi-hop-wireguard/) by [Justin Ludwig](https://www.procustodibus.com/authors/justin-ludwig/).

### VPS Gateway


```yaml
[Interface]
# Use unique WG subnet
Address = 10.0.0.1/24  
ListenPort = 51820
PrivateKey = ####


# Route incoming WG traffic out of WG interface (Forward to home WG server)
Table = 123
PreUp = sysctl -w net.ipv4.ip_forward=1
PreUp = ip rule add iif wg0 table 123 priority 456
PostDown = ip rule del iif wg0 table 123 priority 456

PreUp = sysctl -w net.ipv6.conf.all.forwarding=1
PreUp = ip -6 rule add iif wg0 table 123 priority 456
PostDown = ip -6 rule del iif wg0 table 123 priority 456

# Make this your home subnet. Custom route for traffic from VPS/reverse proxy

PostUp = ip route add 192.168.1.1/24 dev wg0

# Home WG server. Use keep alive behind NAT.
[Peer]
PublicKey = ####
PresharedKey = ####
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25

# Add other clients here
```

### Home Server


```yaml
[Interface]
PrivateKey = ####
Address = 10.0.0.2/24

# Masquerade traffic over default network (Home lan/wan)
PreUp = sysctl -w net.ipv4.ip_forward=1
PreUp = iptables -t mangle -A PREROUTING -i wg0 -j MARK --set-mark 0x30
PreUp = iptables -t nat -A POSTROUTING ! -o wg0 -m mark --mark 0x30 -j MASQUERADE
PostDown = iptables -t mangle -D PREROUTING -i wg0 -j MARK --set-mark 0x30
PostDown = iptables -t nat -D POSTROUTING ! -o wg0 -m mark --mark 0x30 -j MASQUERADE

PreUp = sysctl -w net.ipv6.conf.all.forwarding=1
PreUp = ip6tables -t mangle -A PREROUTING -i wg0 -j MARK --set-mark 0x30
PreUp = ip6tables -t nat -A POSTROUTING ! -o wg0 -m mark --mark 0x30 -j MASQUERADE
PostDown = ip6tables -t mangle -D PREROUTING -i wg0 -j MARK --set-mark 0x30
PostDown = ip6tables -t nat -D POSTROUTING ! -o wg0 -m mark --mark 0x30 -j MASQUERADE

# Public VPS gateway
[Peer]
PublicKey = ####
PresharedKey = ####
Endpoint = ##.##.##.##:51820
AllowedIPs = 10.0.0.1/24
PersistentKeepalive = 25
```
