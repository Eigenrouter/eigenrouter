# MikroTik IPv6 en Multicast delta's versus factory reset

## Placeholder-legenda

Vervang onderstaande placeholders door je eigen namen/waarden:

- `<wan-uplink-interface>`: fysieke of logische WAN-interface (bijv. `ether1`, `pppoe-out1`)
- `<lan-bridge>`: LAN bridge (bijv. `bridge`)
- `<vlan1>`, `<vlan2>`, `<vlan3>`, `<vlan4>`: VLAN interfaces
- `<pd-pool>`: prefix-delegation pool (bijv. `isp-prefix`)
- `<ula-dns-vlan1>`, `<ula-dns-vlan3>`: lokale IPv6 DNS adressen
- `<wan-iptv-interface>`: WAN zijde voor IPTV (bijv. `vlan-wan-iptv`)
- `<wan-iptv-list>`: interface-list voor IPTV WAN (bijv. `WAN_IPTV`)
- `<iptv-provider-list>`: address-list met geautoriseerde IPTV netwerken

Voor publieke voorbeelden in dit document worden RFC 3849/rfc4193-stijl ranges gebruikt (zoals `2001:db8::/32` en `fd00::/8`).

Aanbevolen generieke rolverdeling:

- `vlan1`: intern/trusted
- `vlan2`: gastnetwerk
- `vlan3`: IoT of beperkt segment
- `vlan4`: IPTV of multicast-specifiek segment

---

## Samenvatting van de belangrijkste IPv6-delta's

1. Multi-VLAN IPv6-ontwerp in plaats van enkelvoudig LAN-segment.
2. Prefixdelegatie verdeeld over meerdere VLAN-interfaces.
3. ULA-adressering naast globale prefixen.
4. Zone-specifieke ND/RA en DNS-advertenties.
5. Extra inter-zone IPv6 firewallsegmentatie bovenop defconf.
6. Optionele AP-rolreductie: AP's als IPv6-light edge nodes.

## Samenvatting van de multicast-delta's

1. IGMP snooping expliciet aan op relevante bridges.
2. Multicast querier/multicast-router gedrag expliciet ingesteld op trunk/uplink.
3. IGMP proxy-keten voor provider-IPTV (upstream -> downstream VLAN).
4. Firewall allowlist voor IPTV multicast-pad.
5. MLD/routed IPv6 multicast bewust niet of juist wel apart ingericht.

---

## Gedetailleerde delta's met uitleg en template-config

## Delta 1: Upstream IPv6 en prefixdelegatie

Verschil met factory:
- Factory gebruikt meestal een eenvoudige WAN-koppeling.
- Geavanceerde setups koppelen DHCPv6-PD expliciet aan de juiste uplink (bijv. PPPoE/VLAN chain).

Template CLI:

```rsc
/ipv6 dhcp-client
add add-default-route=yes interface=<wan-uplink-interface> pool-name=<pd-pool> \
    pool-prefix-length=64 request=prefix use-peer-dns=no
```

Waarom:
- Zorgt voor voorspelbare prefixdelegatie en routegedrag, los van factory-assumpties.

## Delta 2: IPv6 op meerdere VLAN-zones

Verschil met factory:
- Factory is vaak single-LAN.
- Hier wordt IPv6 per security-zone verdeeld.

Template CLI:

```rsc
/ipv6 address
add from-pool=<pd-pool> interface=<vlan1>
add from-pool=<pd-pool> interface=<vlan2>
add from-pool=<pd-pool> interface=<vlan3>
add from-pool=<pd-pool> interface=<vlan4>
```

Waarom:
- Consistente dual-stack segmentatie per VLAN-zone.

## Delta 3: ULA-structuur naast globale prefixen

Verschil met factory:
- Factory heeft vaak geen expliciete multi-zone ULA-strategie.

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

Waarom:
- Stabiele interne IPv6 endpoints, ook als ISP-prefix wijzigt. Niet perse nodig maar kan handig zijn als je bijv. je router en accesspoints op een vast adres wilt kunnen bereiken.

## Delta 4: ND/RA en DNS-advertentie per VLAN

Verschil met factory:
- Factory adverteert meestal uniform op 1 LAN.
- Geavanceerd ontwerp maakt DNS/RA zone-specifiek.

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

Waarom:
- Clients krijgen per zone het juiste resolver- en RA-gedrag.
- Voorbeeld voor bijv. vlan2 als gast-netwerk met dns servers van cloudflare, andere vlan's met interne dns server.

## Delta 5: DHCPv6 server op geselecteerde VLAN's

Verschil met factory:
- Factory is vaak minimaal of enkel op default LAN.

Template CLI:

```rsc
/ipv6 dhcp-server option
add code=23 name=dns-v6 value="'<ula-dns-vlan1>'"

/ipv6 dhcp-server
add name=dhcpv6-vlan1 interface=<vlan1> dhcp-option=dns-v6 \
    address-pool=local-prefix prefix-pool=<pd-pool>
```

Waarom:
- Meer controle over DNS-opties en clientgedrag op de gekozen zone.
- Devices gebruiken verschillende manieren om ipv6 adressen te verkrijgen, bijv. linux/windows/macos/android/iphone etc. sommige gebruiken ND/RA en anderen willen DHCPv6.


## Delta 6: Extra IPv6 inter-zone firewallbeleid

Verschil met factory:
- Factory bevat basisdefconf, meestal zonder specifieke oost-west blokkades tussen meerdere LAN-zones.

Template CLI (bovenop defconf):

```rsc
/ipv6 firewall filter
add action=drop chain=forward comment="block vlan3->vlan1 (v6)" \
    connection-state=new in-interface=<vlan3> out-interface=<vlan1>
add action=drop chain=forward comment="block vlan2->vlan1 (v6)" \
    in-interface=<vlan2> out-interface=<vlan1>
add action=drop chain=forward comment="block vlan2->vlan3 (v6)" \
    in-interface=<vlan2> out-interface=<vlan3>
```

Waarom:
- Beperkt laterale beweging over IPv6 tussen zones.

## Delta 7: AP's als IPv6-light nodes (optioneel ontwerp)

Verschil met factory:
- In managed omgevingen krijgen AP's vaak een beperkte L3-rol en centrale policy op core-router.

Template CLI (op AP's):

```rsc
/ipv6 settings
set accept-router-advertisements=yes

/ipv6 address
add advertise=no disabled=yes from-pool=<pd-pool> interface=<vlan1>

/ipv6 dhcp-client
add add-default-route=yes disabled=yes interface=<vlan1> pool-name=<pd-pool> \
    pool-prefix-length=64 request=address,prefix
```

Waarom:
- Houdt AP-beheer voorspelbaar en centraliseert policy op de router.

---

## Multicast-delta's met template-config

## Delta 8: IGMP snooping op relevante bridges

Verschil met factory:
- Factory heeft vaak geen multicast-optimalisatie voor complexe L2-topologie.

Template CLI:

```rsc
/interface bridge
set [ find name=<lan-bridge> ] igmp-snooping=yes multicast-querier=yes
```

Waarom:
- Vermindert onnodige multicast-flooding en houdt membership-state stabiel.

## Delta 9: multicast-router gedrag op trunk/uplink poorten

Verschil met factory:
- Factory laat dit vaak op defaults; grotere topologieen hebben baat bij expliciet gedrag.

Template CLI:

```rsc
/interface bridge port
set [ find interface=<trunk-port-1> ] multicast-router=permanent
set [ find interface=<trunk-port-2> ] multicast-router=permanent
set [ find interface=<uplink-port> ] multicast-router=permanent
```

Waarom:
- Voorkomt instabiele querier/receiver detectie in chained switches/AP's.
- Dit is belangrijk in combinatie met de voorgaande delta, IGMP snooping heb je nodig voor KPN IPTV, maar dit blokkeerd 'standaard' ook de multicast berichten voor IPv6, daarom moet je expliciet multicast weer enablen op je trunk poorten (connecties tussen bijv. je router en access-points).

## Delta 10: IGMP proxy voor IPTV-keten

Verschil met factory:
- Factory heeft doorgaans geen provider-specifieke multicast proxyflow.

Template CLI:

```rsc
/routing igmp-proxy
set quick-leave=yes

/routing igmp-proxy interface
add interface=<wan-iptv-interface> upstream=yes \
    alternative-subnets=<provider-iptv-subnet-1>,<provider-iptv-subnet-2>
add interface=<vlan4> alternative-subnets=<local-iptv-subnet>
```

Waarom:
- Leidt alleen benodigde multicastgroepen van provider-WAN naar lokaal IPTV-segment.

## Delta 11: Firewall allowlist voor IPTV multicast-pad

Verschil met factory:
- Factory bevat meestal geen specifieke provider-allowlist voor IPTV multicast.

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

Waarom:
- Beperkt ongewenste multicast/traffic injection buiten verwachte provider-ranges.

## Delta 12: MLD-gerelateerd IPv6 multicastgedrag

Verschil met factory:
- Factory bevat basisdefconf voor IPv6 multicastfiltering, maar meestal geen expliciete routed MLD-proxy-architectuur.

Template CLI (defconf-achtig gedrag expliciet borgen):

```rsc
/ipv6 firewall raw
add action=accept chain=prerouting comment="accept local multicast scope" \
    dst-address=ff02::/16
add action=drop chain=prerouting comment="drop other multicast destinations" \
    dst-address=ff00::/8
```

Waarom:
- Houdt link-local multicast functioneel en beperkt ongewenste bredere multicast.

Opmerking:
- Wil je routed IPv6 multicast tussen zones, ontwerp dan apart een MLD-proxy of multicast routing-architectuur; dat is een aparte use-case.
