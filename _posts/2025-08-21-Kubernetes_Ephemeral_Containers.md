---
layout: post
title: "Kubernetes Ephemeral Containers for Debugging and Troubleshooting"
date: 2025-08-20 21:00:00 -07:00
categories: kubernetes
tags:
- kubernetes
- containers
- aks
---

I recently had to run a vendor's shell script to patch a solution running in Azure Kubernetes Services. In order to do this quickly and efficiently, I used an Ephemeral container based 

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