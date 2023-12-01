---
title: Routed-iptv
description: 
published: true
date: 2023-12-01T21:07:18.109Z
tags: 
editor: markdown
dateCreated: 2023-12-01T21:07:18.109Z
---

# Routed-iptv KPN

**De volgende modellen zijn geschikt voor routed iptv**

![routediptvondersteuning.png](/images/kpn/draytek/routediptvondersteuning.png)

Zorg er voor dat je Draytek de nieuwste firmware heeft.  
Deze kun je [hier downloaden](https://www.draytek.nl/firmware/)
### **Routed iptv config**
**Belangrijke instellingen:**
- VLAN Tag:4 voor IPTV (Virtual WAN).
- WAN DHCP Option 60:IPTV_RG voor het verkrijgen van een IP-adres aan de WAN
kant. Als dit niet wordt geconfigureerd, kan er **GEEN IP-adres**
worden verkregen.
- IGMP Proxy/Snooping:Inschakelen.
- Hardware Acceleration:Manual voor UDP sessies.

Navigeer in de webinterface van de DrayTek naar WAN >> Multi-VLAN om een vrije WAN
Channel te selecteren. In dit voorbeeld maken we gebruik van WAN5. Deze dient u in te
schakelen om vervolgens het volgende in te stellen:

WAN Type:
Deze dient u op ADSL, VDSL of Ethernet (WAN1 of
WAN2) te zetten. (afhankelijk van welke WAN poort er
gebruikt wordt).
VLAN Tag:
Geef hier het VLAN Tag voor IPTV op, bij KPN is
dit VLAN Tag 4.
Open WAN Interface :
Zet deze op Enable en vink IPTV aan, zet de WAN Setup
vervolgens op Static or Dynamic IP.
WAN IP Network Settings:
Vink hier Obtain an IP Address Automatically aan.

![routed1.png](/images/kpn/draytek/routed1.png)

Om een IP-adres te verkrijgen op WAN 5 (Virtual WAN) dient u Option 60 IPTV_RG in te
stellen. Wanneer dit niet is ingesteld, krijgt u ook geen IP-adres op de WAN verbinding.
Navigeer naar WAN >> Internet Access en klik op de knop DHCP Client Option.

![routed2.png](/images/kpn/draytek/routed2.png)

Zet hier een vinkje bij Enable en vul hier het volgende in:
Interface:WAN5 (afhankelijk van de virtuele WAN die gebruikt wordt op pagina 5)
Option:60
DataType:ASCII Character
Data:IPTV_RG

![routed3.png](/images/kpn/draytek/routed3.png)
Klik op Add om de betreffende regel toe te voegen aan de DHCP Option lijst.

Wanneer u op Add hebt geklikt krijgt u de onderstaande regel in de Options List te zien:
![routed4.png](/images/kpn/draytek/routed4.png)

Klik op OK om de instellingen op te slaan.
Controleer bij “Online Status >> Virtual WAN” of er een IP adres op de WAN5 interface
wordt verkregen. In het onderstaande voorbeeld kunt u zien dat de WAN5 interface online
is, waarbij een IP adres wordt verkregen van 10.0.0.68 (dit kan ook een ander IP adres zijn).

![routed5.png](/images/kpn/draytek/routed5.png)
De WAN5 configuratie is afgerond, vervolgens dient u de onderstaande stappen te volgen
voor het configureren van IGMP Proxy en Snooping.

### **IGMP Proxy & IGMP Snooping**

Om de IPTV pakketten beter te begeleiden dient IGMP Proxy en IGMP Snooping te worden
ingeschakeld. Navigeer in het menu naar Application >> IGMP, schakel zowel IGMP Proxy,
IGMP Snooping als IGMP Fast Leave in. Selecteer bij Interface PVC/VLAN en zorg ervoor
dat de IGMP version op Auto staat.

***Let op: Wanneer u meerdere IPTV ontvangers op één enkele LAN poort gebruikt
(bijvoorbeeld door middel van een switch), dient u de optie IGMP Fast Leave uit te
schakelen!***

![igmp1.png](/images/kpn/draytek/igmp1.png)
Klik op OK om de instellingen op te slaan.

**Hardware Acceleration**

De Hardware Acceleration moet ingesteld worden (m.u.v. V3910/V2962 serie), zodat
Multicast verkeer door de DrayTek modem/router versneld wordt. Dit heeft als voordeel
dat de processor van de router niet extra belast wordt.

Navigeer in het hoofdmenu naar Hardware Acceleration. Zet hierbij de Mode op **Enable**
en het NAT Protocol op **UDP**.

**Opmerking:** Wanneer u een internet abonnement heeft met een snelheid van meer dan
300Mbps dient u ook het vinkje voor TCP in te schakelen om de maximale snelheden te kunnen
behalen.

![igmp2.png](/images/kpn/draytek/igmp2.png)

De aanwezige IPTV decoder(s) kunt u nu aansluiten op uw DrayTek modem/router. Op
basis van de bovenstaande instellingen zal de IPTV decoder(s) online moeten komen.

**Software en updates TV decoder**

Na het opstarten van de TV decoder zal de decoder een software/update proberen te
downloaden vanaf het IPTV platform. Om deze software/updates binnen te kunnen halen,
dient u in de DrayTek te navigeren naar “Applications >> IGMP >> Working status” en
zet u een vinkje bij Unblock achter de betreffende weergegeven IP range.
Klik vervolgens op OK.

![igmp3.png](/images/kpn/draytek/igmp3.png)

**Het kan zijn dat de TV decoder hierna nog één keer opnieuw moet worden opgestart.**

**Waarom heeft mijn TV stream hakkelig of vastlopend beeld?**

- Controleer of DoS Defense onder “Firewall >> Defense Setup” uit staat. Wanneer de
 functie Enable UDP flood defense aan staat gevinkt, kan dit problemen opleveren met
 IPTV.
- Controleer of de Data Flow Monitor uit staat onder “Diagnostics >> Data Flow Monitor”.
 Wanneer deze functie aanstaat, zal de Hardware Acceleration (die nodig is om IPTV
 pakketjes versneld door te sturen) niet actief worden.
 
 