# MikroTik IPv6 and Multicast Deltas versus Factory Reset

## Placeholder legend

Replace the placeholders below with your own names and values:

- `<wan-uplink-interface>`: physical or logical WAN interface (for example `ether1`, `pppoe-out1`)
- `<lan-bridge>`: LAN bridge (for example `bridge`)
- `<vlan1>`, `<vlan2>`, `<vlan3>`, `<vlan4>`: VLAN interfaces
- `<pd-pool>`: prefix delegation pool (for example `isp-prefix`)
- `<ula-dns-vlan1>`, `<ula-dns-vlan3>`: local IPv6 DNS addresses
- `<wan-iptv-interface>`: WAN side for IPTV (for example `vlan-wan-iptv`)
- `<wan-iptv-list>`: interface list for IPTV WAN (for example `WAN_IPTV`)
- `<iptv-provider-list>`: address list with authorized IPTV networks

Public examples in this document use RFC 3849 / RFC 4193 style ranges such as `2001:db8::/32` and `fd00::/8`.

Recommended generic role split:

- `vlan1`: internal / trusted
- `vlan2`: guest network
- `vlan3`: IoT or restricted segment
- `vlan4`: IPTV or multicast-specific segment

---

## Summary of the main IPv6 deltas

1. Multi-VLAN IPv6 design instead of a single LAN segment.
2. Prefix delegation distributed across multiple VLAN interfaces.
3. ULA addressing alongside global prefixes.
4. Zone-specific ND/RA and DNS advertisements.
5. Additional inter-zone IPv6 firewall segmentation on top of defconf.
6. Optional AP role reduction: APs as IPv6-light edge nodes.

## Summary of the multicast deltas

1. IGMP snooping explicitly enabled on relevant bridges.
2. Multicast querier / multicast-router behavior explicitly set on trunk / uplink links.
3. IGMP proxy chain for provider IPTV (upstream to downstream VLAN).
4. Firewall allowlist for the IPTV multicast path.
5. MLD / routed IPv6 multicast intentionally not or separately designed.

---

## Detailed deltas with explanation and template config

## Delta 1: Upstream IPv6 and prefix delegation

Difference from factory:
- Factory usually uses a simple WAN connection.
- More advanced setups bind DHCPv6-PD explicitly to the correct uplink, for example a PPPoE/VLAN chain.

Template CLI:

```rsc
/ipv6 dhcp-client
add add-default-route=yes interface=<wan-uplink-interface> pool-name=<pd-pool> \
    pool-prefix-length=64 request=prefix use-peer-dns=no
```

Why:
- Ensures predictable prefix delegation and routing behavior, independent of factory assumptions.

## Delta 2: IPv6 on multiple VLAN zones

Difference from factory:
- Factory is often single-LAN.
- Here IPv6 is split per security zone.

Template CLI:

```rsc
/ipv6 address
add from-pool=<pd-pool> interface=<vlan1>
add from-pool=<pd-pool> interface=<vlan2>
add from-pool=<pd-pool> interface=<vlan3>
add from-pool=<pd-pool> interface=<vlan4>
```

Why:
- Provides consistent dual-stack segmentation per VLAN zone.

## Delta 3: ULA structure alongside global prefixes

Difference from factory:
- Factory often has no explicit multi-zone ULA strategy.

Template CLI:

```rsc
/ipv6 pool
add name=local-prefix prefix=fd00:1::/64 prefix-length=128

/ipv6 address
add address=fd00::1 advertise=no interface=<vlan1>
add address=fd00::2:1 advertise=no interface=<vlan2>
add address=fd00::3:1 advertise=no interface=<vlan3>
add address=fd00::4:1 advertise=no interface=<vlan4>
```

Why:
- Stable internal IPv6 endpoints, even if the ISP prefix changes. Not strictly required, but useful if you want to reach the router and access points via fixed addresses.

## Delta 4: ND/RA and DNS advertisement per VLAN

Difference from factory:
- Factory usually advertises uniformly on one LAN.
- An advanced design makes DNS and RA zone-specific.

Template CLI:

```rsc
/ipv6 nd
set [ find default=yes ] interface=<vlan1> advertise-dns=yes \
    dns=<ula-dns-vlan1> other-configuration=yes
add interface=<vlan2> advertise-dns=yes \
    dns=2606:4700:4700::1111,2606:4700:4700::1001
add interface=<vlan3> advertise-dns=yes dns=<ula-dns-vlan3>
add interface=<vlan4> advertise-dns=yes advertise-mac-address=no
```

Why:
- Clients receive the correct resolver and RA behavior per zone.
- Example: `vlan2` can be a guest network using Cloudflare DNS, while other VLANs use an internal DNS server.

## Delta 5: DHCPv6 server on selected VLANs

Difference from factory:
- Factory is often minimal or only enabled on the default LAN.

Template CLI:

```rsc
/ipv6 dhcp-server option
add code=23 name=dns-v6 value="'<ula-dns-vlan1>'"

/ipv6 dhcp-server
add name=dhcpv6-vlan1 interface=<vlan1> dhcp-option=dns-v6 \
    address-pool=local-prefix prefix-pool=<pd-pool>
```

Why:
- Gives more control over DNS options and client behavior on the selected zone.
- Different device families obtain IPv6 in different ways, for example Linux, Windows, macOS, Android, iPhone, and so on. Some rely on ND/RA, others want DHCPv6.

## Delta 6: Additional inter-zone IPv6 firewall policy

Difference from factory:
- Factory includes baseline defconf rules, usually without specific east-west blocks between multiple LAN zones.

Template CLI (on top of defconf):

```rsc
/ipv6 firewall filter
add action=drop chain=forward comment="block vlan3->vlan1 (v6)" \
    connection-state=new in-interface=<vlan3> out-interface=<vlan1>
add action=drop chain=forward comment="block vlan2->vlan1 (v6)" \
    in-interface=<vlan2> out-interface=<vlan1>
add action=drop chain=forward comment="block vlan2->vlan3 (v6)" \
    in-interface=<vlan2> out-interface=<vlan3>
```

Why:
- Restricts lateral movement over IPv6 between zones.

## Delta 7: APs as IPv6-light nodes (optional design)

Difference from factory:
- In managed environments, APs often get a limited Layer 3 role and centralized policy on the core router.

Template CLI (on APs):

```rsc
/ipv6 settings
set accept-router-advertisements=yes

/ipv6 address
add advertise=no disabled=yes from-pool=<pd-pool> interface=<vlan1>

/ipv6 dhcp-client
add add-default-route=yes disabled=yes interface=<vlan1> pool-name=<pd-pool> \
    pool-prefix-length=64 request=address,prefix
```

Why:
- Keeps AP management predictable and centralizes policy on the router.

---

## Multicast deltas with template config

## Delta 8: IGMP snooping on relevant bridges

Difference from factory:
- Factory often lacks multicast optimization for complex Layer 2 topologies.

Template CLI:

```rsc
/interface bridge
set [ find name=<lan-bridge> ] igmp-snooping=yes multicast-querier=yes
```

Why:
- Reduces unnecessary multicast flooding and keeps membership state stable.

## Delta 9: multicast-router behavior on trunk / uplink ports

Difference from factory:
- Factory often leaves this at defaults; larger topologies benefit from explicit behavior.

Template CLI:

```rsc
/interface bridge port
set [ find interface=<trunk-port-1> ] multicast-router=permanent
set [ find interface=<trunk-port-2> ] multicast-router=permanent
set [ find interface=<uplink-port> ] multicast-router=permanent
```

Why:
- Prevents unstable querier / receiver detection in chained switches and APs.
- This matters in combination with the previous delta: IGMP snooping is needed for IPTV, but it can also block standard multicast traffic for IPv6, so you may need to explicitly re-enable multicast on your trunk ports, that is, links between for example the router and the access points.

## Delta 10: IGMP proxy for the IPTV chain

Difference from factory:
- Factory typically has no provider-specific multicast proxy flow.

Template CLI:

```rsc
/routing igmp-proxy
set quick-leave=yes

/routing igmp-proxy interface
add interface=<wan-iptv-interface> upstream=yes \
    alternative-subnets=<provider-iptv-subnet-1>,<provider-iptv-subnet-2>
add interface=<vlan4> alternative-subnets=<local-iptv-subnet>
```

Why:
- Forwards only the required multicast groups from the provider WAN to the local IPTV segment.

## Delta 11: Firewall allowlist for the IPTV multicast path

Difference from factory:
- Factory usually has no specific provider allowlist for IPTV multicast.

Template CLI:

```rsc
/ip firewall address-list
add list=<iptv-provider-list> address=<provider-iptv-subnet-1>
add list=<iptv-provider-list> address=<provider-iptv-subnet-2>

/ip firewall filter
add action=accept chain=input comment="Allow IPTV IGMP" dst-address=224.0.0.0/4 \
    in-interface-list=<wan-iptv-list> protocol=igmp
add action=drop chain=forward comment="Drop non-allowlist from IPTV WAN" \
    in-interface-list=<wan-iptv-list> src-address-list=!<iptv-provider-list>
add action=drop chain=forward comment="Drop non-allowlist to IPTV WAN" \
    out-interface-list=<wan-iptv-list> dst-address-list=!<iptv-provider-list>
```

Why:
- Limits unwanted multicast or traffic injection outside the expected provider ranges.

## Delta 12: MLD-related IPv6 multicast behavior

Difference from factory:
- Factory includes baseline defconf for IPv6 multicast filtering, but usually no explicit routed MLD proxy architecture.

Template CLI (explicitly preserving defconf-like behavior):

```rsc
/ipv6 firewall raw
add action=accept chain=prerouting comment="accept local multicast scope" \
    dst-address=ff02::/16
add action=drop chain=prerouting comment="drop other multicast destinations" \
    dst-address=ff00::/8
```

Why:
- Keeps link-local multicast functional and limits unwanted broader multicast.

Note:
- If you want routed IPv6 multicast between zones, design a separate MLD proxy or multicast routing architecture; that is a separate use case.

