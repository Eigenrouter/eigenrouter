Code snippet to make more than one internet connection individually usable (active-active):

Create new routing tables:
```
/routing table
add disabled=no fib name=ROS>WAN1
add disabled=no fib name=ROS>WAN2
```

Add mangle rules that use the new routing tables:
```
/ip firewall mangle
add action=mark-connection chain=input in-interface=sfp-sfpplus1 new-connection-mark=WAN1>ROS passthrough=yes
add action=mark-connection chain=input in-interface=sfp-sfpplus2 new-connection-mark=WAN2>ROS passthrough=yes
add action=mark-routing chain=output connection-mark=WAN1>ROS new-routing-mark=ROS>WAN1 passthrough=yes
add action=mark-routing chain=output connection-mark=WAN2>ROS new-routing-mark=ROS>WAN2 passthrough=yes
```

Add routes for the new routing tables:
```
/ip route
add disabled=no distance=1 dst-address=0.0.0.0/0 gateway=sfp-sfpplus1 routing-table=ROS>WAN1
add disabled=no distance=1 dst-address=0.0.0.0/0 gateway=sfp-sfpplus2 routing-table=ROS>WAN2
```

Add more tables/rules/routes based on the counting mechanism above if needed.
