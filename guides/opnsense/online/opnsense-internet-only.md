---
title: OPNsense internet only
description: OPNsense internet only with Online
published: true
date: 2023-04-22T14:37:56.205Z
tags: 
editor: markdown
dateCreated: 2023-04-22T14:37:56.205Z
---

### How to Setup Online.nl on OPNsense [Internet only]

In this How-To we are going to setup Online.nl on OPNsense

### Step. 1

Go to

```
Interfaces > Other Types > VLAN
```

![VLANs](/images/online/opnsense-internet-only/iface_other_vlan.png)

Create VLAN 1001 on your WAN interface and press Save.

### Step. 2
Go to

```
Interfaces > Assignments
```

![Assignments](/images/online/opnsense-internet-only/iface_assignments.png)

Change the WAN interface to ```Parent: [wan-interface], Tag: 1001```

And press Save.

### Step. 3
Go to

```
Interfaces > WAN
```

![InterfaceWAN](/images/online/opnsense-internet-only/iface_wan.png)

**Generic configuration**

Set IPv4 Configuration Type: ```DHCP```

Set IPv6 Configuration Type: ```DHCPv6```
Or None. So far i have not seen IPv6 on this connection.

Finish these settings my pressing save.

### Step. 4
If you go back to:
```
Lobby > Dashboard
```

![Gateway](/images/online/opnsense-internet-only/gateway.png)

You should see that the gateway should be up.
Next thing to do.... Go fast!