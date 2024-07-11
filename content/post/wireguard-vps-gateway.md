+++
title = 'WireGuard VPS as Reverse Proxy and Gateway to Home Network'
date = 2023-08-23T12:20:06-04:00
draft = false

description = "Navigate around CGNAT by securely exposing your homelab services with a reverse proxy. Additionally, access your home network from anywhere using WireGuard. This setup requires only an affordable VPS hosting a public IP, coupled with a WireGuard connection established within your home network."
tags = ["Homelab", "WireGuard", "VPS", "Reverse Proxy", "Networking"]
categories = ["Homelab"]

#image = "/image.jpg" #featured image
comments = false
math = false
toc = true
readingTime = true
+++



## Why use this over services like Tailscale, Zerotier, etc?

WireGuard through a VPS will give you  
- More control over your network and where your data goes.
- Infrastructure independence; you will not need to rely on proprietary servers that could go down at any time.
- Host on VPS of your choosing and get a public IP for your homelab.

Also, it's _extremely_ fast. It can easily transfer over 500 mbit/s while only adding a few milliseconds of latency.
This website is currently running through the setup and I am able to establish a connection through the VPS (hosted about 100 miles away) and back to my home network with less than 20ms of latency. I even host game servers with this setup and have friends connect from across the country without issue.

## Network Diagram

```less
   +-------------+
   |  Internet   |
   +-------------+
        |
        |
   +------------+      +------------+
   |   Public   |      |  Public    |
   |    Users   |<---->|    VPS     |  
   +------------+      +------------+
                          |      |
                          |      | 
                          |      | Public Reverse Proxy 
   +-------------+        |      | (for specific, forwarded services)
   |   CGNAT     |        |      
   +-------------+        | Secure WireGuard Network
        |                 | (Access entire home network
        |                 | if connected through WireGuard)
   +------------+         | 
   |   Home     |<--------+
   | WireGuard  |
   |  Server    |
   +------------+
        |
   +-------------+
   | Home Network|
   | & Services  |
   +-------------+
```
## Requirments
 - Device or VM on home network with Wireguard
 - VPS with public IP, Wireguard, and a reverse proxy of choice.  
   (Nginx Proxy Manager is great for most basic uses)

## WireGuard Configs

### VPS Gateway

First, we have the WireGuard config for the VPS. The config has IP rules that route any traffic coming in from the WireGuard interface to go back out of the WireGuard interface. This is what allows you to connect to the rest of your home network, as well as the internet through your home network. It also adds an optional route to your home network. This route is for the VPS itself to route to your home network through WireGuard, and should be used for if you have a public reverse proxy forwarding from your home network. Finally, it has your home WireGuard server as a peer, along with any other devices you'd like on the network.

Your VPS may or may not have a firewall setup by default, but I recommend using UFW and only allowing connections on the ports for WireGuard and your reverse proxy.

```toml
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

# Example: laptop client
[Peer]
PublicKey = ####
PresharedKey = ####
# Unique IP with /32 subnet (will only route packets for that ip to that device)
AllowedIPs = 10.0.0.5/32 
PersistentKeepalive = 25

```
> **NOTE:  The Wireguard network and your home network must be on different subnets!** Custom routing and masquarading rules will allow you to connect to home network through Wireguard. These rules were adapted from adapted from [Multi Hop Wireguard](https://www.procustodibus.com/blog/2022/06/multi-hop-wireguard/) by [Justin Ludwig](https://www.procustodibus.com/authors/justin-ludwig/).

### Home Server

Your home WireGuard server takes all traffic coming in and marks it with a tag. Packets with this tag are then masqueraded out through your servers default gateway (your home network or home router). This allows you to connect to other devices on the home network as well as the internet through your home network, such that any devices connected through WireGuard will share the same public IP as your home network.

```toml
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

## Nginx Reverse Proxy

Finally, you may want to set up a reverse proxy. This will allow you to connect to services at home through a public domain as well as forward port streams directly, such as for a game server.

I would highly recommend using Nginx Proxy Manager (NPM). Use the following Docker Compose to get it running very quick.

```yaml
version: '3.8'
services:
  nginx:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: proxy
    restart: unless-stopped
    network_mode: "host"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
      - /var/log/nginx:/var/log/nginx
```

From the NPM web interface you can add domains and requests to those domain are forwarded to the specified IP address and port of a service on your home network. Now just point that domain towards your public VPS and you can connect to that domain from anywhere!