---
title: Network unreachable in virtualbox centOS host
layout: post
---

# Network unreachable in virtualbox centOS host

*When ethX is not listed upon running:* ifconfig

- check that the HWADDR listed in
1. /etc/udev/rules.d/70-persistent-net.rules		
2. /etc/sysconfig/network-scripts/ifcfg-ethx
- are same or the interface will be ignored and not brought up.
- Now that is is listed and not communicating.

*still stuck here…*
LOL
