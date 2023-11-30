---
title: Draytek KPN Internet
description: 
published: true
date: 2023-11-30T09:16:49.837Z
tags: 
editor: markdown
dateCreated: 2023-11-29T20:48:18.547Z
---

# Draytek KPN Internet Only
Type verbinding(en): ADSL, VDSL & Fiber

ADSL instellingen
Navigeer in het menu van de DrayTek naar “WAN >> General Setup >> WAN 1”.
Zet de VLAN Tag insertion onder het kopje “Customer” op Enable om
vervolgens Tag value 6 op te geven. De Priority kan op 0 blijven staan.
**Particulier ADSL:
PPPoA
VPI: 8
VCI: 48**
**Zakelijk ADSL:
PPPoA
VPI: 2
VCI: 32**

![image.png](/images/kpn/draytek/image.png)

**VDSL instellingen
PPPoE – VLAN tag 6**
Username/password (dit kunt u zelf kiezen, maar dient wel ingevuld te worden)
Dynamic IP
Navigeer in het menu van de DrayTek naar “WAN >> General Setup >> WAN 1”.
Zet de VLAN Tag insertion onder het kopje “Customer” op Enable om
vervolgens Tag value 6 op te geven. De Priority kan op 0 blijven staan.

![vdsl1.png](/images/kpn/draytek/vdsl1.png)

Klik op OK om de instellingen op te slaan. Bij **WAN >> Internet Access**
configureert u de WAN poort als PPPoE.

![vdsl2.png](/images/kpn/draytek/vdsl2.png)

**Fiber instellingen**
**PPPoE – VLAN tag 6**
Username/password (dit kunt u zelf kiezen, maar dient wel ingevuld te worden)
Dynamic IP

Navigeer in het menu van de DrayTek naar **“WAN >> General Setup >> WAN 1”**.
Zet de VLAN Tag insertion onder het kopje **“Customer”** op Enable om
vervolgens Tag value 6 op te geven. De Priority kan op 0 blijven staan.

![fiber1.png](/images/kpn/draytek/fiber1.png)

Klik op OK om de instellingen op te slaan. Bij **WAN >> Internet Access**
configureert u de WAN poort als PPPoE.

![fiber2.png](/images/kpn/draytek/fiber2.png)

Klik op OK om de instellingen op te slaan. Na deze aanpassing zal internettoegang
op de DrayTek mogelijk moeten zijn.
