---
layout: post
title:  "Simple Queues to Prioritize Traffic"
date:   2022-08-11 20:00:00 -07:00
categories: network 
tags:
- routeros
- cake
- mikrotik
---

After [eviscerating buffer bloat](https://joeywas.github.io/bufferbloat/network/2022/07/28/Buffer_Bloat_Begone.html) with a single simple queue using [CAKE](https://www.bufferbloat.net/projects/codel/wiki/CakeTechnical/), the next goal is to treat higher priority traffic with, well, higher priority than other "lower priority" traffic. For our purposes, higher priority traffic is Microsoft Teams, T-Mobile WiFi calling, and SSH. Lower priority traffic includes general web browsing and video streaming on Netflix, Hulu, etc.

## Equipment
Internet provided via a Wireless ISP that is approximately 20Mbps/5Mbps. The WISP modem is connected to a Mikrotik router RB750Gr3 aka HEx, running RouterOS 7.3.1. Internal DNS and DHCP are provided by a pihole connected directly to the router.

The wireless home network is a mesh Wifi system made up of a Netgear Orbi RBR50 in AP Mode and two RBS50 satellites, all running Voxel firmware. 

## Address Lists
In order to mark packets for Microsoft Teams, we create an address list based off the [Optimize Required section of Microsoft 365 endpoints page](https://docs.microsoft.com/en-us/microsoft-365/enterprise/urls-and-ip-address-ranges?view=o365-worldwide#skype-for-business-online-and-microsoft-teams). 
```
/ip firewall address-list
add address=52.120.0.0/14 list=M365TEAMSLIST
add address=52.112.0.0/14 list=M365TEAMSLIST
add address=13.107.64.0/18 list=M365TEAMSLIST
```

## Mangle Marking 
Packets must be marked in order to be assigned to a queue. This is done with [Mangle in RouterOS](https://help.mikrotik.com/docs/display/ROS/Mangle). 

### Fasttrack disabled
In order to use mangle marking and queues, Fasttrack must be disabled first

```
/ip firewall filter
print

Flags: X - disabled, I - invalid; D - dynamic
...
10    chain=forward action=fasttrack-connection hw-offload=yes connection-state=established,related

disable 10
print

Flags: X - disabled, I - invalid; D - dynamic
...
10 X  chain=forward action=fasttrack-connection hw-offload=yes connection-state=established,related
```

### Marking WAN packets
Mark ingress packets from the internet to the LAN.

```
/ip firewall mangle
add action=mark-packet chain=prerouting comment=mark-WAN in-interface-list=WAN new-packet-mark=mark-WAN passthrough=yes
add action=mark-packet chain=prerouting comment=mark-WAN-SSH new-packet-mark=mark-WAN-SSH packet-mark=mark-WAN passthrough=no protocol=tcp src-port=22
add action=mark-packet chain=prerouting comment=mark-WAN-Tmobile new-packet-mark=mark-WAN-Tmobile packet-mark=mark-WAN passthrough=no protocol=udp src-port=500,4500,5061
add action=mark-packet chain=forward comment=mark-WAN-Teams new-packet-mark=mark-WAN-Teams packet-mark=mark-WAN passthrough=no protocol=udp src-address-list=M365TEAMSLIST
add action=mark-packet chain=prerouting comment=mark-WAN new-packet-mark=mark-WAN packet-mark=mark-WAN passthrough=no
```

### Marking LAN packets
Mark egress packets from the LAN to the internet.

```
/ip firewall mangle
add action=mark-packet chain=prerouting comment=mark-LAN in-interface-list=LAN new-packet-mark=mark-LAN passthrough=yes
add action=mark-packet chain=prerouting comment=mark-LAN-SSH dst-port=22 new-packet-mark=mark-LAN-SSH packet-mark=mark-LAN passthrough=no protocol=tcp
add action=mark-packet chain=prerouting comment=mark-LAN-Tmobile dst-port=500,4500,5061 new-packet-mark=mark-LAN-Tmobile packet-mark=mark-LAN passthrough=no protocol=udp
add action=mark-packet chain=forward comment=mark-LAN-Teams dst-address-list=M365TEAMSLIST new-packet-mark=mark-LAN-Teams packet-mark=mark-LAN passthrough=no protocol=udp
add action=mark-packet chain=prerouting comment=mark-LAN new-packet-mark=mark-LAN packet-mark=mark-LAN passthrough=no
```

## Queue Type
Create two cake queue types. The `cake-dual-dst` type will be used for ingress traffic to the LAN from the internet. The `cake-dual-src` type will be used for egress traffic from the LAN to the internet.
The memlimits were tuned to provide lowest latency during waveform tests.

```
/queue type
add cake-flowmode=dual-dsthost cake-memlimit=195.3KiB cake-nat=yes kind=cake name=cake-dual-dst
add cake-flowmode=dual-srchost cake-memlimit=48.8KiB cake-nat=yes kind=cake name=cake-dual-src
```

## Tiered Simple Queue

The top level queue `01-WAN-Bandwidth` is created first and targets the ethernet interface that is connected to the ISP modem. This top level queue is limited at 20M download and 5M upload, which is what we get from the ISP.

```
/queue simple
add max-limit=20M/5M name=01-WAN-Bandwidth packet-marks="mark-WAN,mark-WAN-SSH,mark-WAN-Tmobile,mark-WAN-Teams,mark-LAN,mark-LAN-SSH,mark-LAN-Tmobile,mark-LAN-Teams" queue=cake-dual-dst/cake-dual-src target=ether1
```

The next level queues are prefixed with `02` and have the `01-WAN-Bandwidth` queue as a parent.
We guarrantee the following bandwidth:
  - 512k/512k for SSH with 1M/1M max
  - 512k/512k for Teams with 4M/3500K max
  - 128k/128k for Tmobile wifi calling

The T-Mobile wifi calling queue has the highest priority, followed by the SSH and TEAMS queues. 

The `02-Catchall` queue is for all other network traffic.

```
/queue simple
add limit-at=512k/512k max-limit=1M/1M name=02-SSH packet-marks=mark-WAN-SSH,mark-LAN-SSH parent=01-WAN-Bandwidth priority=7/7 queue=cake-dual-dst/cake-dual-src target=""
add limit-at=512k/512k max-limit=4M/3500k name=02-Teams packet-marks=mark-WAN-Teams,mark-LAN-Teams parent=01-WAN-Bandwidth priority=7/7 queue=cake-dual-dst/cake-dual-src target=""
add limit-at=128k/128k max-limit=512k/512k name=02-Tmobile packet-marks=mark-LAN-Tmobile,mark-WAN-Tmobile parent=01-WAN-Bandwidth priority=6/6 queue=cake-dual-dst/cake-dual-src target=""
add limit-at=15M/3M max-limit=13M/3M name=02-Catchall parent=01-WAN-Bandwidth queue=cake-dual-dst/cake-dual-src target=""
```
