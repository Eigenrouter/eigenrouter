### Mikrotik - adding ipv6 to internet only

This guide is based on a Mikrotik device with at least two Ethernet ports. We start with a completely empty configuration. To wipe your configuration, connect to your Mikrotik device via Winbox or Webfig (HTTP) and navigate to System > Reset Configuration. Tick “No Default Configuration” and reset the device via the “Reset Configuration” button. The device will now restart.

After wiping the configuration, you can connect to the Mikrotik via MAC-Winbox on Windows or via Wine. We recommend configuring a management network on an Ethernet interface that you will not use for the network that we will create in this guide. The simplest way to do this is to set a unique IP address on for example ether3 (as ether3 will not be used in this guide) and to connect to the device via IP on ether3. Refer to the IP Address section in the steps below to see how to do this.

You can download Winbox here: [ MikroTik Routers and Wireless - Software](https://mikrotik.com/download)

This guide requires that you install the ‘multicast’ package for Routeros version 6. If you’re running Routeros 7, skip this step. 

The default login credentials are “admin” with no password. If you are running the latest version of Routeros 6 or 7, you will be prompted to change the password on first login.

This guide will give you commands to enter in the Mikrotik Terminal. You can do this either via SSH or in the Terminal that is included in the Winbox application.

This guide will only work if you have installed this guide [Mikrotik-internet-only](guides/mikrotik/KPN/Mikrotik-Internet-only.md) 


Make a ppp profile for ipv6

```
/ppp profile
set *0 only-one=yes use-compression=yes use-ipv6=no use-upnp=no
add name=default-ipv6 only-one=yes remote-ipv6-prefix-pool=*0 use-compression=yes use-upnp=no
```
Now we make the pppoe connection with KPN (REMOVE YOUR OLD PPPOE CONNECTION)

```
/interface pppoe-client
add add-default-route=yes allow=pap disabled=no interface=vlan1.6 keepalive-timeout=20 max-mru=1500 max-mtu=1500 name="pppoe-client" profile=default-ipv6 user=kpn@kpn
```

Add ipv6 dhcp client

```
/ipv6 dhcp-client
add add-default-route=yes interface=pppoe-client pool-name=0 pool-prefix-length=48 request=prefix use-peer-dns=no
```

Add basic firewall

```
/ipv6 firewall filter
add action=accept chain=input comment="defconf: accept established,related,untracked" connection-state=established,related,untracked
add action=accept chain=input comment="defconf: accept DHCPv6-Client prefix delegation." dst-port=546 protocol=udp src-address=fe80::/10
```

ND Settings

```
/ipv6 nd
add advertise-mac-address=no disabled=yes hop-limit=64 interface=bridge
```

Add Lan Address

``` 
/ipv6 address
add from-pool=*0 interface=local
```



Now, if you connect your device to ether2, you should get an IP address via DHCP in the 10.0.0.0/24 range, and you will have internet access. We recommend updating the Mikrotik device to the latest Stable version of Routeros 6 or 7. You can do this at System > Packages > Check for Updates. Also update the firmware at System > RouterBOARD > Upgrade. Both updates require a restart. 
