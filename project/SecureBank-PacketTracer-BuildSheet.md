# Secure Multi-Branch Banking Network — Packet Tracer Build Sheet

Topology: 2 branches (HQ + Remote), each with Router-on-a-Stick + Core/Access switches, 
connected over a WAN serial link running OSPF. Covers: VLANs, Trunking, Router-on-a-Stick, 
DHCP, OSPF, WAN, DNS, HTTP, ACLs, port security, SSH hardening.

---

## 1. Topology Overview

```
                         WAN (Serial, OSPF Area 0)
   [R-HQ]==========Serial0/0/0 <-> Serial0/0/0==========[R-BR2]
     | Gig0/0 (trunk)                                Gig0/0 (trunk) |
 [SW-CORE-A]                                                  [SW-CORE-B]
   |   |   |                                                     |    |
 (trunk)(trunk)(access: Server)                              (trunk)(trunk)
   |       |                                                       |
[SW-ACC1-A]  [SW-ACC2-A]                                     [SW-ACC1-B]
 (Mgmt+Staff)  (Guest)                                     (Mgmt+Staff+Guest)
   |  |  |       |  |                                          |  |  |
 Mgmt Staff PCs  Guest PCs                                 Mgmt Staff Guest PC
```

Devices needed in Packet Tracer:
- 2x Router **1941** or **4321** (needs 1x GigabitEthernet + 1x Serial WIC — use `NM-1FE-TX`/onboard Gig + `HWIC-2T` module for serial)
- 4x Switch **2960**
- 1x Server-PT (DNS + HTTP)
- 9x PC-PT

---

## 2. VLAN Plan (same numbering both branches)

| VLAN ID | Name    | Purpose                | Branch A Subnet   | Branch B Subnet   |
|---------|---------|-------------------------|--------------------|--------------------|
| 10      | MGMT    | Bank admin/IT staff     | 10.10.10.0/24      | 10.20.10.0/24      |
| 20      | STAFF   | Tellers/back office     | 10.10.20.0/24      | 10.20.20.0/24      |
| 30      | SERVERS | DNS/HTTP servers        | 10.10.30.0/24      | 10.20.30.0/24 (reserved) |
| 40      | GUEST   | Customer WiFi/lobby     | 10.10.40.0/24      | 10.20.40.0/24      |
| 99      | NATIVE  | Unused native (security)| 10.10.99.0/24 (no hosts) | 10.20.99.0/24 (no hosts) |

WAN link: **172.16.1.0/30** — R-HQ Se0/0/0 = `.1` (DCE, clock rate), R-BR2 Se0/0/0 = `.2` (DTE)

Gateway for every VLAN = subnet's `.1` (the router sub-interface).

---

## 3. Device & Port-to-Port Cabling Table

### Branch A (HQ)

| From             | Port      | To                | Port      | Cable Type       |
|-------------------|-----------|-------------------|-----------|------------------|
| R-HQ              | Gig0/0    | SW-CORE-A         | Gig0/1    | Copper Straight  |
| R-HQ              | Se0/0/0   | R-BR2             | Se0/0/0   | Serial DCE/DTE   |
| SW-CORE-A         | Gig0/2    | SW-ACC1-A         | Gig0/1    | Copper Straight  |
| SW-CORE-A         | Gig0/3    | SW-ACC2-A         | Gig0/1    | Copper Straight  |
| SW-CORE-A         | Fa0/1     | Server-DNS-HTTP-A | Fa0       | Copper Straight  |
| SW-ACC1-A         | Fa0/2     | PC-Mgmt1-A        | Fa0       | Copper Straight  |
| SW-ACC1-A         | Fa0/3     | PC-Staff1-A       | Fa0       | Copper Straight  |
| SW-ACC1-A         | Fa0/4     | PC-Staff2-A       | Fa0       | Copper Straight  |
| SW-ACC2-A         | Fa0/2     | PC-Guest1-A       | Fa0       | Copper Straight  |
| SW-ACC2-A         | Fa0/3     | PC-Guest2-A       | Fa0       | Copper Straight  |

### Branch B (Remote)

| From             | Port      | To                | Port      | Cable Type       |
|-------------------|-----------|-------------------|-----------|------------------|
| R-BR2             | Gig0/0    | SW-CORE-B         | Gig0/1    | Copper Straight  |
| SW-CORE-B         | Gig0/2    | SW-ACC1-B         | Gig0/1    | Copper Straight  |
| SW-ACC1-B         | Fa0/2     | PC-Mgmt1-B        | Fa0       | Copper Straight  |
| SW-ACC1-B         | Fa0/3     | PC-Staff1-B       | Fa0       | Copper Straight  |
| SW-ACC1-B         | Fa0/4     | PC-Guest1-B       | Fa0       | Copper Straight  |

> In Packet Tracer, all switch-switch, switch-router, and switch-PC copper links auto-detect — straight-through works everywhere. Serial cable: drag DCE end onto R-HQ.

---

## 4. PC / Server IP Table

| Device            | VLAN | IP Assignment | IP Address     | Subnet Mask     | Gateway      | DNS Server   |
|-------------------|------|----------------|-----------------|-------------------|---------------|---------------|
| PC-Mgmt1-A        | 10   | Static         | 10.10.10.10     | 255.255.255.0     | 10.10.10.1    | 10.10.30.10   |
| PC-Staff1-A       | 20   | DHCP            | (from pool)     | —                  | —             | —             |
| PC-Staff2-A       | 20   | DHCP            | (from pool)     | —                  | —             | —             |
| Server-DNS-HTTP-A | 30   | Static          | 10.10.30.10     | 255.255.255.0     | 10.10.30.1    | 10.10.30.10   |
| PC-Guest1-A       | 40   | DHCP            | (from pool)     | —                  | —             | —             |
| PC-Guest2-A       | 40   | DHCP            | (from pool)     | —                  | —             | —             |
| PC-Mgmt1-B        | 10   | Static          | 10.20.10.10     | 255.255.255.0     | 10.20.10.1    | 10.10.30.10   |
| PC-Staff1-B       | 20   | DHCP            | (from pool)     | —                  | —             | —             |
| PC-Guest1-B       | 40   | DHCP            | (from pool)     | —                  | —             | —             |

---

## 5. Switch CLI

Apply to **every** switch first (adjust hostname):

```
enable
configure terminal
no ip domain-lookup
service password-encryption
enable secret Bank@dmin2026
username admin privilege 15 secret Admin@2026
banner motd # AUTHORIZED ACCESS ONLY - SecureBank Network - Violators Prosecuted #
line console 0
 password Console@2026
 login
 logging synchronous
line vty 0 15
 transport input ssh
 login local
ip domain-name securebank.local
crypto key generate rsa
 1024
ip ssh version 2
```

### SW-CORE-A
```
hostname SW-CORE-A
vlan 10
 name MGMT
vlan 20
 name STAFF
vlan 30
 name SERVERS
vlan 40
 name GUEST
vlan 99
 name NATIVE_UNUSED
exit

interface range GigabitEthernet0/1-3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,99
exit

interface FastEthernet0/1
 switchport mode access
 switchport access vlan 30
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
exit

interface vlan 10
 ip address 10.10.10.2 255.255.255.0
 no shutdown
exit
ip default-gateway 10.10.10.1
interface vlan1
 shutdown
end
write memory
```

### SW-ACC1-A (Mgmt + Staff access)
```
hostname SW-ACC1-A
vlan 10
 name MGMT
vlan 20
 name STAFF
vlan 99
 name NATIVE_UNUSED
exit

interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
exit

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 spanning-tree portfast
 spanning-tree bpduguard enable
exit

interface range FastEthernet0/3-4
 switchport mode access
 switchport access vlan 20
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 spanning-tree portfast
 spanning-tree bpduguard enable
exit

interface vlan 10
 ip address 10.10.10.3 255.255.255.0
 no shutdown
exit
ip default-gateway 10.10.10.1
end
write memory
```

### SW-ACC2-A (Guest access)
```
hostname SW-ACC2-A
vlan 40
 name GUEST
vlan 99
 name NATIVE_UNUSED
exit

interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 40,99
exit

interface range FastEthernet0/2-3
 switchport mode access
 switchport access vlan 40
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 spanning-tree portfast
 spanning-tree bpduguard enable
exit

interface vlan 10
 ip address 10.10.10.4 255.255.255.0
 no shutdown
exit
ip default-gateway 10.10.10.1
end
write memory
```

### SW-CORE-B
```
hostname SW-CORE-B
vlan 10
 name MGMT
vlan 20
 name STAFF
vlan 30
 name SERVERS
vlan 40
 name GUEST
vlan 99
 name NATIVE_UNUSED
exit

interface range GigabitEthernet0/1-2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,40,99
exit

interface vlan 10
 ip address 10.20.10.2 255.255.255.0
 no shutdown
exit
ip default-gateway 10.20.10.1
end
write memory
```

### SW-ACC1-B (Mgmt + Staff + Guest access)
```
hostname SW-ACC1-B
vlan 10
 name MGMT
vlan 20
 name STAFF
vlan 40
 name GUEST
vlan 99
 name NATIVE_UNUSED
exit

interface GigabitEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,40,99
exit

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown
 spanning-tree portfast
 spanning-tree bpduguard enable
exit

interface FastEthernet0/3
 switchport mode access
 switchport access vlan 20
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 spanning-tree portfast
 spanning-tree bpduguard enable
exit

interface FastEthernet0/4
 switchport mode access
 switchport access vlan 40
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
 spanning-tree portfast
 spanning-tree bpduguard enable
exit

interface vlan 10
 ip address 10.20.10.3 255.255.255.0
 no shutdown
exit
ip default-gateway 10.20.10.1
end
write memory
```

---

## 6. Router CLI

### R-HQ (full config)

```
enable
configure terminal
hostname R-HQ
no ip domain-lookup
ip domain-name securebank.local
service password-encryption
enable secret Bank@dmin2026
username admin privilege 15 secret Admin@2026
banner motd # AUTHORIZED ACCESS ONLY - SecureBank HQ Router #

crypto key generate rsa
 1024
ip ssh version 2

line console 0
 password Console@2026
 login
 logging synchronous
line vty 0 4
 transport input ssh
 login local
 access-class MGMT-ONLY in

! ---- Router-on-a-Stick sub-interfaces ----
interface GigabitEthernet0/0
 no shutdown
exit

interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.10.10.1 255.255.255.0
exit

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 10.10.20.1 255.255.255.0
exit

interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.10.30.1 255.255.255.0
exit

interface GigabitEthernet0/0.40
 encapsulation dot1Q 40
 ip address 10.10.40.1 255.255.255.0
 ip access-group GUEST-RESTRICT-A in
exit

! ---- WAN ----
interface Serial0/0/0
 ip address 172.16.1.1 255.255.255.252
 clock rate 64000
 no shutdown
exit

! ---- OSPF ----
router ospf 1
 router-id 1.1.1.1
 network 10.10.10.0 0.0.0.255 area 0
 network 10.10.20.0 0.0.0.255 area 0
 network 10.10.30.0 0.0.0.255 area 0
 network 10.10.40.0 0.0.0.255 area 0
 network 172.16.1.0 0.0.0.3 area 0
 passive-interface GigabitEthernet0/0.10
 passive-interface GigabitEthernet0/0.20
 passive-interface GigabitEthernet0/0.30
 passive-interface GigabitEthernet0/0.40
exit

! ---- DHCP ----
ip dhcp excluded-address 10.10.20.1 10.10.20.10
ip dhcp excluded-address 10.10.40.1 10.10.40.10

ip dhcp pool STAFF-A
 network 10.10.20.0 255.255.255.0
 default-router 10.10.20.1
 dns-server 10.10.30.10
exit

ip dhcp pool GUEST-A
 network 10.10.40.0 255.255.255.0
 default-router 10.10.40.1
 dns-server 10.10.30.10
exit

! ---- Management-only ACL (SSH/VTY) ----
ip access-list standard MGMT-ONLY
 permit 10.10.10.0 0.0.0.255
 permit 10.20.10.0 0.0.0.255
 deny any

! ---- Guest isolation ACL ----
ip access-list extended GUEST-RESTRICT-A
 permit udp any host 10.10.30.10 eq 53
 permit tcp any host 10.10.30.10 eq 80
 deny ip any 10.10.10.0 0.0.0.255
 deny ip any 10.10.20.0 0.0.0.255
 deny ip any 10.20.10.0 0.0.0.255
 deny ip any 10.20.20.0 0.0.0.255
 permit ip any any

end
write memory
```

### R-BR2 (full config)

```
enable
configure terminal
hostname R-BR2
no ip domain-lookup
ip domain-name securebank.local
service password-encryption
enable secret Bank@dmin2026
username admin privilege 15 secret Admin@2026
banner motd # AUTHORIZED ACCESS ONLY - SecureBank Branch Router #

crypto key generate rsa
 1024
ip ssh version 2

line console 0
 password Console@2026
 login
 logging synchronous
line vty 0 4
 transport input ssh
 login local
 access-class MGMT-ONLY in

interface GigabitEthernet0/0
 no shutdown
exit

interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.20.10.1 255.255.255.0
exit

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 10.20.20.1 255.255.255.0
exit

interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.20.30.1 255.255.255.0
exit

interface GigabitEthernet0/0.40
 encapsulation dot1Q 40
 ip address 10.20.40.1 255.255.255.0
 ip access-group GUEST-RESTRICT-B in
exit

interface Serial0/0/0
 ip address 172.16.1.2 255.255.255.252
 no shutdown
exit

router ospf 1
 router-id 2.2.2.2
 network 10.20.10.0 0.0.0.255 area 0
 network 10.20.20.0 0.0.0.255 area 0
 network 10.20.30.0 0.0.0.255 area 0
 network 10.20.40.0 0.0.0.255 area 0
 network 172.16.1.0 0.0.0.3 area 0
 passive-interface GigabitEthernet0/0.10
 passive-interface GigabitEthernet0/0.20
 passive-interface GigabitEthernet0/0.30
 passive-interface GigabitEthernet0/0.40
exit

ip dhcp excluded-address 10.20.20.1 10.20.20.10
ip dhcp excluded-address 10.20.40.1 10.20.40.10

ip dhcp pool STAFF-B
 network 10.20.20.0 255.255.255.0
 default-router 10.20.20.1
 dns-server 10.10.30.10
exit

ip dhcp pool GUEST-B
 network 10.20.40.0 255.255.255.0
 default-router 10.20.40.1
 dns-server 10.10.30.10
exit

ip access-list standard MGMT-ONLY
 permit 10.10.10.0 0.0.0.255
 permit 10.20.10.0 0.0.0.255
 deny any

ip access-list extended GUEST-RESTRICT-B
 permit udp any host 10.10.30.10 eq 53
 permit tcp any host 10.10.30.10 eq 80
 deny ip any 10.10.10.0 0.0.0.255
 deny ip any 10.10.20.0 0.0.0.255
 deny ip any 10.20.10.0 0.0.0.255
 deny ip any 10.20.20.0 0.0.0.255
 permit ip any any

end
write memory
```

> `Serial0/0/0` naming can vary by router model — if you're using a 1941 with a serial HWIC it may show as `Serial0/0/0`; on 4321 with NIM it could be `Serial0/2/0`. Check `show ip interface brief` and adjust interface names accordingly.

---

## 7. Server Setup (Server-DNS-HTTP-A)

**Desktop tab → IP Configuration:**
- Static IP: `10.10.30.10`
- Subnet Mask: `255.255.255.0`
- Gateway: `10.10.30.1`
- DNS Server: `10.10.30.10`

**Services tab → DNS:**
- Service: **On**
- Add resource record: Type `A Record`, Name `www.securebank.com`, Address `10.10.30.10` → Add

**Services tab → HTTP:**
- HTTP: **On** (HTTPS optional: On)
- Edit `index.html` → replace with a simple banking portal welcome page if you want a visible demo result

---

## 8. ACL Summary Table

| ACL Name/Number    | Applied On                     | Direction | Purpose |
|----------------------|----------------------------------|-----------|---------|
| MGMT-ONLY (std)     | R-HQ & R-BR2, `line vty 0 4`    | in        | Only Mgmt VLAN (10.10.10.0/24, 10.20.10.0/24) can SSH into routers |
| GUEST-RESTRICT-A    | R-HQ `Gig0/0.40`                | in        | Guest VLAN A can only reach DNS/HTTP server; blocked from Mgmt/Staff subnets both branches |
| GUEST-RESTRICT-B    | R-BR2 `Gig0/0.40`               | in        | Same guest isolation logic for Branch B |
| Port security        | All access switch ports         | —         | Limits MACs per port; sticky learning; violation restrict/shutdown |

---

## 9. Final Testing Checklist

**Layer 2 / VLAN / Trunk**
- [ ] `show vlan brief` on every switch — correct VLANs and ports assigned
- [ ] `show interfaces trunk` — all trunk links up, native VLAN 99 matches on both ends (no "native VLAN mismatch" errors in log)
- [ ] `show port-security interface <intf>` — sticky MACs learned correctly

**Layer 3 / Router-on-a-Stick**
- [ ] `show ip interface brief` on R-HQ and R-BR2 — all sub-interfaces up/up
- [ ] `show vlans` (router side, if supported) confirms sub-interface-to-VLAN mapping

**DHCP**
- [ ] PC-Staff1-A, PC-Staff2-A, PC-Guest1-A/2-A get correct 10.10.20.x / 10.10.40.x leases (`ipconfig` on PC Desktop)
- [ ] Branch B staff/guest PCs get 10.20.20.x / 10.20.40.x leases with DNS server 10.10.30.10

**OSPF / WAN**
- [ ] `show ip ospf neighbor` on both routers — neighbor state **FULL**
- [ ] `show ip route ospf` — each router sees the other branch's VLAN subnets via O
- [ ] `show controllers serial0/0/0` — confirms DCE/clock rate on R-HQ side

**DNS / HTTP**
- [ ] From any PC browser: `http://www.securebank.com` resolves and loads the page
- [ ] `nslookup www.securebank.com` from PC command prompt returns 10.10.30.10

**Security / ACL**
- [ ] PC-Mgmt1-A → PC-Mgmt1-B: ping succeeds (mgmt-to-mgmt allowed across WAN)
- [ ] PC-Guest1-A → PC-Staff1-A: ping **fails** (blocked by GUEST-RESTRICT-A)
- [ ] PC-Guest1-A → Server-DNS-HTTP-A (HTTP): succeeds
- [ ] PC-Guest1-A → R-HQ SSH: fails (not in MGMT-ONLY ACL)
- [ ] PC-Mgmt1-A → R-HQ: `ssh -l admin 10.10.10.1` succeeds, prompts for `Admin@2026`
- [ ] Plug an unauthorized/extra PC into an access port already at its port-security max → interface goes to `err-disabled` (shutdown ports) or drops traffic (restrict ports); verify with `show port-security interface`

**General**
- [ ] `show running-config` saved with `write memory` on every device
- [ ] All banners display on login (SSH and console)
