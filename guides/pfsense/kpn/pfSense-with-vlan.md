---
title: KPN with seperate TV VLAN
description: How to Setup KPN on pfSense with seperate TV VLAN
published: true
date: 2023-04-23T15:59:18.082Z
tags: 
editor: markdown
dateCreated: 2023-04-16T16:46:07.042Z
---

# How to Setup KPN on pfSense with seperate TV VLAN

In this How-To we are going to use ```VLAN89``` as our seperate TV VLAN, you can choose to use your own VLAN number.

### Step. 1

After install pfSense go to

```
Interface > Assignments > VLANs
```

![vlan](/images/kpn/pfsense-with-vlan/interfacevlans.png)

Create 2 VLANs on your WAN interface:
```VLAN6```
```VLAN4```

Create 1 VLAN on your LAN interface:
```VLAN89```

### Step. 2

Go to

```
Interface > Assignments > PPPs
```

Add a new interface and link it with your WAN port

![ppps](/images/kpn/pfsense-with-vlan/ppp.png)

Username 
```kpn@internet```

Password 
```kpn```

### Step. 3

Go to 

```
Interface > Assignments 
```

![interfaces](/images/kpn/pfsense-with-vlan/interfaces.png)

Set Interface WAN to PPPOE0

Add ```VLAN4``` for ```IPTV_WAN```

Add ```VLAN89``` for ```IPTV_VLAN89 ```

### Step. 4

Edit Interface IPTV_WAN

![vlan89_1](/images/kpn/pfsense-with-vlan/leaserequirementsandrequests.png)

Set Send options:
```
dhcp-class-identifier "IPTV_RG"
```

Set Request options:
```
subnet-mask, routers, broadcast-address, classless-routes
```

### Step. 5

Edit Interface IPTV_VLAN89

![vlan89_1](/images/kpn/pfsense-with-vlan/interfacevlaniptv.png)

IPv4 Configuration Type: 
```Static IPv4```

IPv4 Address: 
```192.168.89.1```

### Step. 6

Go to

```
Services > DHCP Server > IPTV_VLAN89
```

![dhcpvlan89](/images/kpn/pfsense-with-vlan/dhcpiptvvlan.png)

Set Range:
```192.168.89.10 / 192.168.89.245```
(*Or use your own range)

![dhcpvlan89dns](/images/kpn/pfsense-with-vlan/dhcpiptvvlandns.png)

Set DNS Servers:
```195.121.1.34``` 
```195.121.1.66```

![dhcpvlan89other](/images/kpn/pfsense-with-vlan/dhcpiptvvlanother.png)

Set Domain name:
```kpn.home```

![dhcpvlan89bootp](/images/kpn/pfsense-with-vlan/dhcpiptvvlanbootp.png)

Set BOOTP/DHCP Options 
```
60 / Text / IPTV_RG
```
```
28 / IP address or host / 192.168.89.255
```

### Step. 7

Go to

```
Services > IGMP Proxy
```

<img width="1157" alt="Schermafbeelding 2023-04-30 om 07 55 11" src="https://user-images.githubusercontent.com/132186624/235338160-8e713c2e-ceff-4cbc-b1c7-7744efdfb100.png">

Here we are going to add 2 streams (Upstream and Downstream)

```
Add Upstream
```

Set Interface: ```IPTV_WAN```

Set Type: ```Upstream Interface```

Create 1 network: ```0.0.0.0 / 0 ``` 



```
Add Downstream
```
Set Interface: ```IPTV_VLAN89```

Set Type: ```Downstream Interface```

Create 1 networks: ```192.168.89.0 / 24```

pfsense will give an error message, which is a result of a bug. Please ignore the message.

### Step. 8

Go to

```
Firewall > NAT > Outbound
```

![natoutbound](/images/kpn/pfsense-with-vlan/natoutbound.png)

Set to Hybrid Outbound NAT

Create a Mapping

<img width="961" alt="Schermafbeelding 2023-04-23 om 13 50 41" src="https://user-images.githubusercontent.com/132186624/235338398-578bf3c1-3adf-4688-8b77-2f43cc899ce3.png">

### Step. 9

Go to

```
Firewall > Rules > IPTV_WAN
```

Add the following rules

<img width="946" alt="Schermafbeelding 2023-04-23 om 13 52 29" src="https://user-images.githubusercontent.com/132186624/235338423-51e09e00-21c6-46e9-883e-947aa284614b.png">

On the IPv4 IGMP rule enable the following line. 

![allowipoptions](/images/kpn/pfsense-with-vlan/allowipoptions.png)

### Step. 10

Go to

```
Firewall > Rules > IPTV_VLAN89
```

Add the following rules

<img width="1137" alt="Schermafbeelding 2023-04-30 om 07 58 50" src="https://user-images.githubusercontent.com/132186624/235338349-ff82e880-d367-49dd-86f5-9011e3c354a8.png">

On the IPv4 IGMP rule enable the following line.

![allowipoptions](/images/kpn/pfsense-with-vlan/allowipoptions.png)

Use the correct order for the firewall rules. Rulesets on the Interface tabs are evaluated on a first match basis. Allow IGMP and UDP first, and then block acces to LAN net or other interfaces. The last rule must be an allow all on ipv4.
