---
layout: post
title:  "Buffer Bloat Begone"
date:   2022-07-27 20:00:00 -07:00
categories: bufferbloat network 
tags:
- routeros
- cake
- mikrotik
---

# Buffer Bloat
Come aboard the Buffer Bloat Begone Boat and let us set sail for seas of lower latency under load! 

## The journey begins
I became interested in [buffer bloat](https://www.bufferbloat.net/projects/bloat/wiki/What_can_I_do_about_Bufferbloat/) after [watching a podcast interview](https://www.reddit.com/r/Starlink/comments/okmx3x/bufferbloat_and_beyond_talking_with_dave_taht/) with David Taht.
At a friend's suggestion, I purchased an inexpensive Mikrotik RB750Gr3 device, in order to implement a more robust and fully featured network router for our home network

## Equipment
We obtain internet via a Wireless ISP over a 5Ghz link to a tower 18 miles away, with a plan for 20Mbps/5Mbps. In reality, it's more like 13Mbps/4Mbps. The WISP modem is connected to an RB750Gr3 aka HEx, running RouterOS 7.3.1. Internal DNS and DHCP for the home network is provided by a pihole connected directly to the router.

The wireless home network is a mesh Wifi system made up of a Netgear Orbi RBR50 in AP Mode and two RBS50 satellites, all running Voxel firmware.

## RouterOS Config
The configuration is default out of the box "initial walk through", except for the changes below.

### Fasttrack disabled
```
[admin@RouterOS] > /ip firewall filter
[admin@RouterOS] /ip/firewall/filter> print
Flags: X - disabled, I - invalid; D - dynamic
...
10    chain=forward action=fasttrack-connection hw-offload=yes connection-state=established,related
[admin@RouterOS] /ip/firewall/filter> disable 10
[admin@RouterOS] /ip/firewall/filter> print
...
10 X  chain=forward action=fasttrack-connection hw-offload=yes connection-state=established,related
[admin@RouterOS] /ip/firewall/filter>
```
### Cake Queue config
Adapted from [this post from jbl42](https://forum.mikrotik.com/viewtopic.php?t=179307#p937633)

```
/queue type
add name=cake-WAN-tx kind=cake cake-diffserv=diffserv3  cake-flowmode=dual-srchost cake-nat=yes 
add name=cake-WAN-rx kind=cake cake-diffserv=besteffort cake-flowmode=dual-dsthost cake-nat=yes
/queue simple
add max-limit=12M/3.8M name=queue1 queue=cake-WAN-rx/cake-WAN-tx target=INTERNET
```

## Tests results
| DateTime | Down Mbps | Up Mbps | Latency Unloaded | Latency Download | Latency Upload | Link To Test |
| ------------------ | ---- | ---- | -- | -- | --- | --------
| 7/10/2022 22:00:00 | 9.21 | 4.06 | 31 | +40 | +507 | [Grade F](https://www.waveform.com/tools/bufferbloat?test-id=6497c52c-9b2d-40fd-bb27-adc7a13c2983)
| 7/17/2022 21:27:00 | 13.2 | 4.02 | 45 | +56 | +134 | [Grade C](https://www.waveform.com/tools/bufferbloat?test-id=1b92ac5b-554a-42da-ab22-06ec0034a36c)
| 7/19/2022 22:08:00 | 12.8 | 4.15 | 32 | +64 | +123 | [Grade C](https://www.waveform.com/tools/bufferbloat?test-id=6a003bb7-78b1-4bc9-9c78-951749a7db87)

....queue activated....

| DateTime | Down Mbps | Up Mbps | Latency Unloaded | Latency Download | Latency Upload | Link To Test |
| ------------------ | ---- | ---- | -- | -- | --- | --------
| 7/19/2022 22:19:00 | 11.4 | 3.6  | 33 | +0 | +0 | [Grade A](https://www.waveform.com/tools/bufferbloat?test-id=a21c612a-de90-4ea7-b861-c7fbffc75b21)
| 7/19/2022 23:25:00 | 11.4 | 3.46 | 39 | +0 | +0 | [Grade A](https://www.waveform.com/tools/bufferbloat?test-id=e04d2c96-5fb9-4a5b-944e-7a3689749677)

## Further CAKE Queue config adjusting

After a bit of consideration, I removed the first queue and created a new one that had `max-limit` values just under what the WISP claims we get.

```
[admin@RouterOS] > /queue simple
[admin@RouterOS] /queue/simple> print
Flags: X - disabled, I - invalid; D - dynamic 
 0    name="queue1" target=INTERNET parent=none packet-marks="" priority=8/8 queue=cake-WAN-rx/cake-WAN-tx limit-at=0/0 max-limit=12M/3800k burst-limit=0/0 
      burst-threshold=0/0 burst-time=0s/0s bucket-size=0.1/0.1 
[admin@RouterOS] /queue/simple> remove 0
[admin@RouterOS] /queue/simple> add max-limit=24M/4.8M name=queue1 queue=cake-WAN-rx/cake-WAN-tx target=INTERNET
[admin@RouterOS] /queue/simple>
```

This did not result in any sort of improvement, so I set `max-limit` back to 12M/3.8M

| DateTime | Down Mbps | Up Mbps | Latency Unloaded | Latency Download | Latency Upload | Link To Test |
| ------------------ | ---- | ---- | -- | -- | --- | --------
| 7/19/2022 23:35:00 | 13.1 | 4  | 41 | +64 | +177 | [Grade C](https://www.waveform.com/tools/bufferbloat?test-id=e9b977c6-1513-4e53-a3fa-6c9c0ce61dd5)
