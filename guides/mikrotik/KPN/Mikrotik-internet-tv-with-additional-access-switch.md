### Mikrotik internet + tv wth an additional access swith between router and TV

This is the configuration I use with a mikrotik model RouterBOARD
962UiGS-5HacT2HnT and the KPN set top box connected to a netgear GS108Ev3
managed switch. I will not explain all the steps, you should already know what
vlans are, what trunk port and an access port are. I cannot give a step-by-step
explanation, because I cannot verify every configuration with every combination
of network switches, and also there are at least three ways of configuring vlans
in the mikrotik, so this is what works for me.

This is the mikrotik configuration. The uplink to the netgear switch is ether5.

```
/interface bridge add arp=proxy-arp igmp-snooping=yes name=br-vlan4
/interface bridge add name=br-vlan6
/interface bridge add name=bridge-local
/interface ethernet set [ find default-name=ether5 ] comment="uplink to netgear
under desk living-room"
/interface wireless set [ find default-name=wlan1 ] disabled=no ssid=MikroTik
/interface vlan add interface=ether1 name=ether1.4 vlan-id=4
/interface vlan add interface=ether1 mtu=1508 name=ether1.6 vlan-id=6
/interface vlan add interface=ether5 name=ether5.4 vlan-id=4
/interface vlan add interface=ether5 name=ether5.6 vlan-id=6
/ip dhcp-client option add code=60 name=option60-vendorclass value="'IPTV_RG'"
/ip dhcp-server option add code=60 name=option60-vendorclass value="'IPTV_RG'"
/ip dhcp-server option add code=28 name=option28-broadcast value="'10.0.4.255'"
/ip dhcp-server option sets add name=IPTV
options=option60-vendorclass,option28-broadcast
/ip pool add name=dhcp_pool0 ranges=192.168.88.2-192.168.88.254
/ip pool add name=dhcp_pool2 ranges=10.0.4.2-10.0.4.254
/ip pool add name=dhcp_pool3 ranges=10.0.6.2-10.0.6.254
/ip dhcp-server add address-pool=dhcp_pool0 disabled=no interface=bridge-local
name=dhcp1
/ip dhcp-server add address-pool=dhcp_pool2 dhcp-option-set=IPTV disabled=no
interface=br-vlan4 name=dhcp3
/ip dhcp-server add address-pool=dhcp_pool3 disabled=no interface=br-vlan6
name=dhcp4
/ppp profile set *0 only-one=yes use-compression=yes use-ipv6=no use-upnp=no
/ppp profile add name=default-ipv6 only-one=yes use-compression=yes use-upnp=no
/interface pppoe-client add add-default-route=yes disabled=no interface=ether1.6
max-mru=1500 max-mtu=1500 name=pppoe-client password=kpn profile=default-ipv6
use-peer-dns=yes user=user@internet
/interface bridge port add bridge=bridge-local interface=ether3
/interface bridge port add bridge=br-vlan4 interface=ether5.4
/interface bridge port add bridge=br-vlan6 interface=ether5.6
/interface bridge port add bridge=br-vlan5 interface=ether5.5
/ip address add address=192.168.88.1/24 interface=bridge-local
network=192.168.88.0
/ip address add address=10.0.4.1/24 interface=br-vlan4 network=10.0.4.0
/ip address add address=10.0.6.1/24 interface=br-vlan6 network=10.0.6.0
/ip dhcp-client add default-route-distance=210 dhcp-options=option60-vendorclass
disabled=no interface=ether1.4 use-peer-dns=no use-peer-ntp=no
/ip dhcp-server network add address=10.0.4.0/24 dhcp-option=domain-search-option
gateway=10.0.4.1
/ip dhcp-server network add address=10.0.6.0/24 gateway=10.0.6.1
/ip dhcp-server network add address=192.168.88.0/24 dns-server=8.8.8.8
gateway=192.168.88.1
/ip dns set allow-remote-requests=yes cache-max-ttl=1d servers=8.8.8.8,8.8.4.4
/ip firewall address-list add address=0.0.0.0/8 comment=rfc8690
list=not_inet_routable
/ip firewall address-list add address=192.168.88.2-192.168.88.254
list=allowed_to_router
/ip firewall address-list add address=224.0.252.127 list=iptv_kpn
/ip firewall address-list add address=224.0.252.126 list=iptv_kpn
/ip firewall address-list add address=224.0.252.178 list=iptv_kpn
/ip firewall address-list add address=224.0.252.128 list=iptv_kpn
/ip firewall filter add action=accept chain=input
connection-state=established,related
/ip firewall filter add action=accept chain=input
src-address-list=allowed_to_router
/ip firewall filter add action=accept chain=input comment="iptv multicast vlan
1.4" in-interface=ether1.4 protocol=udp src-address=217.166.226.138
src-port=49152
/ip firewall filter add action=drop chain=input in-interface=pppoe-client
log=yes
/ip firewall filter add action=fasttrack-connection chain=forward
connection-state=established,related
/ip firewall filter add action=accept chain=forward comment="Established,
Related" connection-state=established,related
/ip firewall filter add action=drop chain=forward comment="drop incoming not
nat-ed pakkets" connection-nat-state=!dstnat connection-state=new
in-interface=pppoe-client log=yes log-prefix=!NAT
/ip firewall filter add action=drop chain=forward comment="Drop invalid"
connection-nat-state="" connection-state=invalid
/ip firewall filter add action=accept chain=forward comment="iptv traffic"
dst-address-list=iptv_kpn in-interface=ether1.4 out-interface=br-vlan4
protocol=udp
/ip firewall filter add action=drop chain=forward disabled=yes log=yes
/ip firewall nat add action=masquerade chain=srcnat comment="Needed for IPTV"
dst-address=213.75.112.0/21 out-interface=ether1.4
/ip firewall nat add action=masquerade chain=srcnat comment="Needed for IPTV"
dst-address=217.166.0.0/16 out-interface=ether1.4
/ip firewall nat add action=masquerade chain=srcnat out-interface=pppoe-client
/ipv6 dhcp-client add add-default-route=yes disabled=yes interface=pppoe-client
pool-name=pool pool-prefix-length=48 rapid-commit=no request=prefix
use-peer-dns=no
/routing igmp-proxy set quick-leave=yes
/routing igmp-proxy interface add alternative-subnets=0.0.0.0/0
interface=ether1.4 upstream=yes
/routing igmp-proxy interface add interface=br-vlan4
/system clock set time-zone-name=Europe/Amsterdam
```

### Netgear switch configuration

#### VLAN configuration: 
The netgear has the uplink on port 1. On port 2 of the netgear, I have connected
the KPN set top box (vlan id 4). The rest are devices on the internet vlan 6.

On the netgear web interface (if you do not know how to get there, check the
manual), you need to click on 'VLAN', then on 8021.Q, then 'Advanced'. 

On the 'Vlan configuration' menu ensure both VLAN 4 and 6 exist on the swith.
Otherwise enter the vlan id number and click on 'create' for each of them.

Then on the 'Vlan membership' menu, select the vlan id 4, and and set port 1
(uplink) as 'T' (tagged) and port 2 as 'U' (untagged). The rest of the ports,
empty. Obviously, replace port 1 and 2 for the correct ports you have used on
your switch.

Repeat for vlan id 6: port 1 'T', ports 3, 4, ...., 'U'.

Last, on the 'Port PVID' menu, set on port 2 pvid '4' and the internet ports
'6'. 

As the last step, I removed the vlan id 1 membership of all the ports. 

#### Multicast configuration:

The final setting consists of turning on IGMP snooping for vlan id 4 on the
netgear. Click on 'System', then 'Multicast'. Select 'Enable' for 'IGMP snooping
status'. Then enter '4' for 'VLAN ID enabled for IGMP snooping'. The rest of the
settings can be left alone.

That's it, connect your KPN set top box to port 2 of the netgear switch and if
all has gone well it should work.

### Troubleshooting
Check the logs of the mikrotik.
