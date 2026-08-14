# Secure Multi-Branch Banking Network Infrastructure
## Cisco Packet Tracer — Complete Step-by-Step Build Reference

---

## 1. Project Overview

### Project Title

**Secure Multi-Branch Banking Network Infrastructure**

### Objective

Design and implement a secure enterprise banking network connecting:

- Head Office
- Branch 1
- Branch 2
- Branch 3

The network will demonstrate:

- VLAN
- Inter-VLAN Routing
- Router-on-a-Stick
- DHCP
- OSPF Dynamic Routing
- WAN connectivity
- DNS
- HTTP Banking Server
- ACL-based security
- Departmental network segmentation
- End-to-end connectivity testing

---

# 2. Final Network Architecture

```text
                         HEAD OFFICE
                    ┌─────────────────┐
                    │      R1-HQ      │
                    │    Cisco 2911   │
                    └───────┬─────────┘
                            │ G0/0
                            │
                          SW1-HQ
                            │
          ┌─────────┬───────┼────────┬────────┬─────────┐
          │         │       │        │        │
       Admin      Employee Finance    IT      ATM    Server
       VLAN 10    VLAN 20  VLAN 30  VLAN 40 VLAN 50 VLAN 60
                            │
                            │
                    R1 G0/1
                 172.16.12.1/30
                            │
                            │
                    172.16.12.2/30
                         R2-BR1
                            │
                          SW2-BR1
                            │
             ┌──────────────┼──────────────┐
             │              │              │
           Mgmt           Employee       Finance
           VLAN 10        VLAN 20        VLAN 30
                            │
                            │
                     R2 G0/2
                  172.16.23.1/30
                            │
                     172.16.23.2/30
                            │
                         R3-BR2
                            │
                          SW3-BR2
                            │
             ┌──────────────┼──────────────┐
             │              │              │
           Mgmt           Employee       Finance
           VLAN 10        VLAN 20        VLAN 30
                            │
                            │
                     R3 G0/2
                  172.16.34.1/30
                            │
                     172.16.34.2/30
                            │
                         R4-BR3
                            │
                          SW4-BR3
                            │
             ┌──────────────┼──────────────┐
             │              │              │
           Mgmt           Employee       Finance
           VLAN 10        VLAN 20        VLAN 30
```

---

# 3. Devices Required

## Routers

Add:

- 4 × Cisco 2911

Rename them:

```text
R1-HQ
R2-BRANCH1
R3-BRANCH2
R4-BRANCH3
```

## Switches

Add:

- 4 × Cisco 2960-24TT

Rename:

```text
SW1-HQ
SW2-BRANCH1
SW3-BRANCH2
SW4-BRANCH3
```

## End Devices

### Head Office

- PC-HQ-ADMIN
- PC-HQ-EMP
- PC-HQ-FINANCE
- PC-HQ-IT
- PC-HQ-ATM
- SERVER-BANK

### Branch 1

- PC-B1-MGMT
- PC-B1-EMP
- PC-B1-FINANCE
- PC-B1-ATM

### Branch 2

- PC-B2-MGMT
- PC-B2-EMP
- PC-B2-FINANCE
- PC-B2-ATM

### Branch 3

- PC-B3-MGMT
- PC-B3-EMP
- PC-B3-FINANCE
- PC-B3-ATM

---

# 4. Physical Cabling

## Router-to-Router

| From | Port | To | Port |
|---|---|---|---|
| R1-HQ | G0/1 | R2-BRANCH1 | G0/1 |
| R2-BRANCH1 | G0/2 | R3-BRANCH2 | G0/1 |
| R3-BRANCH2 | G0/2 | R4-BRANCH3 | G0/1 |

Use Copper Cross-Over or Packet Tracer's automatic connection tool.

## Router-to-Switch

| Router | Port | Switch | Port |
|---|---|---|---|
| R1-HQ | G0/0 | SW1-HQ | G0/1 |
| R2-BRANCH1 | G0/0 | SW2-BRANCH1 | G0/1 |
| R3-BRANCH2 | G0/0 | SW3-BRANCH2 | G0/1 |
| R4-BRANCH3 | G0/0 | SW4-BRANCH3 | G0/1 |

Use Copper Straight-Through.

---

# 5. Head Office Switch Connections

Connect:

| Device | SW1 Port | VLAN |
|---|---|---:|
| PC-HQ-ADMIN | Fa0/1 | 10 |
| PC-HQ-EMP | Fa0/2 | 20 |
| PC-HQ-FINANCE | Fa0/3 | 30 |
| PC-HQ-IT | Fa0/4 | 40 |
| PC-HQ-ATM | Fa0/5 | 50 |
| SERVER-BANK | Fa0/6 | 60 |

Router:

```text
R1-HQ G0/0 → SW1-HQ G0/1
```

---

# 6. Branch 1 Switch Connections

| Device | SW2 Port | VLAN |
|---|---|---:|
| PC-B1-MGMT | Fa0/1 | 10 |
| PC-B1-EMP | Fa0/2 | 20 |
| PC-B1-FINANCE | Fa0/3 | 30 |
| PC-B1-ATM | Fa0/4 | 50 |

Router:

```text
R2-BRANCH1 G0/0 → SW2-BRANCH1 G0/1
```

---

# 7. Branch 2 Switch Connections

| Device | SW3 Port | VLAN |
|---|---|---:|
| PC-B2-MGMT | Fa0/1 | 10 |
| PC-B2-EMP | Fa0/2 | 20 |
| PC-B2-FINANCE | Fa0/3 | 30 |
| PC-B2-ATM | Fa0/4 | 50 |

Router:

```text
R3-BRANCH2 G0/0 → SW3-BRANCH2 G0/1
```

---

# 8. Branch 3 Switch Connections

| Device | SW4 Port | VLAN |
|---|---|---:|
| PC-B3-MGMT | Fa0/1 | 10 |
| PC-B3-EMP | Fa0/2 | 20 |
| PC-B3-FINANCE | Fa0/3 | 30 |
| PC-B3-ATM | Fa0/4 | 50 |

Router:

```text
R4-BRANCH3 G0/0 → SW4-BRANCH3 G0/1
```

---

# 9. IP Addressing Plan

## Head Office

| VLAN | Name | Network | Gateway |
|---:|---|---|---|
| 10 | Management | 10.1.10.0/24 | 10.1.10.1 |
| 20 | Employees | 10.1.20.0/24 | 10.1.20.1 |
| 30 | Finance | 10.1.30.0/24 | 10.1.30.1 |
| 40 | IT | 10.1.40.0/24 | 10.1.40.1 |
| 50 | ATM | 10.1.50.0/24 | 10.1.50.1 |
| 60 | Servers | 10.1.60.0/24 | 10.1.60.1 |

## Branch 1

| VLAN | Name | Network | Gateway |
|---:|---|---|---|
| 10 | Management | 10.2.10.0/24 | 10.2.10.1 |
| 20 | Employees | 10.2.20.0/24 | 10.2.20.1 |
| 30 | Finance | 10.2.30.0/24 | 10.2.30.1 |
| 50 | ATM | 10.2.50.0/24 | 10.2.50.1 |

## Branch 2

| VLAN | Name | Network | Gateway |
|---:|---|---|---|
| 10 | Management | 10.3.10.0/24 | 10.3.10.1 |
| 20 | Employees | 10.3.20.0/24 | 10.3.20.1 |
| 30 | Finance | 10.3.30.0/24 | 10.3.30.1 |
| 50 | ATM | 10.3.50.0/24 | 10.3.50.1 |

## Branch 3

| VLAN | Name | Network | Gateway |
|---:|---|---|---|
| 10 | Management | 10.4.10.0/24 | 10.4.10.1 |
| 20 | Employees | 10.4.20.0/24 | 10.4.20.1 |
| 30 | Finance | 10.4.30.0/24 | 10.4.30.1 |
| 50 | ATM | 10.4.50.0/24 | 10.4.50.1 |

---

# 10. WAN Addressing

| Link | Device | Interface | IP |
|---|---|---|---|
| R1-R2 | R1 | G0/1 | 172.16.12.1/30 |
| R1-R2 | R2 | G0/1 | 172.16.12.2/30 |
| R2-R3 | R2 | G0/2 | 172.16.23.1/30 |
| R2-R3 | R3 | G0/1 | 172.16.23.2/30 |
| R3-R4 | R3 | G0/2 | 172.16.34.1/30 |
| R3-R4 | R4 | G0/1 | 172.16.34.2/30 |

---

# 11. Build Order

Follow this exact order:

```text
1. Place devices
2. Rename devices
3. Wire all devices
4. Configure switches
5. Configure router LAN interfaces
6. Configure router WAN interfaces
7. Configure OSPF
8. Configure DHCP
9. Configure server
10. Configure ACL
11. Configure PCs for DHCP
12. Test VLANs
13. Test DHCP
14. Test OSPF
15. Test inter-branch communication
16. Test server
17. Test ACL
18. Save the Packet Tracer file
```

Do not configure everything randomly.

---

# 12. Configure SW1-HQ

Open:

```text
SW1-HQ → CLI
```

Enter:

```cisco
enable
configure terminal

hostname SW1-HQ

vlan 10
 name MANAGEMENT
exit

vlan 20
 name EMPLOYEES
exit

vlan 30
 name FINANCE
exit

vlan 40
 name IT
exit

vlan 50
 name ATM
exit

vlan 60
 name SERVERS
exit

interface gigabitEthernet 0/1
 switchport mode trunk
exit

interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
exit

interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
exit

interface fastEthernet 0/3
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
exit

interface fastEthernet 0/4
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast
exit

interface fastEthernet 0/5
 switchport mode access
 switchport access vlan 50
 spanning-tree portfast
exit

interface fastEthernet 0/6
 switchport mode access
 switchport access vlan 60
 spanning-tree portfast
exit

end
write memory
```

Check:

```cisco
show vlan brief
show interfaces trunk
```

---

# 13. Configure SW2-BRANCH1

```cisco
enable
configure terminal

hostname SW2-BRANCH1

vlan 10
 name MANAGEMENT
exit

vlan 20
 name EMPLOYEES
exit

vlan 30
 name FINANCE
exit

vlan 50
 name ATM
exit

interface gigabitEthernet 0/1
 switchport mode trunk
exit

interface fastEthernet 0/1
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
exit

interface fastEthernet 0/2
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
exit

interface fastEthernet 0/3
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
exit

interface fastEthernet 0/4
 switchport mode access
 switchport access vlan 50
 spanning-tree portfast
exit

end
write memory
```

---

# 14. Configure SW3-BRANCH2

Use the same configuration as SW2, changing:

```text
hostname SW3-BRANCH2
```

Then:

```cisco
write memory
```

---

# 15. Configure SW4-BRANCH3

Use the same configuration as SW2, changing:

```text
hostname SW4-BRANCH3
```

Then:

```cisco
write memory
```

---

# 16. Configure R1-HQ

Open:

```text
R1-HQ → CLI
```

Enter:

```cisco
enable
configure terminal

hostname R1-HQ
no ip domain-lookup

interface gigabitEthernet 0/0
 no shutdown
exit

interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 10.1.10.1 255.255.255.0
exit

interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 10.1.20.1 255.255.255.0
exit

interface gigabitEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 10.1.30.1 255.255.255.0
exit

interface gigabitEthernet 0/0.40
 encapsulation dot1Q 40
 ip address 10.1.40.1 255.255.255.0
exit

interface gigabitEthernet 0/0.50
 encapsulation dot1Q 50
 ip address 10.1.50.1 255.255.255.0
exit

interface gigabitEthernet 0/0.60
 encapsulation dot1Q 60
 ip address 10.1.60.1 255.255.255.0
exit

interface gigabitEthernet 0/1
 ip address 172.16.12.1 255.255.255.252
 no shutdown
exit
```

---

# 17. Configure OSPF on R1

```cisco
router ospf 1
 router-id 1.1.1.1

 network 10.1.10.0 0.0.0.255 area 0
 network 10.1.20.0 0.0.0.255 area 0
 network 10.1.30.0 0.0.0.255 area 0
 network 10.1.40.0 0.0.0.255 area 0
 network 10.1.50.0 0.0.0.255 area 0
 network 10.1.60.0 0.0.0.255 area 0
 network 172.16.12.0 0.0.0.3 area 0

end
write memory
```

---

# 18. Configure R1 DHCP

```cisco
enable
configure terminal

ip dhcp excluded-address 10.1.10.1 10.1.10.10

ip dhcp pool HQ-MANAGEMENT
 network 10.1.10.0 255.255.255.0
 default-router 10.1.10.1
 dns-server 10.1.60.10
exit

ip dhcp excluded-address 10.1.20.1 10.1.20.10

ip dhcp pool HQ-EMPLOYEES
 network 10.1.20.0 255.255.255.0
 default-router 10.1.20.1
 dns-server 10.1.60.10
exit

ip dhcp excluded-address 10.1.30.1 10.1.30.10

ip dhcp pool HQ-FINANCE
 network 10.1.30.0 255.255.255.0
 default-router 10.1.30.1
 dns-server 10.1.60.10
exit

ip dhcp excluded-address 10.1.40.1 10.1.40.10

ip dhcp pool HQ-IT
 network 10.1.40.0 255.255.255.0
 default-router 10.1.40.1
 dns-server 10.1.60.10
exit

ip dhcp excluded-address 10.1.50.1 10.1.50.10

ip dhcp pool HQ-ATM
 network 10.1.50.0 255.255.255.0
 default-router 10.1.50.1
 dns-server 10.1.60.10
exit

end
write memory
```

---

# 19. Configure R2-BRANCH1

```cisco
enable
configure terminal

hostname R2-BRANCH1
no ip domain-lookup

interface gigabitEthernet 0/0
 no shutdown
exit

interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 10.2.10.1 255.255.255.0
exit

interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 10.2.20.1 255.255.255.0
exit

interface gigabitEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 10.2.30.1 255.255.255.0
exit

interface gigabitEthernet 0/0.50
 encapsulation dot1Q 50
 ip address 10.2.50.1 255.255.255.0
exit

interface gigabitEthernet 0/1
 ip address 172.16.12.2 255.255.255.252
 no shutdown
exit

interface gigabitEthernet 0/2
 ip address 172.16.23.1 255.255.255.252
 no shutdown
exit

router ospf 1
 router-id 2.2.2.2

 network 10.2.10.0 0.0.0.255 area 0
 network 10.2.20.0 0.0.0.255 area 0
 network 10.2.30.0 0.0.0.255 area 0
 network 10.2.50.0 0.0.0.255 area 0
 network 172.16.12.0 0.0.0.3 area 0
 network 172.16.23.0 0.0.0.3 area 0

end
write memory
```

---

# 20. Configure R2 DHCP

```cisco
enable
configure terminal

ip dhcp excluded-address 10.2.10.1 10.2.10.10

ip dhcp pool B1-MANAGEMENT
 network 10.2.10.0 255.255.255.0
 default-router 10.2.10.1
 dns-server 10.1.60.10
exit

ip dhcp excluded-address 10.2.20.1 10.2.20.10

ip dhcp pool B1-EMPLOYEES
 network 10.2.20.0 255.255.255.0
 default-router 10.2.20.1
 dns-server 10.1.60.10
exit

ip dhcp excluded-address 10.2.30.1 10.2.30.10

ip dhcp pool B1-FINANCE
 network 10.2.30.0 255.255.255.0
 default-router 10.2.30.1
 dns-server 10.1.60.10
exit

ip dhcp excluded-address 10.2.50.1 10.2.50.10

ip dhcp pool B1-ATM
 network 10.2.50.0 255.255.255.0
 default-router 10.2.50.1
 dns-server 10.1.60.10
exit

end
write memory
```

---

# 21. Configure R3-BRANCH2

```cisco
enable
configure terminal

hostname R3-BRANCH2
no ip domain-lookup

interface gigabitEthernet 0/0
 no shutdown
exit

interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 10.3.10.1 255.255.255.0
exit

interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 10.3.20.1 255.255.255.0
exit

interface gigabitEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 10.3.30.1 255.255.255.0
exit

interface gigabitEthernet 0/0.50
 encapsulation dot1Q 50
 ip address 10.3.50.1 255.255.255.0
exit

interface gigabitEthernet 0/1
 ip address 172.16.23.2 255.255.255.252
 no shutdown
exit

interface gigabitEthernet 0/2
 ip address 172.16.34.1 255.255.255.252
 no shutdown
exit

router ospf 1
 router-id 3.3.3.3

 network 10.3.10.0 0.0.0.255 area 0
 network 10.3.20.0 0.0.0.255 area 0
 network 10.3.30.0 0.0.0.255 area 0
 network 10.3.50.0 0.0.0.255 area 0
 network 172.16.23.0 0.0.0.3 area 0
 network 172.16.34.0 0.0.0.3 area 0

end
write memory
```

---

# 22. Configure R3 DHCP

```cisco
enable
configure terminal

ip dhcp excluded-address 10.3.10.1 10.3.10.10

ip dhcp pool B2-MANAGEMENT
 network 10.3.10.0 255.255.255.0
 default-router 10.3.10.1
 dns-server 10.1.60.10
exit

ip dhcp excluded-address 10.3.20.1 10.3.20.10

ip dhcp pool B2-EMPLOYEES
 network 10.3.20.0 255.255.255.0
 default-router 10.3.20.1
 dns-server 10.1.60.10
exit

ip dhcp excluded-address 10.3.30.1 10.3.30.10

ip dhcp pool B2-FINANCE
 network 10.3.30.0 255.255.255.0
 default-router 10.3.30.1
 dns-server 10.1.60.10
exit

ip dhcp excluded-address 10.3.50.1 10.3.50.10

ip dhcp pool B2-ATM
 network 10.3.50.0 255.255.255.0
 default-router 10.3.50.1
 dns-server 10.1.60.10
exit

end
write memory
```

---

# 23. Configure R4-BRANCH3

```cisco
enable
configure terminal

hostname R4-BRANCH3
no ip domain-lookup

interface gigabitEthernet 0/0
 no shutdown
exit

interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 10.4.10.1 255.255.255.0
exit

interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 10.4.20.1 255.255.255.0
exit

interface gigabitEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 10.4.30.1 255.255.255.0
exit

interface gigabitEthernet 0/0.50
 encapsulation dot1Q 50
 ip address 10.4.50.1 255.255.255.0
exit

interface gigabitEthernet 0/1
 ip address 172.16.34.2 255.255.255.252
 no shutdown
exit

router ospf 1
 router-id 4.4.4.4

 network 10.4.10.0 0.0.0.255 area 0
 network 10.4.20.0 0.0.0.255 area 0
 network 10.4.30.0 0.0.0.255 area 0
 network 10.4.50.0 0.0.0.255 area 0
 network 172.16.34.0 0.0.0.3 area 0

end
write memory
```

---

# 24. Configure R4 DHCP

```cisco
enable
configure terminal

ip dhcp excluded-address 10.4.10.1 10.4.10.10

ip dhcp pool B3-MANAGEMENT
 network 10.4.10.0 255.255.255.0
 default-router 10.4.10.1
 dns-server 10.1.60.10
exit

ip dhcp excluded-address 10.4.20.1 10.4.20.10

ip dhcp pool B3-EMPLOYEES
 network 10.4.20.0 255.255.255.0
 default-router 10.4.20.1
 dns-server 10.1.60.10
exit

ip dhcp excluded-address 10.4.30.1 10.4.30.10

ip dhcp pool B3-FINANCE
 network 10.4.30.0 255.255.255.0
 default-router 10.4.30.1
 dns-server 10.1.60.10
exit

ip dhcp excluded-address 10.4.50.1 10.4.50.10

ip dhcp pool B3-ATM
 network 10.4.50.0 255.255.255.0
 default-router 10.4.50.1
 dns-server 10.1.60.10
exit

end
write memory
```

---

# 25. Configure the Banking Server

Select:

```text
SERVER-BANK
→ Desktop
→ IP Configuration
```

Set:

```text
IP Address:      10.1.60.10
Subnet Mask:     255.255.255.0
Default Gateway: 10.1.60.1
DNS Server:      10.1.60.10
```

Do NOT select DHCP.

---

# 26. Enable DNS

Go:

```text
SERVER-BANK
→ Services
→ DNS
```

Set:

```text
DNS: ON
```

Add:

```text
Name: bank.local
Address: 10.1.60.10
```

Add another record:

```text
Name: www.bank.local
Address: 10.1.60.10
```

---

# 27. Enable Web Server

Go:

```text
SERVER-BANK
→ Services
→ HTTP
```

Set:

```text
HTTP: ON
HTTPS: ON
```

Edit the index page:

```text
SECURE BANKING NETWORK

Welcome to National Banking Corporation

Head Office
Branch 1
Branch 2
Branch 3

Services:
- Online Banking
- ATM Services
- Account Management
- Customer Support
```

---

# 28. Configure All PCs for DHCP

For every PC:

```text
PC
→ Desktop
→ IP Configuration
→ DHCP
```

Wait a few seconds.

The PC should receive an address.

Example:

```text
IP Address:      10.1.20.11
Subnet Mask:     255.255.255.0
Default Gateway: 10.1.20.1
DNS:             10.1.60.10
```

The exact host number may differ.

---

# 29. Verify DHCP

On R1:

```cisco
show ip dhcp binding
```

On R2:

```cisco
show ip dhcp binding
```

On R3:

```cisco
show ip dhcp binding
```

On R4:

```cisco
show ip dhcp binding
```

If bindings appear, DHCP is working.

---

# 30. Configure Banking Security ACL

## HQ

Employees must not access Finance.

On R1:

```cisco
enable
configure terminal

access-list 110 deny ip 10.1.20.0 0.0.0.255 10.1.30.0 0.0.0.255
access-list 110 permit ip any any

interface gigabitEthernet 0/0.20
 ip access-group 110 in
exit

end
write memory
```

## Branch 1

```cisco
enable
configure terminal

access-list 120 deny ip 10.2.20.0 0.0.0.255 10.2.30.0 0.0.0.255
access-list 120 permit ip any any

interface gigabitEthernet 0/0.20
 ip access-group 120 in
exit

end
write memory
```

## Branch 2

```cisco
enable
configure terminal

access-list 130 deny ip 10.3.20.0 0.0.0.255 10.3.30.0 0.0.0.255
access-list 130 permit ip any any

interface gigabitEthernet 0/0.20
 ip access-group 130 in
exit

end
write memory
```

## Branch 3

```cisco
enable
configure terminal

access-list 140 deny ip 10.4.20.0 0.0.0.255 10.4.30.0 0.0.0.255
access-list 140 permit ip any any

interface gigabitEthernet 0/0.20
 ip access-group 140 in
exit

end
write memory
```

---

# 31. First Verification — Interfaces

Run on every router:

```cisco
show ip interface brief
```

Everything important should show:

```text
up    up
```

Check:

```text
R1:
G0/0
G0/0.10
G0/0.20
G0/0.30
G0/0.40
G0/0.50
G0/0.60
G0/1

R2:
G0/0
G0/0.10
G0/0.20
G0/0.30
G0/0.50
G0/1
G0/2

R3:
G0/0
G0/0.10
G0/0.20
G0/0.30
G0/0.50
G0/1
G0/2

R4:
G0/0
G0/0.10
G0/0.20
G0/0.30
G0/0.50
G0/1
```

---

# 32. Second Verification — VLAN

On SW1:

```cisco
show vlan brief
```

Expected:

```text
10  MANAGEMENT
20  EMPLOYEES
30  FINANCE
40  IT
50  ATM
60  SERVERS
```

On SW2/SW3/SW4:

```text
10  MANAGEMENT
20  EMPLOYEES
30  FINANCE
50  ATM
```

---

# 33. Third Verification — Trunk

On every switch:

```cisco
show interfaces trunk
```

The router-facing:

```text
G0/1
```

should be trunking.

---

# 34. Fourth Verification — OSPF

On R1:

```cisco
show ip ospf neighbor
```

Expected neighbor:

```text
R2
```

On R2:

```cisco
show ip ospf neighbor
```

Expected:

```text
R1
R3
```

On R3:

```cisco
show ip ospf neighbor
```

Expected:

```text
R2
R4
```

On R4:

```cisco
show ip ospf neighbor
```

Expected:

```text
R3
```

---

# 35. Fifth Verification — Routing Table

Run:

```cisco
show ip route
```

Look for routes beginning with:

```text
O
```

Example:

```text
O 10.2.20.0/24
O 10.3.20.0/24
O 10.4.20.0/24
```

The `O` means the route was learned through OSPF.

---

# 36. Sixth Verification — Router Ping

From R1:

```cisco
ping 172.16.12.2
```

Expected:

```text
Success
```

From R2:

```cisco
ping 172.16.23.2
```

Expected:

```text
Success
```

From R3:

```cisco
ping 172.16.34.2
```

Expected:

```text
Success
```

---

# 37. Seventh Verification — Inter-Branch Ping

From an HQ PC, ping a Branch 1 PC.

Example:

```text
ping 10.2.20.11
```

Then Branch 2:

```text
ping 10.3.20.11
```

Then Branch 3:

```text
ping 10.4.20.11
```

All should succeed.

---

# 38. Eighth Verification — Banking Server

From any PC:

```text
ping 10.1.60.10
```

Expected:

```text
Reply from 10.1.60.10
```

Then open:

```text
Desktop
→ Web Browser
```

Enter:

```text
http://10.1.60.10
```

The banking webpage should load.

Then test:

```text
http://bank.local
```

If DNS is working, this should also load the banking page.

---

# 39. Ninth Verification — ACL

From an HQ Employee PC:

```text
ping 10.1.30.x
```

where `10.1.30.x` is the Finance PC.

Expected:

```text
Request timed out
```

This is intentional.

Then:

```text
ping 10.1.60.10
```

Expected:

```text
Reply
```

Therefore:

```text
Employee → Finance       BLOCKED
Employee → Bank Server   ALLOWED
```

---

# 40. ACL Verification

On R1:

```cisco
show access-lists
```

You should see packet counters.

For example:

```text
Extended IP access list 110
deny ip 10.1.20.0 ...
permit ip any any
```

After attempting the blocked ping, the deny counter should increase.

---

# 41. Complete Final Testing Checklist

## Physical

- [ ] Four routers placed
- [ ] Four switches placed
- [ ] All PCs placed
- [ ] Banking server placed
- [ ] Router-to-router cables connected
- [ ] Router-to-switch cables connected
- [ ] PC-to-switch cables connected
- [ ] Links become green

## Switches

- [ ] SW1 VLANs configured
- [ ] SW2 VLANs configured
- [ ] SW3 VLANs configured
- [ ] SW4 VLANs configured
- [ ] Access ports assigned correctly
- [ ] Router ports configured as trunks

## Routers

- [ ] R1 LAN configured
- [ ] R2 LAN configured
- [ ] R3 LAN configured
- [ ] R4 LAN configured
- [ ] WAN IPs configured
- [ ] All interfaces `up/up`

## DHCP

- [ ] HQ Management receives IP
- [ ] HQ Employees receives IP
- [ ] HQ Finance receives IP
- [ ] HQ IT receives IP
- [ ] HQ ATM receives IP
- [ ] Branch 1 receives IP
- [ ] Branch 2 receives IP
- [ ] Branch 3 receives IP

## Routing

- [ ] R1 sees R2
- [ ] R2 sees R1 and R3
- [ ] R3 sees R2 and R4
- [ ] R4 sees R3
- [ ] OSPF routes appear
- [ ] Branches can communicate

## Server

- [ ] Static IP configured
- [ ] Gateway configured
- [ ] DNS enabled
- [ ] HTTP enabled
- [ ] bank.local configured
- [ ] Banking webpage loads

## Security

- [ ] HQ Employee → Finance blocked
- [ ] Branch 1 Employee → Finance blocked
- [ ] Branch 2 Employee → Finance blocked
- [ ] Branch 3 Employee → Finance blocked
- [ ] Employee → Banking Server allowed
- [ ] ACL counters increase

---

# 42. Important Troubleshooting Commands

If something does not work, don't immediately reconfigure everything.

Use these commands.

### Router interfaces

```cisco
show ip interface brief
```

### Router configuration

```cisco
show running-config
```

### Routing

```cisco
show ip route
```

### OSPF

```cisco
show ip ospf neighbor
```

### DHCP

```cisco
show ip dhcp binding
show ip dhcp pool
```

### Switch VLAN

```cisco
show vlan brief
```

### Switch trunk

```cisco
show interfaces trunk
```

### ACL

```cisco
show access-lists
```

---

# 43. Common Problems

## PC receives `169.254.x.x`

DHCP failed.

Check:

```text
PC → correct switch port
Switch → correct VLAN
Switch → trunk
Router → correct subinterface
Router → DHCP pool
```

---

## Router interface says `administratively down`

Run:

```cisco
configure terminal
interface gigabitEthernet 0/0
no shutdown
```

---

## OSPF neighbor doesn't appear

Check both sides:

```cisco
show ip interface brief
```

Then:

```cisco
show ip ospf neighbor
```

Verify the WAN IP addresses and `/30` networks.

---

## VLAN PC cannot reach gateway

Check:

```cisco
show vlan brief
show interfaces trunk
```

Then verify that the router has the corresponding subinterface:

```text
G0/0.10
G0/0.20
G0/30
...
```

and that the VLAN number matches:

```cisco
encapsulation dot1Q 10
```

---

## Branches cannot communicate

Check:

```cisco
show ip ospf neighbor
show ip route
```

If OSPF routes aren't present, check the OSPF `network` statements.

---

# 44. Final Configuration Save

After everything works, run on every router:

```cisco
write memory
```

or:

```cisco
copy running-config startup-config
```

Do the same for all switches.

Then save your Packet Tracer file:

```text
File
→ Save As
```

Suggested filename:

```text
Secure_Multi_Branch_Banking_Network.pkt
```

---

# 45. What You Can Claim in the Project

Your project implements:

```text
✓ Enterprise Banking Network
✓ Multi-Branch Architecture
✓ VLAN Segmentation
✓ Inter-VLAN Routing
✓ Router-on-a-Stick
✓ DHCP
✓ OSPF
✓ WAN Connectivity
✓ Central Banking Server
✓ DNS
✓ HTTP
✓ ACL Security
✓ Finance Department Protection
✓ ATM Network
✓ Network Monitoring/Verification
```

---

# 46. Recommended Demonstration Flow

During your lab presentation, demonstrate the project in this order:

### Step 1 — Topology

Explain:

> "Our system consists of a Head Office and three banking branches connected through routed WAN links."

### Step 2 — VLAN

Show:

```cisco
show vlan brief
```

Explain that departments are separated logically.

### Step 3 — DHCP

Show:

```cisco
show ip dhcp binding
```

Explain that clients automatically receive IP configuration.

### Step 4 — OSPF

Show:

```cisco
show ip ospf neighbor
```

Explain that OSPF dynamically exchanges routes between branches.

### Step 5 — Routing

Show:

```cisco
show ip route
```

Point out the `O` routes.

### Step 6 — Branch Communication

Ping:

```text
HQ → Branch 1
HQ → Branch 2
HQ → Branch 3
```

### Step 7 — Banking Server

Open:

```text
http://bank.local
```

Show the banking webpage.

### Step 8 — Security

Show:

```text
Employee → Finance
```

and demonstrate that it is blocked.

Then:

```text
Employee → Banking Server
```

and demonstrate that it is allowed.

---

# 47. Final Project Statement

**Secure Multi-Branch Banking Network Infrastructure** is an enterprise networking simulation designed in Cisco Packet Tracer to provide secure communication between a central banking headquarters and multiple branches. The system uses VLANs for departmental segmentation, router-on-a-stick for inter-VLAN communication, DHCP for automated IP assignment, OSPF for dynamic routing, and ACLs to protect sensitive financial networks. A centralized banking server provides DNS and HTTP services for internal banking applications.

---

# 48. One-Page Quick Reference

```text
PROJECT:
Secure Multi-Branch Banking Network Infrastructure

ROUTERS:
R1-HQ
R2-BRANCH1
R3-BRANCH2
R4-BRANCH3

SWITCHES:
SW1-HQ
SW2-BRANCH1
SW3-BRANCH2
SW4-BRANCH3

WAN:
R1-R2 = 172.16.12.0/30
R2-R3 = 172.16.23.0/30
R3-R4 = 172.16.34.0/30

HQ:
10.1.x.x

BRANCH 1:
10.2.x.x

BRANCH 2:
10.3.x.x

BRANCH 3:
10.4.x.x

VLAN:
10 = Management
20 = Employees
30 = Finance
40 = IT
50 = ATM
60 = Servers

ROUTING:
OSPF Area 0

DHCP:
Configured on each branch router

SERVER:
10.1.60.10

DNS:
bank.local

WEB:
http://bank.local

SECURITY:
Employees → Finance = BLOCKED

KEY COMMANDS:
show ip interface brief
show vlan brief
show interfaces trunk
show ip ospf neighbor
show ip route
show ip dhcp binding
show access-lists
```

**Build principle:** Physical topology → Switch VLANs → Router subinterfaces → WAN → OSPF → DHCP → Server → ACL → Testing. Follow that sequence and troubleshoot at the layer where the failure occurs.