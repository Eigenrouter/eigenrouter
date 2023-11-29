---
title: Draytek KPN Internet
description: 
published: true
date: 2023-11-29T21:01:07.994Z
tags: 
editor: markdown
dateCreated: 2023-11-29T20:48:18.547Z
---

# Draytek KPN Internet Only
**KPN**
**Type verbinding(en): ADSL, VDSL & Fiber**

**ADSL instellingen**
Navigeer in het menu van de DrayTek naar “WAN >> General Setup >> WAN 1”.
Zet de VLAN Tag insertion onder het kopje “Customer” op Enable om
vervolgens Tag value 6 op te geven. De Priority kan op 0 blijven staan.  

**Particulier ADSL:**
PPPoA
VPI: 8
VCI: 48  

**Zakelijk ADSL:**
PPPoA
VPI: 2
VCI: 32

**VDSL instellingen**
**PPPoE – VLAN tag 6**

Username/password (dit kunt u zelf kiezen, maar dient wel ingevuld te worden)
Dynamic IP
Navigeer in het menu van de DrayTek naar “WAN >> General Setup >> WAN 1”.
Zet de VLAN Tag insertion onder het kopje “Customer” op Enable om
vervolgens Tag value 6 op te geven. De Priority kan op 0 blijven staan.

Klik op OK om de instellingen op te slaan. Bij WAN >> Internet Access
configureert u de WAN poort als PPPoE.

username: kpn
Password: kpn

Wan connection detection: PPP Detect
MTU: 1492
PPP/MP setup
Authentication: pap or chap
idle timeout: -1
Ip Assingment: Dynamic
Dail out schedule on none
TTL: set maker on
Default mac address

**Fiber instellingen**
**PPPoE – VLAN tag 6**
Username/password (dit kunt u zelf kiezen, maar dient wel ingevuld te worden)
Dynamic IP
Navigeer in het menu van de DrayTek naar “WAN >> General Setup >> WAN 1”.
Zet de VLAN Tag insertion onder het kopje “Customer” op Enable om
vervolgens Tag value 6 op te geven. De Priority kan op 0 blijven staan.

# Aan deze handleiding word gewerkt....