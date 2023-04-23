---
title: Mikrotik Internet Only
description: How to Setup KPN Internet Only on Mikrotik
published: true
date: 2023-04-23T15:54:22.743Z
tags: 
editor: markdown
dateCreated: 2023-04-16T17:27:19.359Z
---

# Mikrotik internet only

This guide is based on a Mikrotik device with at least two Ethernet ports. We start with a completely empty configuration. To wipe your configuration, connect to your Mikrotik device via Winbox or Webfig (HTTP) and navigate to System > Reset Configuration. Tick “No Default Configuration” and reset the device via the “Reset Configuration” button. The device will now restart.

After wiping the configuration, you can connect to the Mikrotik via MAC-Winbox on Windows or via Wine. We recommend configuring a management network on an Ethernet interface that you will not use for the network that we will create in this guide. The simplest way to do this is to set a unique IP address on for example ether3 (as ether3 will not be used in this guide) and to connect to the device via IP on ether3. Refer to the IP Address section in the steps below to see how to do this.

You can download Winbox here: [ MikroTik Routers and Wireless - Software](https://mikrotik.com/download)

This guide requires that you install the ‘multicast’ package for Routeros version 6. If you’re running Routeros 7, skip this step. 

The default login credentials are “admin” with no password. If you are running the latest version of Routeros 6 or 7, you will be prompted to change the password on first login.

This guide will give you commands to enter in the Mikrotik Terminal. You can do this either via SSH or in the Terminal that is included in the Winbox application.


We will build the WAN connection on ether1, and the LAN connection on ether2. Ether2 will be configured as slave interface of a bridge interface. Add the following:
```
/interface vlan
add interface=ether1 name=vlan1.6 vlan-id=6
```
Now we make the pppoe connection with KPN
```
/interface pppoe-client
add add-default-route=yes allow=pap disabled=no interface=vlan1.6 \
    keepalive-timeout=20 max-mru=1500 max-mtu=1500 name=pppoe-client \
    password=kpn profile=default user=kpn
```
We will now add the bridge interface for the LAN
```
/interface bridge
add name=local arp=proxy-arp
```
Add the necessary ports to your bridge
```
/interface bridge port
add bridge=local interface=ether2
```
Set the gateway address for the LAN. You can change this to your needs.
```
/ip address
add address=10.0.0.1/24 interface=local 
```
Set DNS forwarders. In this example we will be using Google DNS. We will also use the Mikrotik as DNS server for the LAN. 
```
/ip dns
set allow-remote-requests=yes servers=8.8.8.8,8.8.4.4
```
Configure a basic firewall. For proper hardening, refer to this Mikrotik article: [Mikrotik Router Hardening](https://help.mikrotik.com/docs/display/ROS/Building+Your+First+Firewall#BuildingYourFirstFirewall-Ipv4firewall). The following filter rules only cover the input chain on the pppoe-client. This means the LAN side is not hardened. 
```
/ip firewall filter
add action=accept chain=input in-interface=pppoe-client protocol=icmp
add action=accept chain=input connection-state=established,related 
add action=drop chain=input in-interface=pppoe-client 
```
Configure outbound NAT
```
/ip firewall nat
add action=masquerade chain=srcnat out-interface=pppoe-client
```
Configure a DHCP server for the LAN. If you changed anything in the IP Address section above, also change the values here to match. 
```
/ip pool
add name=thuisnetwerk ranges=10.0.0.10-10.0.0.254  
/ip dhcp-server
add address-pool=thuisnetwerk authoritative=yes interface=local \
    lease-time=8h name=dhcp-home
/ip dhcp-server network
add address=10.0.0.0/24 dns-server=10.0.0.1 domain=home.local gateway=\
    10.0.0.1
```
Now, if you connect your device to ether2, you should get an IP address via DHCP in the 10.0.0.0/24 range, and you will have internet access. We recommend updating the Mikrotik device to the latest Stable version of Routeros 6 or 7. You can do this at System > Packages > Check for Updates. Also update the firmware at System > RouterBOARD > Upgrade. Both updates require a restart. 