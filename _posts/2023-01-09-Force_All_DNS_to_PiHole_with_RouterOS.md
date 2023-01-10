---
layout: post
title:  "Redirect All DNS Requests to PiHole with Router OS"
date:   2023-01-09 21:00:00 -07:00
categories: homenetwork
tags:
- routeros
- mikrotik
- dns
- pihole
---

Our goal is to have all devices on the internal network use an on-premise PiHole server, enforced by a Mikrotik router.

## Devices
- Mikrotik RB750GR3 router 192.168.88.1
- PiHole 192.168.88.2

## Redirect all DNS to Pihole
Below will force all local DNS traffic to the PiHole at `192.168.88.2`.

```
/ip firewall nat
add action=dst-nat chain=dstnat dst-address=!192.168.88.2 dst-port=53 protocol=udp src-address=!192.168.88.2 to-addresses=192.168.88.2
add action=dst-nat chain=dstnat dst-address=!192.168.88.2 dst-port=53 protocol=tcp src-address=!192.168.88.2 to-addresses=192.168.88.2
add action=masquerade chain=srcnat dst-address=192.168.88.2 dst-port=53 protocol=udp src-address=192.168.88.0/24
add action=masquerade chain=srcnat dst-address=192.168.88.2 dst-port=53 protocol=tcp src-address=192.168.88.0/24
```

## References
[Erik Thiart](https://erikthiart.com/blog/force-all-dns-traffic-to-go-through-pi-hole-using-mikrotik)
[Mikrotik](https://forum.mikrotik.com/viewtopic.php?t=189188)