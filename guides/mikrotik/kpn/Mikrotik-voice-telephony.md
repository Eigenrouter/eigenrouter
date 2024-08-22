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

Previously it was necessary to add VLAN 7 to the WAN interface in order to register SIP accounts. This is no longer the case. VLAN 6 now carries the required connections. You will already have VLAN 6 in use after setting up internet access. 

You may have to disable the 'sip' service in /ip firewall service-ports. Generally it depends on the equipment used. If you experience problems like one-way audio or not being able to register your SIP account, disable this service. 

```
/ip firewall service-port set disabled=yes sip
```
