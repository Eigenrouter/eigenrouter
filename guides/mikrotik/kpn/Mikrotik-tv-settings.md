---
title: Mikrotik TV configuration
description: How to Setup KPN IPTV on Mikrotik
published: true
date: 2023-04-23T15:53:57.024Z
tags: 
editor: markdown
dateCreated: 2023-04-16T17:30:49.328Z
---

# Mikrotik tv settings

This guide explains how to add KPN IPTV to your router configuration. Note that we are assuming that you followed our tutorial on how to set up internet on your Mikrotik. If your configuration differs, change the settings below to match your configuration. If you are unsure how to do this, feel free to ask about it in our Discord server. The reference internet configuration can be found here: [Mikrotik-Internet-only](/guides/mikrotik/kpn/Mikrotik-Internet-only.md)

Before you begin, if you are running Routeros 6, make sure the extra "multicast" package is installed. If you are running Routeros 7, skip this step. Routeros 6 does not include the IGMP Proxy by default.

Add a VLAN interface with VLAN id 4 and add it to ether1. We are assuming that ether1 is the port that connects to your NTU or (bridged) Modem. 
```
/interface vlan
add interface=ether1 name=vlan1.4 vlan-id=4
```

Turn on *igmp-snooping* on the bridge interface. The command below adds a bridge. If you already have a bridge that you want to use for this, only set igmp-snooping on it to yes.
```
/interface bridge
add arp=proxy-arp igmp-snooping=yes name=local protocol-mode=none
```

Add DHCP Options
```
/ip dhcp-server option
add code=60 name=option60-vendorclass value="'IPTV_RG'"
add code=28 name=option28-broadcast value="'10.0.0.255'"

/ip dhcp-server option sets
add name=IPTV options=option60-vendorclass,option28-broadcast
```

Configure the dhcp client and its options
```
/ip dhcp-client option
add code=60 name=option60-vendorclass value="'IPTV_RG'"
/ip dhcp-client
add add-default-route=special-classless default-route-distance=210 dhcp-options=option60-vendorclass disabled=no \
    interface=vlan1.4 use-peer-dns=no use-peer-ntp=no
```

Configure the IGMP Proxy
```
/routing igmp-proxy interface
add alternative-subnets=0.0.0.0/0 interface=vlan1.4 upstream=yes
add interface=local

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
add action=accept chain=input comment="Optional: Allow UDP input for multicast" dst-address=224.0.0.0/4 in-interface=vlan1.4 protocol=udp
add action=accept chain=forward comment="Optional: Allow IGMP proxy" dst-address=224.0.0.0/4 in-interface=vlan1.4 protocol=udp
```
The last two rules are marked as 'Optional', these are not always needed but depend on the rest of your firewall config (e.g. whether or not you accept or drop input or forward from the vlan1.4).
The DHCP options for the existing DHCP Server don't "just work", you have to select them in the DHCP Network. See IP > DHCP Server > Networks and double click your existing network. Add the option set to it with the dropdown menu.

Note that IPTV will only work on ports that you add to the bridge interface "local" in Bridge > Ports. Not on switches behind the Mikrotik router. Keep this in mind when budgetting interfaces to client devices or switches or any other device that might need an ethernet connection. 
