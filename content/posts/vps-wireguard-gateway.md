---
title: "Wireguard VPS as Reverse Proxy and Gateway to Home Network"
date: 2023-08-23T12:20:06-04:00
draft: true

# weight: 1
# aliases: ["/first"]
tags: ["Homelab", "Wireguard", "VPS", "Reverse Proxy"]
author: "Andrew Houser"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Connect to home network and forward public services through VPS with Wireguard. Bypasses CGNAT and avoids directly exposing network."
canonicalURL: "https://andrew-houser.com/posts/vps-wireguard-gateway/"
disableHLJS: true # to disable highlightjs
disableShare: true
disableHLJS: false
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

### Requirments
 - Device or VM on home network with Wireguard
 - VPS with public ip, Wireguard, and reverse proxy of choice.  
  (Nginx Proxy Manager is great for most basic uses.)

###