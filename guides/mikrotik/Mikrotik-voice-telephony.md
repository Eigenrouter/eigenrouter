### Mikrotik-voice-telephony


This is a short manual on how we can make telephony work with Mikrotik.

We make a new Vlan7 on the Mikrotik and we connect this on the WAN interface.

```
/interface vlan
add interface=ether1 l2mtu=1594 name=vlan1.7 vlan-id=7
```

Make now a Vlan on a LAN interface, use a free port in this example ETH5.

```
add interface=ether5 l2mtu=1594 name=vlan5.7 vlan-id=7
```

Make now a bridge for the telephony Vlan to connect.

```
/interface bridge
add name=telephony-bridge
```

Add the Vlan7 from port 1 and 5 in the bridge.

```
/interface bridge port
add bridge=telephony-bridge interface=vlan1.7
add bridge=telephony-bridge interface=vlan5.7
```

Now you can connect your Experiabox from kpn.
Connect port 5 from the Mikrotik to the WAN port of the Experiabox.
Now you can make phone calls with your "normal" phone.

You go more advanced? than connect port 5 to your freepbx / 3cx server or somthing like that.
make sure the server has internet access.