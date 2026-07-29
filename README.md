# Enterprise-3Floor-Redundant-Network-Design
Enterprise multi-floor Cisco network design featuring ISP multi-homing redundancy, Layer 3 Core SVI routing, OSPF Area 0, Centralized DHCP Relay, PAT NAT, Access Control Lists, and Secure WLAN deployment.

# Project Overview & Scenario
This project models a 3-floor corporate campus network designed for maximum uptime, high availability, and secure departmental isolation.

The enterprise infrastructure connects 6 primary business departments across three floors, complete with a dedicated Server Room DMZ and Dual-ISP External Connectivity for redundant internet reachability.

+-------------------------+
                  |  Dual ISPs (ISP1/ISP2)  |
                  +------------+------------+
                               |
              +----------------+----------------+
              |                                 |
     +--------+-------+                +--------+-------+
     | Core Router R1 |                | Core Router R2 |
     +--------+-------+                +--------+-------+
              |                                 |
              +----------------+----------------+
                               |
              +----------------+----------------+
              |                                 |
     +--------+-------+                +--------+-------+
     | Multilayer SW1 |================| Multilayer SW2 |
     +--------+-------+                +--------+-------+
              |                                 |
  +-----------+-----------+         +-----------+-----------+
  |  Floor 1  |  Floor 2  |         |  Floor 3  | Server DMZ|
  | VLAN 10/20| VLAN 30/40|         |  VLAN 50  |  VLAN 60  |
  +-----------+-----------+         +-----------+-----------+

# Key Technologies & Architecture Implemented

1. Hierarchical Network Design: Core, Distribution (Multilayer), and Access Layers for modular scaling.

2. Redundant ISP Multi-Homing: Dual WAN connections running default routes with administrative distance metrics (ip route 0.0.0.0 0.0.0.0 Serial0/3/1 70) for failover pathing.

3. Dynamic Routing (OSPF Area 0): Full internal backbone routing connecting Core Routers, Multilayer Distribution Switches, and WAN transit links.

4. Layer 3 Switch Virtual Interfaces (SVIs): Distributed default gateways hosted directly on Layer 3 switches for high-speed inter-VLAN routing.

5. Centralized DHCP Relay Architecture: Single centralized DHCP server servicing dynamic IP allocations across all departmental VLANs via ip helper-address.

6. Network Address Translation (NAT / PAT): Port Address Translation via Standard Access Lists permitting internal subnets out through WAN interfaces.

7. Layer 2 Security & Access Controls: Switchport access mode enforcement, banner motd legal warnings, and encrypted SSH (v2) remote management.

8. Secure Enterprise WLAN: Departmental Wireless Access Points configured with WPA2-PSK (AES Encryption).

# Network Addressing & VLAN Allocation
Floor Location,Department / Zone,VLAN ID,Subnet CIDR,Subnet Mask,Default Gateway,Allocated Hosts Range
Floor 1,Sales & Marketing,VLAN 10,172.16.1.0/25,255.255.255.128,172.16.1.1,172.16.1.2 - 172.16.1.126
Floor 1,Human Resources (HR),VLAN 20,172.16.1.128/25,255.255.255.128,172.16.1.129,172.16.1.130 - 172.16.1.254
Floor 2,Finance & Accounts,VLAN 30,172.16.2.0/25,255.255.255.128,172.16.2.1,172.16.2.2 - 172.16.2.126
Floor 2,Administration,VLAN 40,172.16.2.128/25,255.255.255.128,172.16.2.129,172.16.2.130 - 172.16.2.254
Floor 3,ICT Department,VLAN 50,172.16.3.0/25,255.255.255.128,172.16.3.1,172.16.3.2 - 172.16.3.126
Floor 3,Server Room DMZ,VLAN 60,172.16.3.128/28,255.255.255.240,172.16.3.129,172.16.3.130 - 172.16.3.142

#Server DMZ Infrastructure
Server Host,Static IP Address,Subnet Mask,Function / Services Provided
DNS Server,172.16.3.131,255.255.255.240,"Resolves Domain Names (e.g., [www.cisco.com](https://www.cisco.com) -> 172.16.3.131)"
DHCP Server,172.16.3.132,255.255.255.240,"Hosts Central Pools (Sales&MarketingPool, HRPool, FinancePool, etc.)"
Email Server,172.16.3.133,255.255.255.240,Enterprise SMTP / POP3 Messaging

#Key Configuration Highlights
1. Centralized Inter-VLAN SVI & DHCP Relay (Multilayer Switches)
! Interface SVI Configuration with DHCP Relay
interface Vlan10
 description Sales_and_Marketing_Gateway
 ip address 172.16.1.1 255.255.255.128
 ip helper-address 172.16.3.132
!
interface Vlan60
 description Server_Room_Gateway
 ip address 172.16.3.129 255.255.255.240
 ip helper-address 172.16.3.132
!
ip routing

2. Core Routing & NAT / PAT Overload Setup (Core Routers)
! Standard ACL permitting internal subnets
access-list 1 permit 172.16.1.0 0.0.0.127
access-list 1 permit 172.16.1.128 0.0.0.127
access-list 1 permit 172.16.2.0 0.0.0.127
access-list 1 permit 172.16.2.128 0.0.0.127
access-list 1 permit 172.16.3.0 0.0.0.127
access-list 1 permit 172.16.3.128 0.0.0.15

! PAT Translation linked to primary WAN interface
ip nat inside source list 1 interface Serial0/3/1 overload

! Primary and Floating Default Static Routes
ip route 0.0.0.0 0.0.0.0 Serial0/3/0
ip route 0.0.0.0 0.0.0.0 Serial0/3/1 70

3. Dynamic OSPF Backbone Configuration
router ospf 10
 router-id 3.3.3.3
 network 195.136.17.0 0.0.0.3 area 0
 network 195.136.17.12 0.0.0.3 area 0
 network 172.16.3.144 0.0.0.3 area 0
 network 172.16.3.152 0.0.0.3 area 0

# Verification & Connectivity Testing
- Dynamic IP Assignment Verification:

- Host machines successfully receive IP addresses, subnet masks, default gateways, and DNS server (172.16.3.131) settings from the central DHCP server across L3 boundaries.

- End-to-End ICMP Connectivity:

- Executed ICMP ping sweeps from Admin Host PC0(3) (172.16.2.132):

- Gateway Reachability: ping 172.16.2.129 (SUCCESS)

- Inter-VLAN Reachability: ping 172.16.1.3 (SUCCESS)

- DMZ Server Reachability: ping 172.16.3.133 (SUCCESS)

- WLAN Connectivity:

- Mobile clients and tablets successfully associate with ICT_WIFI via WPA2-PSK AES authentication on Channel 6.





