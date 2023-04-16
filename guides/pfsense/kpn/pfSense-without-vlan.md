---
title: KPN without seperate TV VLAN
description: How to set up KPN on pfSense without a seperate TV VLAN
published: true
date: 2023-04-16T17:22:31.264Z
tags: 
editor: markdown
dateCreated: 2023-04-16T14:54:09.153Z
---

### How to Setup KPN on pfSense without a seperate TV VLAN

After installing pfSense go to:

```
Interface > Assignments > Vlans
```
![vlans](/images/kpn/pfsense-without-vlan/vlans.png)

Create 2 VLANs *vlan6* & *vlan4* on your WAN interface, then go to:

```
Interface > Assignments > PPP
```

Add a new interface and link it to you WAN port.

![PPP](/images/kpn/pfsense-without-vlan/ppp.png)

Then go to: 

```
Interface > Assignments 
```
![assignments](/images/kpn/pfsense-without-vlan/assignments.png)

This is where you set your PPPOE interface as WAN.  
Set Lan as your lan interface and assign a new interface (Vlan4) to KPN tv

Then go to:

```
Interface > TV_KPN
```
set this at _Send options_
```
dhcp-class-identifier "IPTV_RG"
```
and set this at _Request options_
```
subnet-mask,routers,classless-routes
```

Save the settings and go to:

```
Services > IGMP Proxy
```

![igmpproxy](/images/kpn/pfsense-without-vlan/igmpproxy.png)

Enable IGMP

Add an upstream and downstream proxy. The upstream proxy takes the default route: 0.0.0.0/0 (or on pfSense: 0.0.0.0/1 , 128.0.0.0/1)
Add your LAN network to the downstream proxy. 

Then go to:

```
Services > DHCP Server > LAN
```

Scroll to the bottom and open *Additional BOOTP/DHCP Options*
and enter the following lines

![bootp](/images/kpn/pfsense-without-vlan/additional-bootp-dhcp.png)

In this example is the broadcast adress 10.0.0.255 set here your own broadcast address

Then go to:

```
Firewall > NAT > Outbound
```

![bootp](/images/kpn/pfsense-without-vlan/outbound.png)

And add the line above under mappings

Then go to

```
Firewall > Rules > WAN
```

Add the following lines

![ruleswan](/images/kpn/pfsense-without-vlan/rules-wan.png)

On the IPv4 IGMP rule enable the following line.
_Allow IP options_

![ipoptions](/images/kpn/pfsense-without-vlan/ipoptions.png)

Then go to:

```
Firewall Rules LAN
```

and add the following lines

![fwlan](/images/kpn/pfsense-without-vlan/firewalllan.png)

On the IPv4 IGMP rule _Allow IP options_

![ipoptions](/images/kpn/pfsense-without-vlan/ipoptions.png)

The _213.75.112.0/21_ is the ip/subnet from kpn itv

Then go to:

```
Firewall > Rules > TV_KPN
```

And add the following lines

![tvkpn](/images/kpn/pfsense-without-vlan/tv_kpn.png)

On the IPv4 IGMP rule _Allow IP options_

![ipoptions](/images/kpn/pfsense-without-vlan/ipoptions.png)