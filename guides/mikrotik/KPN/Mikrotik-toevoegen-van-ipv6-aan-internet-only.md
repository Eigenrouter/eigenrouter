### Mikrotik - adding ipv6 to internet only

You can download Winbox here: [ MikroTik Routers and Wireless - Software](https://mikrotik.com/download)

This guide will give you commands to enter in the Mikrotik Terminal. You can do this either via SSH or in the Terminal that is included in the Winbox application.

This guide will only work if you have installed this guide [Mikrotik-internet-only](guides/mikrotik/KPN/Mikrotik-Internet-only.md) 

On RouterOS version 6, make sure to enable/install the IPv6 package. On RouterOS version 7 this package is included. 

Create a PPP profile for ipv6

```
/ppp profile
add name=default-ipv6 only-one=yes remote-ipv6-prefix-pool=*0 use-compression=yes use-upnp=no
```
Now we make the pppoe connection with KPN (REMOVE YOUR OLD PPPOE CONNECTION)
or on your existing pppoe client change profile to default-ipv6

```
/interface pppoe-client
add add-default-route=yes allow=pap disabled=no interface=vlan1.6 keepalive-timeout=20 max-mru=1500 max-mtu=1500 name="pppoe-client" profile=default-ipv6 user=kpn@kpn
```

Add the DHCPv6 client

```
/ipv6 dhcp-client
add add-default-route=yes interface=pppoe-client pool-name=*0 pool-prefix-length=48 request=prefix use-peer-dns=no
```

Add basic firewall

```
/ipv6 firewall filter
add action=accept chain=input comment="defconf: accept established,related,untracked" connection-state=established,related,untracked
add action=accept chain=input comment="defconf: accept DHCPv6-Client prefix delegation." dst-port=546 protocol=udp src-address=fe80::/10
add action=drop chain=input comment="drop everything else"
```

Configure ND Settings

```
/ipv6 nd
add advertise-mac-address=no disabled=yes hop-limit=64 interface=local
```

Add LAN Address

``` 
/ipv6 address
add from-pool=*0 interface=local
```

Now you have added ipv6 to your Mikrotik device. We recommend updating the Mikrotik device to the latest Stable version of Routeros 6 or 7. You can do this at System > Packages > Check for Updates. Also update the firmware at System > RouterBOARD > Upgrade. Both updates require a restart. 
