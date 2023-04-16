---
title: Mikrotik TV configuration With VLAN
description: 
published: true
date: 2023-04-16T17:32:48.816Z
tags: 
editor: markdown
dateCreated: 2023-04-16T17:32:48.816Z
---

### Mikrotik-tv-settings

This guide explains how to add KPN IPTV to your router configuration. Note that we are assuming that you followed our tutorial on how to set up internet on your Mikrotik. If your configuration differs, change the settings below to match your configuration. If you are unsure how to do this, feel free to ask about it in our Discord server. The reference internet configuration can be found here: [Mikrotik-Internet-only](/guides/mikrotik/kpn/Mikrotik-Internet-only.md)

Before you begin, if you are running Routeros 6, make sure the extra "multicast" package is installed. If you are running Routeros 7, skip this step. Routeros 6 does not include the IGMP Proxy by default.

Add a VLAN interface with VLAN id 4 and add it to ether1. We are assuming that ether1 is the port that connects to your NTU or (bridged) Modem. We will also add a VLAN interface to our bridge (that we created in the internet-only guide), which will be used for our IPTV devices. 
```
/interface vlan
add comment=TV interface=ether1 name=vlan1.4 vlan-id=4
add comment=tv-lan interface=local name=Vlan3 vlan-id=3
```

Turn on *igmp-snooping* on the bridge interface. The command below adds a bridge. If you already have a bridge that you want to use for this, only set igmp-snooping on it to yes. We will also complete the bridge vlan table and assign an ethernet interface to vlan 3 via PVID. 
```
/interface bridge
add arp=proxy-arp igmp-snooping=yes name=local protocol-mode=none vlan-filtering=yes

/interface bridge vlan
add bridge=local tagged=local vlan-ids=3

/interface bridge port
add bridge=local interface=ether3 pvid=3
```

Set Ip Address for tv-lan network
```
/ip pool
add name=tv-lan ranges=10.0.3.20-10.0.3.254

/ip dhcp-server
add address-pool=tv-lan interface=Vlan3 lease-time=30m name=tv-lan

/ip address
add address=10.0.3.1/24 comment="tv-lan (vlan3)" interface=Vlan3 network=10.0.3.0

/ip dhcp-server network
add address=10.0.3.0/24 dns-server=195.121.1.34,195.121.1.66 domain=tv-lan.local gateway=10.0.3.1
```

Add DHCP Options
```
/ip dhcp-server option
add code=60 name=option60-vendorclass value="'IPTV_RG'"
add code=28 name=option28-broadcast value="'10.0.3.255'"

/ip dhcp-server option sets
add name=IPTV options=option60-vendorclass,option28-broadcast
```

Configure the dhcp client and its options
```
/ip dhcp-client option
add code=60 name=option60-vendorclass value="'IPTV_RG'"
/ip dhcp-client
add add-default-route=special-classless default-route-distance=210 dhcp-options=option60-vendorclass disabled=no interface=vlan1.4 use-peer-dns=no use-peer-ntp=no
```

Configure the IGMP Proxy
```
/routing igmp-proxy interface
add alternative-subnets=0.0.0.0/0 interface=vlan1.4 upstream=yes
add interface=vlan3

/routing igmp-proxy
set quick-leave=yes
```

Add firewall rules
```
/ip firewall nat
add action=masquerade chain=srcnat comment="IPTV" dst-address=213.75.0.0/16 out-interface=vlan1.4
add action=masquerade chain=srcnat comment="IPTV" dst-address=217.166.0.0/16 out-interface=vlan1.4
add action=masquerade chain=srcnat comment="IPTV" dst-address=10.207.0.0/20 out-interface=vlan1.4

/ip firewall filter
add action=accept chain=input comment="IPTV IGMP" dst-address=224.0.0.0/4 in-interface=vlan1.4 protocol=igmp
```

Set Dhcp-option for tv
```
/ip dhcp-server 
network set numbers=1 dhcp-option-set=IPTV
```

The DHCP options for the existing DHCP Server don't "just work", you have to select them in the DHCP Network. See IP > DHCP Server > Networks and double click your existing network. Add the option set to it with the dropdown menu.

Note that IPTV will only work on ports that you add to the bridge interface "local" in Bridge > Ports. Not on switches behind the Mikrotik router. Keep this in mind when budgetting interfaces to client devices or switches or any other device that might need an ethernet connection. 