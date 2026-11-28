# Segmented Network Deployment: OPNsense, Cisco Catalyst 3650 & Mobility Express

> A four-VLAN network for a music school that isolates students from staff, AV, and management, built on an OPNsense firewall, a Cisco Catalyst 3650 PoE switch, and Cisco Aironet 3802 access points running Mobility Express.

*IP ranges and port numbers are examples or sanitized. No credentials or identifying information are included.*

## Table of Contents
- [Overview](#overview)
- [My Role](#my-role)
- [Objectives](#objectives)
- [Infrastructure](#infrastructure)
- [Topology](#topology)
- [VLAN & IP Addressing](#vlan--ip-addressing)
- [Management Network](#management-network)
- [Device Placement](#device-placement)
- [Wireless Design](#wireless-design)
- [Firewall & Security Policy](#firewall--security-policy)
- [Cisco Catalyst 3650 Configuration](#cisco-catalyst-3650-configuration)
- [OPNsense Configuration](#opnsense-configuration)
- [Mobility Express Configuration](#mobility-express-configuration)
- [AV Network Considerations](#av-network-considerations)
- [Limitations & Future Improvements](#limitations--future-improvements)
- [Management & Maintenance](#management--maintenance)
- [Skills Demonstrated](#skills-demonstrated)

## Overview
This project deploys a segmented network using a Cisco Catalyst 3650 PoE switch, Cisco Aironet 3802 access points running Mobility Express, and an OPNsense firewall/router.

The primary goal is to **isolate the student network from staff, AV, and management**, while letting staff and AV devices reach the services they need and giving every network internet access where permitted.

The network uses four VLANs (staff, AV, student, and management) and three wireless SSIDs. OPNsense handles routing, DHCP, DNS, and firewall enforcement. The Catalyst 3650 provides switching and PoE. The 3802 APs provide wireless connectivity through Mobility Express.

## My Role
- Designed the VLAN scheme, addressing plan, and firewall policy
- Installed and terminated the Cat6A cabling
- Built and configured the OPNsense firewall: VLAN interfaces, inter-VLAN routing, DHCP, firewall rules, NAT, and DNS filtering
- Configured the Catalyst 3650 as a Layer 2 switch: VLANs, trunk and access ports, PoE, and the management port
- Deployed Cisco Mobility Express wireless: three SSIDs, QoS, and bandwidth limits across six wired APs
- Put the ISP modem into bridge mode so OPNsense is the only router and NAT device
- Tested and documented the deployment

## Objectives
- Isolate students from staff, AV, and network management
- Provide internet access to the staff, AV, and student VLANs
- Keep classroom displays and the office printer on the STAFF VLAN
- Keep the Behringer X32, six Shure SLXD units, and the Waves SoundGrid control PC on the AV VLAN
- Let the AV VLAN reach the STAFF VLAN, including the printer and classroom displays, and nothing else internal
- Put network management on a dedicated VLAN, reachable through a physical management port on the switch
- Provide three dedicated wireless SSIDs with different QoS priority and bandwidth
- Enforce inter-VLAN access restrictions through OPNsense
- Centralize wireless management with Cisco Mobility Express

## Infrastructure

| Device | Role |
|---|---|
| OPNsense (HP ProDesk, 10th-gen Core i3, 8 GB RAM) | Firewall, router, DHCP, DNS, and VLAN gateway |
| LR-LINK LRES3004PT (4-port PCIe, Intel i210) | Network interfaces for OPNsense (WAN and VLAN trunk) |
| Cisco DPC3939B | ISP modem in bridge mode |
| Cisco Catalyst 3650 PoE | Managed switch: VLAN switching and PoE |
| Cisco Aironet 3802 (x6) | Wireless access points: one controller (primary/master) AP and five subordinate APs, all wired to the 3650 over PoE |
| Cisco Mobility Express 8.10.196.0 | Wireless controller, running on one AP |
| Behringer X32 | Digital audio console; REMOTE Ethernet port on the AV VLAN, Waves expansion card LAN port on the separate SoundGrid network |
| Klark Teknik DN32-WSG | Waves SoundGrid expansion card in the X32; its LAN port connects to the separate SoundGrid network |
| Behringer S32 stage box | Remote stage box on the X32's AES50 port A (digital audio, not a network connection) |
| Shure SLXD receivers (x6) | Wireless microphone receivers |
| Waves SoundGrid control PC | Runs SoundGrid SuperRack; connects to this network over AV-WIFI and to the separate SoundGrid network over its wired LAN interface |
| Waves SoundGrid server | Audio processing server on the separate SoundGrid network, alongside the X32's DN32-WSG card and the control PC's wired interface; controlled from SuperRack on the control PC; not connected to this network |
| SoundGrid switch | Dedicated switch for the SoundGrid audio network: the Waves server, the X32's DN32-WSG expansion card, and the control PC's wired interface; not connected to this network |
| Classroom displays | Classroom display equipment |
| Office printer | Office printing |
| Cabling | Cat6A |

## Topology

```mermaid
flowchart TD
    Internet((Internet))
    Modem[Cisco DPC3939B<br/>Bridge mode]
    FW[OPNsense Firewall]
    SW[Cisco Catalyst 3650 PoE]
    AP[Cisco Aironet 3802 APs<br/>Mobility Express]

    Internet --> Modem
    Modem --> FW
    FW -->|802.1Q VLAN trunk| SW
    SW -->|802.1Q trunks| AP

    AP --> StaffWiFi[STAFF-WIFI<br/>VLAN 10]
    AP --> AVWiFi[AV-WIFI<br/>VLAN 20]
    AP --> StudentWiFi[STUDENT-WIFI<br/>VLAN 30]
    AVWiFi --> SG[Waves SoundGrid control PC<br/>VLAN 20, Wi-Fi]
    SG ---|wired LAN, no connection to this network| SGNet[Separate SoundGrid network<br/>dedicated switch: Waves server,<br/>DN32-WSG card, control PC]

    SW --> Displays[Classroom Displays<br/>VLAN 10]
    SW --> Printer[Office Printer<br/>VLAN 10]
    SW -->|REMOTE port| X32[Behringer X32<br/>VLAN 20]
    X32 ---|Waves expansion card LAN port, no connection to this network| SGNet
    X32 -.->|AES50 port A, digital audio only| S32[Behringer S32 stage box]
    SW --> Shure[6x Shure SLXD<br/>VLAN 20]
    SW --> Mgmt[Port 24: Management<br/>VLAN 99]
```

## VLAN & IP Addressing

| VLAN ID | Name | Subnet | Gateway | DHCP |
|---|---|---|---|---|
| 10 | STAFF | 192.168.10.0/24 | 192.168.10.1 | OPNsense |
| 20 | AV | 192.168.20.0/24 | 192.168.20.1 | OPNsense |
| 30 | STUDENT | 192.168.30.0/24 | 192.168.30.1 | OPNsense |
| 99 | MANAGEMENT | 192.168.99.0/24 | 192.168.99.1 | OPNsense (small scope) |

| VLAN | Purpose | Devices |
|---|---|---|
| **10 - STAFF** | Staff computers, classroom displays, office printer, approved administrative devices | Staff computers and laptops, classroom displays, office printer, staff wireless devices |
| **20 - AV** | Dedicated network for audio and AV equipment | Behringer X32, six Shure SLXD receivers, Waves SoundGrid control PC, AV control equipment, approved AV wireless devices |
| **30 - STUDENT** | Student internet access with no access to internal infrastructure | Student laptops, tablets, phones, wireless clients |
| **99 - MANAGEMENT** | Administration of the network infrastructure | Switch, AP/Mobility Express, and OPNsense management; admin workstation on switch port 24 |

## Management Network

Network management lives on its own VLAN (**VLAN 99**), separate from the user networks.

- **Management port:** switch port 24 (`GigabitEthernet1/0/24`) is an access port on VLAN 99. An administrator plugs in a workstation here to manage the network.
- **Switch management:** the 3650 has an IP address on VLAN 99 (192.168.99.2) with OPNsense (192.168.99.1) as its gateway.
- **OPNsense:** a MANAGEMENT interface on the VLAN 99 trunk. The OPNsense web GUI and SSH are restricted to this interface.
- **Mobility Express / APs:** the APs and the Mobility Express controller are on VLAN 99, carried as the untagged (native) VLAN on the AP trunk ports. The SSID VLANs (10, 20, 30) are tagged. VLAN 1 is not used.
- **Access policy:** devices on VLAN 99 can reach all VLANs and the infrastructure. No other VLAN can initiate connections into VLAN 99.
- **Addressing:** infrastructure devices (switch, APs, the Mobility Express controller) use static addresses in the .2-.99 range, outside the DHCP scope. The DHCP scope (.100-.110) covers only the admin workstation.

### Native VLAN Design Note
Best practice is normally to set the native VLAN on trunks to an unused "blackhole" VLAN and to tag the management VLAN. This deployment does not use a blackhole VLAN. On the AP trunks, **VLAN 99 (management) is the native VLAN**, because the Mobility Express controller and the APs' management and CAPWAP traffic are carried **untagged** on the wire. The client VLANs (10, 20, 30) remain tagged.

**Does this make VLAN 99 unsafe?** No. A native VLAN is not an unprotected VLAN. The security of VLAN 99 depends on what is permitted into and out of it, not on whether it is native. The design is hardened as follows:
- VLAN 99 access is limited to the dedicated administrator workstation and trusted infrastructure
- STAFF, AV, and STUDENT cannot initiate connections to management interfaces
- Management services (SSH, HTTPS, SNMP) are restricted to authorized sources
- VLAN 99 is not assigned to any staff or AV access port, and there are no wired student ports. The only access port in VLAN 99 is the physically controlled management port (port 24)
- OPNsense firewall rules restrict access to management IP addresses
- Unused switch ports are disabled
- Management interfaces are not exposed to the WAN

The classic native-VLAN risk is VLAN hopping by double tagging, which requires an attacker on an access port in the native VLAN. Keeping every untrusted port out of VLAN 99 closes that path.

## Device Placement

| Device | VLAN | Network | Connection |
|---|---|---|---|
| Classroom displays | 10 | STAFF | Wired |
| Office printer | 10 | STAFF | Wired |
| Staff devices | 10 | STAFF | Wired / STAFF-WIFI |
| Behringer X32 | 20 | AV | Wired (REMOTE port); Waves expansion card LAN port to the separate SoundGrid network |
| Shure SLXD receivers 1-6 | 20 | AV | Wired |
| Waves SoundGrid control PC | 20 | AV | AV-WIFI (plus a wired NIC to the separate SoundGrid network) |
| Student devices | 30 | STUDENT | STUDENT-WIFI only (no wired student ports) |
| Admin workstation (switch port 24) | 99 | MANAGEMENT | Wired |
| Switch, OPNsense, AP management | 99 | MANAGEMENT | Wired |

Devices are assigned to their VLAN through switch access ports, wireless SSID assignment, or approved network configurations.

## Wireless Design

Six Cisco Aironet 3802 APs provide wireless coverage, all managed by Cisco Mobility Express.

### Controller: Cisco Mobility Express 8.10.196.0
Mobility Express runs the wireless controller function inside one of the Aironet 3802 APs, and the other APs join it as subordinate APs. It handles management and policy: SSIDs, SSID-to-VLAN mapping, QoS, bandwidth limits, RF management, and roaming. This avoided buying a dedicated wireless controller.

### AP deployment
- **6 APs in total:** one controller AP (called the primary AP in the 8.10 documentation and the master AP in older guides) running Mobility Express, and five subordinate APs.
- **All six APs are wired to the Catalyst 3650** over Cat6A and powered by PoE, so there is no wireless backhaul between APs.
- All APs broadcast the same three SSIDs, so clients can roam between them.
- The latency-sensitive AV equipment (the X32 and the Shure receivers) is wired. The SoundGrid control PC uses Wi-Fi for this network because only control traffic crosses it; its wired LAN interface connects to the SoundGrid audio network, which is on a separate switch.

### SSIDs, QoS, and Bandwidth
All six APs broadcast the same three SSIDs, each mapped to a VLAN with its own QoS priority and bandwidth allocation.

| SSID | VLAN | Purpose | QoS priority | Bandwidth limit |
|---|---|---|---|---|
| STAFF-WIFI | 10 | Staff wireless access | Second highest | 500 Mbps |
| AV-WIFI | 20 | AV control devices, including the SoundGrid control PC | Highest | No limit |
| STUDENT-WIFI | 30 | Student wireless access | Lowest | 10 Mbps maximum |

- **QoS priority** is highest for AV, then staff, then students, so AV control traffic is served first when airtime is busy.
- **Bandwidth limits** keep student traffic (10 Mbps maximum) from starving staff and AV traffic on the single ISP connection. Staff is capped at 500 Mbps and AV is uncapped.
- **SSID-to-VLAN mapping** applies the same segmentation to wireless as to the wired network.

**STAFF-WIFI (VLAN 10, 192.168.10.0/24)**
- Intended for staff devices
- Access to approved internal resources, classroom displays, and the office printer
- Internet access

**AV-WIFI (VLAN 20, 192.168.20.0/24)**
- Intended for approved AV devices and controllers
- Anyone on AV-WIFI can remotely control the SoundGrid system through the app on the control PC
- Internet access where required
- Access to the STAFF VLAN (printer and classroom displays)
- No access to STUDENT or management
- Separate credentials from the staff and student networks

**STUDENT-WIFI (VLAN 30, 192.168.30.0/24)**
- Internet access only
- No access to STAFF, AV, or management

## Firewall & Security Policy

OPNsense enforces traffic restrictions between VLANs.

**Security objectives**
- Students cannot initiate connections to STAFF, AV, or MANAGEMENT
- AV can reach the STAFF VLAN (the printer and classroom displays) but not STUDENT or MANAGEMENT
- Staff can reach approved AV equipment for control and operation
- Only the MANAGEMENT VLAN can administer the firewall, switch, and APs
- All user VLANs have internet access, subject to firewall policy
- DNS and DHCP services are provided as required

### Policy Matrix

| Source | Destination | Policy |
|---|---|---|
| STAFF | Internet | Allow |
| STAFF | AV | Allow selectively (control of AV equipment) |
| STAFF | STUDENT | Block |
| STAFF | MANAGEMENT | Block |
| AV | Internet | Allow |
| AV | STAFF | Allow (printer, classroom displays, staff VLAN) |
| AV | STUDENT | Block |
| AV | MANAGEMENT | Block |
| STUDENT | Internet | Allow |
| STUDENT | STAFF | Block |
| STUDENT | AV | Block |
| STUDENT | MANAGEMENT | Block |
| MANAGEMENT | All VLANs | Allow |
| MANAGEMENT | Internet | Allow (updates and firmware) |

Rules are stateful, so replies to allowed connections return automatically. "Block" means the source cannot initiate connections to the destination.

### Rule Implementation
Rules are configured on each VLAN interface in OPNsense.

**STUDENT**
1. Permit required DHCP
2. Permit DNS to the approved resolver
3. Block access to internal/private networks, including MANAGEMENT
4. Allow internet access

**AV**
1. Permit required DHCP and DNS
2. Permit access to the STAFF VLAN
3. Block access to STUDENT and MANAGEMENT
4. Allow internet access

**STAFF**
1. Permit required DHCP and DNS
2. Permit specifically approved AV control traffic
3. Block access to STUDENT and MANAGEMENT
4. Allow internet access

**MANAGEMENT**
1. Permit DHCP and DNS
2. Permit access to all VLANs and infrastructure
3. Allow internet access

> Firewall policy should also account for IPv6, multicast, mDNS, and device-specific control protocols. IPv4 rules alone do not provide complete isolation if IPv6 is enabled.

### Content Filtering
Student web access is restricted with DNS filtering on OPNsense. This blocks selected domains but is not comprehensive web filtering: it does not inspect URLs or content and can be bypassed with alternate DNS, DNS-over-HTTPS, a VPN, or direct IP access.

## Cisco Catalyst 3650 Configuration

<details>
<summary><b>Create VLANs</b></summary>

```text
enable
configure terminal

vlan 10
 name STAFF
exit

vlan 20
 name AV
exit

vlan 30
 name STUDENT
exit

vlan 99
 name MANAGEMENT
exit

end
write memory
```
</details>

<details>
<summary><b>OPNsense trunk port</b></summary>

```text
configure terminal

interface GigabitEthernet1/0/1
 description TRUNK_TO_OPNSENSE
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,99
 no shutdown
exit
```
</details>

<details>
<summary><b>Mobility Express AP trunk ports</b></summary>

All six APs connect on trunk ports. VLAN 99 (management) is the **native VLAN**, so the AP and controller management traffic is untagged on VLAN 99. The SSID VLANs (10, 20, 30) are tagged. VLAN 1 is not used.

```text
interface range GigabitEthernet1/0/2-7
 description MOBILITY_EXPRESS_APS
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 99,10,20,30
 power inline auto
 spanning-tree portfast trunk
 no shutdown
exit
```

Notes:
- The APs get management addresses on VLAN 99 (192.168.99.0/24), and the controller is reachable from the management port.
- Change the controller AP's port first and confirm the controller comes back on its new address before changing the other five ports.
</details>

<details>
<summary><b>Access ports (Staff, AV)</b></summary>

Students connect only over Wi-Fi (STUDENT-WIFI), so there are no wired student ports.

```text
! Staff wired ports (example: 8-12)
interface range GigabitEthernet1/0/8-12
 description STAFF_WIRED
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
 no shutdown
exit

! AV equipment ports (example: 13-18)
interface range GigabitEthernet1/0/13-18
 description AV_EQUIPMENT
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
 no shutdown
exit
```
</details>

<details>
<summary><b>Management port (port 24) and switch management IP</b></summary>

```text
! Management port: admin workstation
interface GigabitEthernet1/0/24
 description MANAGEMENT_PORT
 switchport mode access
 switchport access vlan 99
 spanning-tree portfast
 no shutdown
exit

! Switch management address on the management VLAN
interface Vlan99
 description SWITCH_MANAGEMENT
 ip address 192.168.99.2 255.255.255.0
 no shutdown
exit

! Default gateway to OPNsense (use one, depending on whether ip routing is enabled)
ip default-gateway 192.168.99.1
! ip route 0.0.0.0 0.0.0.0 192.168.99.1

end
write memory
```

Note: in this design OPNsense performs inter-VLAN routing and the switch is Layer 2. The Vlan99 interface only gives the switch a management address; it does not route between user VLANs.
</details>

## OPNsense Configuration

**VLANs** are created on the physical interface connected to the Catalyst 3650.

| VLAN tag | Description | Interface IPv4 |
|---|---|---|
| 10 | STAFF | 192.168.10.1/24 |
| 20 | AV | 192.168.20.1/24 |
| 30 | STUDENT | 192.168.30.1/24 |
| 99 | MANAGEMENT | 192.168.99.1/24 |

**DHCP scopes**

| VLAN | Range |
|---|---|
| STAFF | 192.168.10.100 - 192.168.10.230 |
| AV | 192.168.20.100 - 192.168.20.230 |
| STUDENT | 192.168.30.100 - 192.168.30.230 |
| MANAGEMENT | 192.168.99.100 - 192.168.99.110 |

**Addressing**
- Static DHCP reservations: classroom displays and the office printer
- Static IPs outside the DHCP scope: the Behringer X32 and the Shure SLXD receivers (AV VLAN, .2-.99 range)
- Dynamic DHCP: the Waves SoundGrid control PC, over AV-WIFI
- Static IPs on VLAN 99: the switch, APs, and OPNsense

**Management access:** the OPNsense web GUI and SSH listen on the MANAGEMENT interface and are blocked from the other VLANs.

## Mobility Express Configuration

Cisco Aironet 3802 APs operate under Cisco Mobility Express 8.10.196.0.

| WLAN | VLAN | Security |
|---|---|---|
| STAFF-WIFI | 10 | WPA2/WPA3 Personal |
| AV-WIFI | 20 | WPA2/WPA3 Personal (dedicated credentials) |
| STUDENT-WIFI | 30 | WPA2/WPA3 Personal |

**Configuration method:** the WLANs and their VLAN mappings were configured in the Mobility Express web interface. The switch ports connected to the APs are trunks with VLAN 99 (management) as the native VLAN and VLANs 10, 20, and 30 tagged (see the Catalyst configuration above). The controller's management interface is untagged on VLAN 99.

**Wireless security practices**
- Separate, strong credentials for each SSID
- Staff credentials are not shared with students
- Wireless management restricted to the management VLAN and authorized administrators

## AV Network Considerations

**AV VLAN access:** The AV VLAN can reach the STAFF VLAN so AV equipment and control devices can use the classroom displays and the office printer, which live on that VLAN. AV has no access to the STUDENT or MANAGEMENT VLANs, and students cannot initiate connections into AV.

**Behringer X32 (VLAN 20, multiple connection types)**
- **REMOTE Ethernet port:** the X32's network control port. It connects to a switch access port on the AV VLAN with a static IP, and is used for remote control over the network.
- **Waves expansion card LAN port (SoundGrid):** the LAN port on the X32's Waves expansion card connects to the SoundGrid network on its dedicated switch, which is completely separate from this network. The card is a Klark Teknik DN32-WSG.
- **AES50 ports:** not network ports. They carry digital audio over Cat5e-style cabling and are never connected to the switch or any part of this network. A Behringer S32 stage box connects to AES50 port A.
- Kept on the dedicated AV network
- Student access blocked; staff control devices allowed where necessary
- No unnecessary internet exposure

**Shure SLXD receivers x6 (VLAN 20)**
- Static IPs preferred (DHCP reservations where static is not supported)
- Kept on the AV VLAN
- Required Shure management software can communicate with the receivers
- Management interfaces not exposed to the student VLAN

**Waves SoundGrid control PC (VLAN 20, dual-homed)**
- Runs the SoundGrid SuperRack software used to control the Waves server
- **Interface 1 (Wi-Fi, AV-WIFI):** joins VLAN 20 by DHCP so anyone on AV-WIFI can remotely control the system through the app
- **Interface 2 (wired LAN):** connects to the SoundGrid network, which is completely separate and runs on its own switch
- The SoundGrid network has no connection to the rest of this network. The Waves server, the X32's DN32-WSG card, and this PC's wired interface all sit on that separate network, so all SoundGrid audio and its control path to the server stay there. Only the remote-control traffic from AV-WIFI reaches the PC.
- Needs to reach the X32 and other audio equipment on the AV VLAN

> Some SLXD variants and configurations may not have network management connectivity. Confirm the exact receiver model and network capabilities before assigning ports or addresses.

## Limitations & Future Improvements
- **No firewall redundancy.** A single OPNsense box is a single point of failure. A second unit with CARP failover, or at least a backup config and spare hardware, would fix that.
- **DNS filtering is not full web filtering.** Students can bypass it with alternate DNS, DNS-over-HTTPS, or a VPN. Redirecting or blocking outbound DNS at the firewall helps, and a proxy or next-generation firewall would be needed for comprehensive filtering.
- **Mobility Express is end-of-life.** Cisco has retired Mobility Express, and the Aironet 3800 series is also end-of-life, so this wireless platform no longer receives support. A supported replacement would be a Catalyst 9800 series controller or the Cisco Embedded Wireless Controller on a Catalyst 9100 series AP.
- **Shared Wi-Fi credentials.** Per-user 802.1X authentication would be stronger than shared passphrases.
- **Native VLAN.** The AP trunks use the management VLAN as native because of how Mobility Express carries management and CAPWAP traffic. This is mitigated by firewall policy, but a different AP management design that tags management traffic would allow a blackhole native VLAN.

## Management & Maintenance
- Administer the network only from the MANAGEMENT VLAN (switch port 24)
- Back up the OPNsense and Cisco switch configurations
- Document switch port assignments
- Maintain DHCP reservations and a device inventory
- Update AP and Mobility Express firmware carefully
- Monitor firewall logs and DHCP leases
- Review firewall rules periodically
- Re-test VLAN isolation after major changes

## Skills Demonstrated
VLAN segmentation, 802.1Q trunking, management VLAN design, switch hardening, inter-VLAN routing, OPNsense firewall rule design, DHCP and DNS, DNS filtering, Cisco IOS-XE, PoE, Cisco Mobility Express wireless, WLAN QoS and bandwidth management, SSID-to-VLAN mapping, AV network planning (Behringer X32, Shure SLXD, Waves SoundGrid), network documentation
