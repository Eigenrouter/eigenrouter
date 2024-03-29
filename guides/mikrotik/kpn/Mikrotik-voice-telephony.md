---
title: Mikrotik Voice / Telephony
description: How to Setup KPN Voice / Telephony in Mikrotik
published: true
date: 2023-04-23T15:52:54.054Z
tags: 
editor: markdown
dateCreated: 2023-04-16T17:44:28.220Z
---

# Mikrotik Voice / Telephony

There is little configuration required to get third party VoIP products to work with KPN SIP. Nowadays you can easily find the required information in Mijn KPN. You will need the Mijn KPN account, because the SIP password for your account is only visible there. We will assume that you know how to set up VoIP equipment. If not, feel free to ask about this in our Discord server.

The only requirement as of when this how-to was written, is to add VLAN 7 to your WAN interface. We will once again assume that ether1 is your WAN interface. 

```
/interface vlan
add interface=ether1 l2mtu=1594 name=vlan1.7 vlan-id=7
```

Your LAN can now theoretically connect to the KPN VoIP servers. Configuration on your client device(s) will be required, but this tutorial will not cover that. 

A small footnote: 
You may have to disable the 'sip' service in /ip firewall service-ports. Generally it depends on the equipment used. If you experience problems like one-way audio or not being able to register your SIP account, disable this service. 

```
/ip firewall service-port set disabled=yes sip
```