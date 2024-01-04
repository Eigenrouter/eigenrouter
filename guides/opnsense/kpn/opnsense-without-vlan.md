---
title: OPNsense without seperate TV VLAN
description: How to Setup KPN on OPNsense without a seperate TV VLAN
published: true
date: 2024-01-04T17:27:02.080Z
tags: 
editor: markdown
dateCreated: 2023-04-16T18:13:46.292Z
---

# How to Setup KPN on OPNsense with seperate TV VLAN

In this How-To we are going to setup KPN on OPNsense on with iTV on the LAN interface.

Where we use ```192.168.2.0/24``` as our network, OPNsense admin can be visited on ```192.168.2.1```.

### Step. 1

After install OPNsense go to

```
System > Firmware > Status
```

![FWupdate](/images/kpn/opnsense-without-vlan/fwupdate.png)

Click ```Check for updates``` if there are updates available scroll down the page and install them.

### Step. 2

Go to

```
Interfaces > Other Types > VLAN
```

![VLANs](/images/kpn/opnsense-without-vlan/vlans.png)

Create 2 VLANs on your WAN interface:

```VLAN6```

```VLAN4```


### Step. 3
Go to

```
Interfaces > Assignments
```

![Assignments](/images/kpn/opnsense-without-vlan/assignments.png)

Create a interface with VLAN tag: ```Parent: vtnet0 (wan), Tag: 4``` call it ```IPTV_WAN```

Change the WAN interface to ```Parent: vtnet0 (wan), Tag: 6```

Save now.

### Step. 4

Go to

```
Interfaces > WAN
```

![InterfaceWAN](/images/kpn/opnsense-without-vlan/interfacewan.png)

**Generic configuration**

Set IPv4 Configuration Type: ```PPPoE```

Set IPv6 Configuration Type: ```DHCPv6```

Set MTU: ```1492```

**PPPoE configuration**

Set Username: ```kpn@internet``` 

Set Password: ```kpn``` 

**DHCPv6 client configuration**

Set Request only an IPv6 prefix: ```True```

Set Prefix delegation size: ```48```

Set Send IPv6 prefix hint: ```False```

Set Use IPv4 connectivity: ```True```

Set Use VLAN priority: ```Disabled```

### Step. 5

Go to

```
Interfaces > IPTV_WAN
```

![InterfaceWANIPTV](/images/kpn/opnsense-without-vlan/interfacewaniptv.png)

**Basic configuration**

Set Enable: ```True``` Enable Interface

Set Description: ```IPTV_WAN```

**Generic configuration**

Set IPv4 Configuration Type: ```DHCP```

**DHCP client configuration**

Set Configuration Mode: ```Advanced```

Set Override MTU: ```True```

Set Lease Requirements / Send Options: ```dhcp-class-identifier "IPTV_RG"```

Set Lease Requirements / Request Options: ```subnet-mask, routers, broadcast-address, classless-routes```

### Step. 6

Go to

```
System > Firmware > Plugins
```

![Plugins](/images/kpn/opnsense-without-vlan/plugins.png)

Install ```os-igmp-proxy```

After installation of ```os-igmp-proxy``` refresh the page.

### Step. 7

Go to

```
Services > IGMP Proxy
```

![IGMPproxy](/images/kpn/opnsense-without-vlan/igmpproxy.png)

Here we are going to add 2 streams (Upstream and Downstream)

**Add Upstream**

Set Interface: ```IPTV_WAN```

Set Type: ```Upstream Interface```

Create 2 networks: ```0.0.0.0 / 1``` & ```128.0.0.0 / 1```

**Add Downstream**

Set Interface: ```LAN```

Set Type: ```Downstream Interface```

Create 1 networks: ```192.168.2.0 / 24```

### Step. 8

Go to

```
Services > DHCPv4
```

![DHCPv4](/images/kpn/opnsense-without-vlan/dhcp4.png)

![DHCPv4](/images/kpn/opnsense-without-vlan/dhcp4additional.png)

Set Additional Options
```
60 / Text / IPTV_RG
```
```
28 / IP address or host / 192.168.2.255
```

### Step. 9

Go to

```
Firewall > NAT > Outbound
```

![FWNATOutbound](/images/kpn/opnsense-without-vlan/fwnatoutbound.png)

Set Mode: ```Hybrid outbound NAT rule generation (automatically generated rules are applied after manual rules)```

![FWNATOutboundRule](/images/kpn/opnsense-without-vlan/fwnatoutboundrule.png)

Create Rule: ```IPTV_WAN LAN net * * * IPTV_WAN address * NO```

### Step. 10

Go to

```
Firewall > Rules > IPTV_WAN
```
![FWIPTVWAN](/images/kpn/opnsense-without-vlan/fwiptvwan.png)

Create 3 rules

**Rule 1:**

Set Action: ```Pass```

Set Quick: ```True```

Set Interface: ```IPTV_WAN```

Set Direction: ```in```

Set TCP/IP Version: ```IPv4```

Set Protocol: ```IGMP```

Set Source: ```any```

Set Destination: ```Single host or Network``` ```224.0.0.0 / 4```

Set Advanced Option: ``Show``

Set allow options: ```True```

**Rule 2:**

Set Action: ```Pass```

Set Quick: ```True```

Set Interface: ```IPTV_WAN```

Set Direction: ```out```

Set TCP/IP Version: ```IPv4```

Set Protocol: ```IGMP```

Set Source: ```any```

Set Destination: ```Single host or Network``` ```224.0.0.0 / 4```

Set Advanced Option: ``Show``

Set allow options: ```True```

**Rule 3:**

Set Action: ```Pass```

Set Quick: ```True```

Set Interface: ```IPTV_WAN```

Set Direction: ```in```

Set TCP/IP Version: ```IPv4```

Set Protocol: ```UDP```

Set Source: ```any```

Set Destination: ```Single host or Network``` ```224.0.0.0 / 4```

### Step. 11

Go to

```
Firewall > Rules > LAN
```

![FWLAN](/images/kpn/opnsense-without-vlan/fwlan.png)

Create 2 rules at top of the exisiting ones.

**Rule 1:**

Set Action: ```Pass```

Set Quick: ```True```

Set Interface: ```LAN```

Set Direction: ```in```

Set TCP/IP Version: ```IPv4```

Set Protocol: ```IGMP```

Set Source: ```LAN net```

Set Destination: ```Single host or Network``` ```224.0.0.0 / 4```

Set Advanced Option: ``Show``

Set allow options: ```True```

**Rule 2:**

Set Action: ```Pass```

Set Quick: ```True```

Set Interface: ```LAN```

Set Direction: ```in```

Set TCP/IP Version: ```IPv4```

Set Protocol: ```any```

Set Source: ```LAN net```

Set Destination: ```Single host or Network``` ```213.75.112.0 / 21```
