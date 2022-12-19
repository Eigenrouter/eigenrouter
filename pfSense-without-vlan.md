
#### How to Setup KPN on pfSense without seperate TV VLAN

After install pfSense go to

```
Interface > Assignments > Vlans
```
![vlans](images/pfsense-without-vlan/Vlans.png)

Create 2 Vlans *vlan6* & *Vlan4* on your WAN interface.

Than go to

```
Interface > Assignments > PPP
```

Add a new interface and link it to you WAN port

![PPP](images/pfsense-without-vlan/PPP.png)

Than go to 

```
Interface > Assignments 
```
![assignments](images/pfsense-without-vlan/assignments.png)

Set here your WAN to you PPP interface.  
Set Lan to your lan interface / port.  
And assign a new interface Vlan4 for kpn tv

Than go to

```
Interface > TV_KPN
```
and set this by _Send options_
```
dhcp-class-identifier "IPTV_RG"
```
and set this by _Request options_
```
subnet-mask,routers,classless-routes
```

Save the settings.

Than to go

```
Services > IGMP Proxy
```

![IGMPProxy](images/pfsense-without-vlan/IGMPPROXY.png)

Enable IGMP

add an upstream and downstream.
Upstream takes the default route. 0.0.0.0/0 or on pfSense 0.0.0.0/1 , 128.0.0.0/1
And by downstream add your lan ip and subnet

Than go to

```
Services > DHCP Server > LAN
```

Scroll to the bottom and open *Additional BOOTP/DHCP Options*
and enter the following lines

![BOOTP](images/pfsense-without-vlan/Additional-BOOTP-DHCP.png)

In this example is the broadcast adress 10.0.0.255 set here your own broadcast address

Than go to

```
Firewall > NAT > Outbound
```

![BOOTP](images/pfsense-without-vlan/outbound.png)

And add the line above under mappings

Than go to

```
Firewall > Rules > WAN
```

Add the following lines

![RulesWAN](images/pfsense-without-vlan/rules-wan.png)

On the IPv4 IGMP rule enable the following line.
_Allow IP options_
![ipoptions](images/pfsense-without-vlan/ipoptions.png)

Than go to

```
Firewall Rules LAN
```

and add the following lines

![fwlan](images/pfsense-without-vlan/firewalllan.png)

On the IPv4 IGMP rule _Allow IP options_
![ipoptions](images/pfsense-without-vlan/ipoptions.png)

_The 213.75.112.0/21 is the ip/subnet from kpn itv_

Than go to

```
Firewall > Rules > TV_KPN
```

And add the following lines
![tvKPN](images/pfsense-without-vlan/TV_KPN.png)

On the IPv4 IGMP rule _Allow IP options_
![ipoptions](images/pfsense-without-vlan/ipoptions.png)
