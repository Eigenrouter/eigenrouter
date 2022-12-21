### Mikrotik-tv-settings

Add TV to your router configuration.
you can only follow this how-to if your internet is setup about this [Mikrotik-internet-only](Mikrotik-Internet-only.md)

before you start check if multicast package is installed.
if you running routeros 7 your fine!


Make Vlan4 for itv from KPN, and add this on Eth1

```
/interface vlan
add interface=ether1 name=vlan1.4 vlan-id=4
```

Turn on *igmp-snooping* on your bridge, if your bridge exist than only set igmp-snooping to yes

```
/interface bridge
add arp=proxy-arp igmp-snooping=yes name=local protocol-mode=none
```

set dhcp-client options.

```
/ip dhcp-client option
add code=60 name=option60-vendorclass value="'IPTV_RG'"
/ip dhcp-client
add default-route-distance=210 dhcp-options=option60-vendorclass disabled=no \
    interface=vlan1.4 use-peer-dns=no use-peer-ntp=no
```

setup IGMP Proxy

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
add action=masquerade chain=srcnat comment="IPTV" dst-address=213.75.112.0/21 out-interface=vlan1.4
add action=masquerade chain=srcnat comment="IPTV" dst-address=217.166.0.0/16 out-interface=vlan1.4
```

Add DHCP Options

```
/ip dhcp-server option
add code=60 name=option60-vendorclass value="'IPTV_RG'"
add code=28 name=option28-broadcast value="'10.0.0.255'"

/ip dhcp-server option sets
add name=IPTV options=option60-vendorclass,option28-broadcast
```

