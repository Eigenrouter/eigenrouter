---
title: Online Internet Only
description: 
published: true
date: 2023-12-02T18:07:32.070Z
tags: 
editor: markdown
dateCreated: 2023-12-02T18:07:32.070Z
---

# ONLINE Internet Only

**Type verbinding: ADSL &VDSL**

**ADSL instellingen**
VPI: 8
VCI: 35
DHCP / MPoA

Bij WAN >> Internet Access kunt u de WAN poort configureren als MPoA / Static
or Dynamic IP. Belangrijk is hierbij het aanvinken van ‘Enable’ en ‘Obtain an IP
address automatically’. Geef daarnaast de juiste VPI en VCI waardes.

![online1.png](/images/online/draytek-internet-only/online1.png)

**VDSL instellingen**
DHCP – VLAN tag 100

Navigeer in het menu van de DrayTek naar “WAN >> General Setup >> WAN 1”.
Zet de VLAN Tag insertion onder het kopje “Customer” op Enable om
vervolgens Tag value 1001 op te geven. De Priority kan op 0 blijven staan.

![online2.png](/images/online/draytek-internet-only/online2.png)

Bij WAN >> Internet Access kunt u de WAN poort verder configureren als DHCP
client. Belangrijk is hierbij het aanvinken van ‘Enable’ en ‘Obtain an IP address
automatically’.

![online3.png](/images/online/draytek-internet-only/online3.png)

Je verbinding zou nu moeten werken!

**Weet jij de instellingen voor Fiber? Laat het ons weten ;)**