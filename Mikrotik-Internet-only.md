#### make sure you start with an empty mikrotik, also no default config!

you can access your router on eth2 via winbox
you can download winbox [MikroTik Routers and Wireless - Software](https://mikrotik.com/download)

Default login is admin with no password.
Open the terminal in winbox and do copy & paste the config.

Make your WAN connection
```
/interface ethernet
# Poort 1 (ether1) = Glasvezel KPN
set [ find default-name=ether1 ] arp=proxy-arp l2mtu=1598 loop-protect=off

```

Configureer your eth2

```
# Poort 2 (ether2) = LAN  
set 1 arp=enabled auto-negotiation=yes \
    disabled=no full-duplex=yes l2mtu=1598 \
    mtu=1500 name=ether2 speed=1Gbps
```

Now we make the internet connection we connect vlan6 on the physical interface eth1
```
/interface vlan
add interface=ether1 loop-protect=off name=vlan1.6 vlan-id=6
```

Now we make the pppoe connection with KPN

Basic settings for Ipv6
```
/ppp profile
set *0 only-one=yes use-compression=yes use-upnp=no use-ipv6=no
add name=default-ipv6 only-one=yes use-compression=yes use-upnp=no use-ipv6=yes
```

Now we connect the pppoe-client to the vlan6 that we make earlier.

```
/interface pppoe-client

add add-default-route=yes allow=pap disabled=no interface=vlan1.6 \
    keepalive-timeout=20 max-mru=1500 max-mtu=1500 name=pppoe-client \
    password=kpn profile=default user=kpn
```

Besure BGP is disabled

```
/routing bgp instance
set default disabled=yes
```

Make now a bridge on the mikrotik with all your lan ports.
this is a mikrotik with 5 ports, if you have more ports add them to the bridge local

```
/interface bridge
add name=local arp=proxy-arp

/interface bridge port
add bridge=local interface=ether2
add bridge=local interface=ether3
add bridge=local interface=ether4
add bridge=local interface=ether5
```

Now we make a local ip-address, change it to your own range or use this one.

```
/ip address
add address=10.0.0.1/24 disabled=no interface=bridge-local network=10.0.0.0
```

Setup dns servers in this example i use google dns

```
/ip dns
set allow-remote-requests=yes cache-max-ttl=1d cache-size=2048KiB \
    max-udp-packet-size=4096 servers=8.8.8.8,8.8.4.4
```


Add firewall
Basic rules to get kpn work.
```
/ip firewall filter
add action=accept chain=input disabled=no in-interface=pppoe-client protocol=icmp
add action=accept chain=input connection-state=related disabled=no
add action=accept chain=input connection-state=established disabled=no
add action=reject chain=input in-interface=pppoe-client protocol=tcp reject-with=icmp-port-unreachable
add action=reject chain=input in-interface=pppoe-client protocol=udp reject-with=icmp-port-unreachable
```

Firewall NAT rules

```
/ip firewall nat
add action=masquerade chain=srcnat comment="Needed for internet" \
    out-interface=pppoe-client src-address=10.0.0.0/24
```

Setup DHCP server for you lan

```
/ip pool
add name=local-network ranges=10.0.0.10-10.0.0.254  
/ip dhcp-server
add address-pool=thuisnetwerk authoritative=yes disabled=no interface=local \
    lease-time=1h30m name=dhcp-home
/ip dhcp-server config
set store-leases-disk=15m
/ip dhcp-server network
add address=10.0.0.0/24 dns-server=10.0.0.1 domain=home.local gateway=\
    10.0.0.1
```

Now is your internet working.
