---
title: SOLCON Internet Only
description: 
published: true
date: 2023-12-02T18:21:26.941Z
tags: 
editor: markdown
dateCreated: 2023-12-02T18:21:26.941Z
---

# SOLCON Internet Only

**Solcon
Type verbinding: ADSL, VDSL & Fiber**

ADSL instellingen
VPI: 0
VCI: 36
PPPoA met PPP credentials (gebruikersnaam / wachtwoord)

*Gebruikersnaam en wachtwoord zijn bekend bij je provider*

Navigeer in het menu van de DrayTek router naar WAN >> Internet Access om
hier de WAN poort als PPPoE / PPPoA te configureren.

![solcon1.png](/images/solcon/solcon1.png)

**VDSL instellingen**
PPPoE – VLAN tag 6
Gebruikersnaam / wachtwoord *(bekend bij provider)*
Dynamic IP

Navigeer in het menu van de DrayTek naar “WAN >> General Setup >> WAN 1”.
Zet de VLAN Tag insertion onder het kopje “Customer” op Enable om
vervolgens Tag value 6 op te geven. De Priority kan op 0 blijven staan.

![solcon2.png](/images/solcon/solcon2.png)

Klik op OK om de instellingen op te slaan. Bij WAN >> Internet Access
configureert u de WAN poort als PPPoE.

![solcon3.png](/images/solcon/solcon3.png)


**Fiber instellingen**
DHCP – Untagged
Dynamic IP

Bij WAN >> Internet Access kunt u de WAN poort configureren als DHCP client.
Belangrijk is hierbij het aanvinken van ‘Enable’ en ‘Obtain an IP address
automatically’.

![solcon4.png](/images/solcon/solcon4.png)

