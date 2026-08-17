# Secure Multi-Branch Banking Network Infrastructure
### Cisco Packet Tracer Build Guide (HQ + 2 Branches + ISP/Internet)

**Design summary:** HQ uses a Layer‑3 core switch for inter‑VLAN routing (high port density, wire‑speed switching — justified because HQ carries the most VLANs/traffic and a router would bottleneck it). Both branches use **router‑on‑a‑stick** instead (cheap, low VLAN count, no L3 switch budget needed — classic branch‑office tradeoff). All Internet access is centralized through HQ (single firewall/NAT chokepoint — standard PCI/banking practice) instead of local breakout at every branch. Branch‑1 gets true WAN redundancy (dual serial carriers, OSPF‑cost failover) because it's the larger branch; Branch‑2 is a small stub site on a single static‑routed link (cost tradeoff, explicitly called out).

---

## 1. DEVICE LIST

| # | Device | Exact PT Model | Qty | Purpose | Interfaces Used |
|---|--------|----------------|-----|---------|------------------|
| 1 | HQ Router | **Cisco 2911** + 2x **HWIC‑2T** (slot 0 & slot 1) | 1 | HQ WAN edge, OSPF/static, NAT/PAT, static NAT, ACLs | Gig0/0, Gig0/1, Serial0/0/0, Serial0/0/1, Serial0/1/0 |
| 2 | Branch‑1 Router | **Cisco 2911** + 1x **HWIC‑2T** (slot 0) | 1 | Branch router‑on‑a‑stick + dual WAN (primary/backup) | Gig0/0 (+ sub‑ints .20/.30/.100), Serial0/0/0, Serial0/0/1 |
| 3 | Branch‑2 Router | **Cisco 1941** + 1x **HWIC‑2T** (slot 0) | 1 | Small‑branch router‑on‑a‑stick, single static WAN link | Gig0/0 (+ sub‑ints .20/.30/.100), Serial0/0/0 |
| 4 | ISP Router | **Cisco 2911** | 1 | Simulated Internet edge / upstream ISP | Gig0/0, Gig0/1 |
| 5 | HQ Core Switch | **Cisco 3560‑24PS** (multilayer) | 1 | Inter‑VLAN routing (SVIs), OSPF, DHCP relay, ACLs | Gi0/1 (routed), Gi0/2 (trunk), Fa0/1‑0/5 (servers) |
| 6 | HQ Access Switch | **Cisco 2960‑24TT** | 1 | HQ staff/wireless access layer | Gi0/1 (trunk), Fa0/1‑0/2 (PCs), Fa0/9 (AP) |
| 7 | Branch‑1 Switch | **Cisco 2960‑24TT** | 1 | BR1 access layer | Gi0/1 (trunk), Fa0/1‑0/2 (PCs), Fa0/5 (AP) |
| 8 | Branch‑2 Switch | **Cisco 2960‑24TT** | 1 | BR2 access layer | Gi0/1 (trunk), Fa0/1 (PC), Fa0/3 (AP) |
| 9 | HQ Access Point | **AP‑PT‑AC** (Access Point 802.11ac) | 1 | HQ wireless (WPA2) | Port 1 (Ethernet), Radio |
| 10 | BR1 Access Point | **AP‑PT‑AC** | 1 | Branch‑1 wireless (WPA2) | Port 1, Radio |
| 11 | BR2 Access Point | **AP‑PT‑AC** | 1 | Branch‑2 wireless (WPA2) | Port 1, Radio |
| 12 | DHCP Server | **Server‑PT** | 1 | GUI DHCP for HQ Staff + HQ Wireless VLANs | FastEthernet0 |
| 13 | DNS Server | **Server‑PT** | 1 | Internal DNS (A records for web/mail/ftp) | FastEthernet0 |
| 14 | Web Server | **Server‑PT** | 1 | HTTP — also the public‑facing bank website (Static NAT) | FastEthernet0 |
| 15 | Mail Server | **Server‑PT** | 1 | SMTP + POP3 | FastEthernet0 |
| 16 | FTP Server | **Server‑PT** | 1 | FTP file transfer | FastEthernet0 |
| 17 | Internet Host PC | **PC‑PT** | 1 | Simulates an external Internet user (tests NAT/Static NAT from outside) | FastEthernet0 |
| 18 | HQ PCs | **PC‑PT** | 2 | HQ staff workstations (VLAN 20) | FastEthernet0 |
| 19 | HQ Wireless Laptop | **Laptop‑PT** | 1 | HQ wireless client (ships with built‑in wireless NIC) | Wireless0 |
| 20 | BR1 PCs | **PC‑PT** | 2 | Branch‑1 staff workstations | FastEthernet0 |
| 21 | BR1 Wireless Laptop | **Laptop‑PT** | 1 | Branch‑1 wireless client | Wireless0 |
| 22 | BR2 PC | **PC‑PT** | 1 | Branch‑2 staff workstation | FastEthernet0 |
| 23 | BR2 Wireless Laptop | **Laptop‑PT** | 1 | Branch‑2 wireless client | Wireless0 |

**Total: 4 routers, 4 switches, 3 APs, 5 servers, 8 PCs/laptops, 1 external test PC — 25 devices.** Large enough to demonstrate every requirement, small enough to build and demo in one sitting.

> Note on the router models: the 2911/1941 do **not** ship with serial ports — you must drag the correct **HWIC‑2T** module into an available slot (power the router off first, drag the module in, power back on) to get `Serial0/0/0`/`Serial0/0/1` (and `Serial0/1/0`/`Serial0/1/1` if a second HWIC is added). This is real PT hardware behavior, not invented.

---

## 2. DEVICE NAMING

| Device | Hostname |
|---|---|
| HQ Router | `HQ-RTR` |
| Branch‑1 Router | `BR1-RTR` |
| Branch‑2 Router | `BR2-RTR` |
| ISP Router | `ISP-RTR` |
| HQ Core Switch | `HQ-L3SW` |
| HQ Access Switch | `HQ-ASW1` |
| Branch‑1 Switch | `BR1-SW` |
| Branch‑2 Switch | `BR2-SW` |
| HQ AP | `HQ-AP1` |
| BR1 AP | `BR1-AP1` |
| BR2 AP | `BR2-AP1` |
| DHCP Server | `HQ-DHCP-SRV` |
| DNS Server | `HQ-DNS-SRV` |
| Web Server | `HQ-WEB-SRV` |
| Mail Server | `HQ-MAIL-SRV` |
| FTP Server | `HQ-FTP-SRV` |
| Internet host PC | `INET-HOST-PC` |
| HQ PCs | `HQ-PC1`, `HQ-PC2` |
| HQ Laptop | `HQ-LAPTOP-WIFI` |
| BR1 PCs | `BR1-PC1`, `BR1-PC2` |
| BR1 Laptop | `BR1-LAPTOP-WIFI` |
| BR2 PC | `BR2-PC1` |
| BR2 Laptop | `BR2-LAPTOP-WIFI` |

---

## 3. PHYSICAL TOPOLOGY — Build Steps

**Step order in Packet Tracer:**
1. Drag all devices onto the canvas in three visual clusters: **HQ** (center), **Branch‑1** (left), **Branch‑2** (right), plus **ISP** above HQ, and `INET-HOST-PC` above the ISP router.
2. For `HQ-RTR` and `BR1-RTR`: power off → add **2x HWIC‑2T** (HQ) / **1x HWIC‑2T** (BR1) → power on.
3. For `BR2-RTR`: power off → add **1x HWIC‑2T** → power on.
4. Rename every device (click device → **Config tab → Global Settings → Display Name**, or `hostname` in CLI) to match Section 2.
5. Cable everything per the table below (Connections → correct cable icon in PT's cable palette). PT 8.x auto‑MDIX means straight‑through copper works on every Ethernet link (PC–switch, switch–switch, switch–router) — you do not need crossover cables.
6. For every serial link, use the **Serial DCE** cable; whichever end you connect first becomes arbitrary, but you must explicitly set the **HQ‑RTR end as DCE** (it owns the clock) — Packet Tracer lets you pick which end is DCE in the cable properties / by which interface gets `clock rate` configured.

### Port-to-Port Connection Table

| # | Source Device / Port | Cable Type | Destination Device / Port | Purpose |
|---|---|---|---|---|
| 1 | HQ-RTR Gig0/0 | Copper Straight-Through | HQ-L3SW Gi0/1 | Routed transit link, HQ LAN uplink |
| 2 | HQ-RTR Gig0/1 | Copper Straight-Through | ISP-RTR Gig0/0 | Internet edge (NAT outside) |
| 3 | HQ-RTR Serial0/0/0 | Serial DCE (clock on HQ) | BR1-RTR Serial0/0/0 | Primary WAN (leased line) |
| 4 | HQ-RTR Serial0/0/1 | Serial DCE (clock on HQ) | BR1-RTR Serial0/0/1 | Backup WAN (secondary carrier) |
| 5 | HQ-RTR Serial0/1/0 | Serial DCE (clock on HQ) | BR2-RTR Serial0/0/0 | Branch‑2 WAN (static‑routed, no redundancy) |
| 6 | ISP-RTR Gig0/1 | Copper Straight-Through | INET-HOST-PC FastEthernet0 | Simulated external Internet host |
| 7 | HQ-L3SW Gig0/2 | Copper Straight-Through (802.1Q trunk) | HQ-ASW1 Gig0/1 | HQ core-to-access trunk |
| 8 | HQ-L3SW Fa0/1 | Copper Straight-Through | HQ-DHCP-SRV FastEthernet0 | Server farm (VLAN 10) |
| 9 | HQ-L3SW Fa0/2 | Copper Straight-Through | HQ-DNS-SRV FastEthernet0 | Server farm (VLAN 10) |
| 10 | HQ-L3SW Fa0/3 | Copper Straight-Through | HQ-WEB-SRV FastEthernet0 | Server farm (VLAN 10) |
| 11 | HQ-L3SW Fa0/4 | Copper Straight-Through | HQ-MAIL-SRV FastEthernet0 | Server farm (VLAN 10) |
| 12 | HQ-L3SW Fa0/5 | Copper Straight-Through | HQ-FTP-SRV FastEthernet0 | Server farm (VLAN 10) |
| 13 | HQ-ASW1 Fa0/1 | Copper Straight-Through | HQ-PC1 FastEthernet0 | Staff PC (VLAN 20) |
| 14 | HQ-ASW1 Fa0/2 | Copper Straight-Through | HQ-PC2 FastEthernet0 | Staff PC (VLAN 20) |
| 15 | HQ-ASW1 Fa0/9 | Copper Straight-Through | HQ-AP1 Port 1 | Wireless AP uplink (VLAN 30, access port) |
| 16 | BR1-RTR Gig0/0 | Copper Straight-Through (802.1Q trunk) | BR1-SW Gig0/1 | Router-on-a-stick trunk |
| 17 | BR1-SW Fa0/1 | Copper Straight-Through | BR1-PC1 FastEthernet0 | Staff PC (VLAN 20) |
| 18 | BR1-SW Fa0/2 | Copper Straight-Through | BR1-PC2 FastEthernet0 | Staff PC (VLAN 20) |
| 19 | BR1-SW Fa0/5 | Copper Straight-Through | BR1-AP1 Port 1 | Wireless AP uplink (VLAN 30) |
| 20 | BR2-RTR Gig0/0 | Copper Straight-Through (802.1Q trunk) | BR2-SW Gig0/1 | Router-on-a-stick trunk |
| 21 | BR2-SW Fa0/1 | Copper Straight-Through | BR2-PC1 FastEthernet0 | Staff PC (VLAN 20) |
| 22 | BR2-SW Fa0/3 | Copper Straight-Through | BR2-AP1 Port 1 | Wireless AP uplink (VLAN 30) |

**Wireless (no cable):** `HQ-LAPTOP-WIFI`, `BR1-LAPTOP-WIFI`, `BR2-LAPTOP-WIFI` associate to their local AP over radio — just place them within AP range on the canvas.

---

## 4. VLSM / IP ADDRESSING TABLE

**Master space:** `172.16.0.0/16` (bank‑owned private range), split into three `/24`s for clarity:
- `172.16.0.0/24` → WAN transit links (subnetted into /30s)
- `172.16.1.0/24` → HQ LAN (VLSM)
- `172.16.2.0/24` → Branch‑1 LAN (VLSM)
- `172.16.3.0/24` → Branch‑2 LAN (VLSM)

Plus a small **public/ISP** block `203.0.113.0/24` (documentation range, standing in for real public IPs) for the Internet edge and the bank's NAT'd public block.

### VLSM Math — HQ LAN example (172.16.1.0/24)
Requirement: Servers 5 hosts→need /28(14), Staff 45 hosts→need /26(62), Wireless 25 hosts→need /27(30), Mgmt 4 hosts→need /29(6). Sort largest‑first and carve sequentially from 172.16.1.0/24:

1. **Staff (needs 62 usable):** borrow 2 bits from host portion → `/26` = 172.16.1.**0**/26 (range .0–.63)
2. **Servers (needs 14 usable):** next block → `/28` = 172.16.1.**64**/28 (range .64–.79)
3. **Wireless (needs 30 usable):** next block → `/27` = 172.16.1.**96**/27 (range .96–.127)
4. **Mgmt (needs 6 usable):** next block → `/29` = 172.16.1.**128**/29 (range .128–.135)

The same largest‑first VLSM method was applied to Branch‑1 (172.16.2.0/24) and Branch‑2 (172.16.3.0/24) at smaller scale, and to the WAN block (172.16.0.0/24 → /30s).

### Full Addressing Table

| Subnet / VLAN | Network | Mask | Usable Range | Gateway | Purpose |
|---|---|---|---|---|---|
| VLAN 20 – HQ Staff | 172.16.1.0/26 | 255.255.255.192 | .1–.62 | 172.16.1.1 | HQ wired staff PCs |
| VLAN 10 – HQ Servers | 172.16.1.64/28 | 255.255.255.240 | .65–.78 | 172.16.1.65 | HQ server farm |
| VLAN 30 – HQ Wireless | 172.16.1.96/27 | 255.255.255.224 | .97–.126 | 172.16.1.97 | HQ wireless clients |
| VLAN 100 – HQ Mgmt | 172.16.1.128/29 | 255.255.255.248 | .129–.134 | 172.16.1.129 | Switch/AP management |
| VLAN 20 – BR1 Staff | 172.16.2.0/27 | 255.255.255.224 | .1–.30 | 172.16.2.1 | BR1 wired staff PCs |
| VLAN 30 – BR1 Wireless | 172.16.2.32/28 | 255.255.255.240 | .33–.46 | 172.16.2.33 | BR1 wireless clients |
| VLAN 100 – BR1 Mgmt | 172.16.2.48/29 | 255.255.255.248 | .49–.54 | 172.16.2.49 | BR1 switch/AP management |
| VLAN 20 – BR2 Staff | 172.16.3.0/28 | 255.255.255.240 | .1–.14 | 172.16.3.1 | BR2 wired staff PC |
| VLAN 30 – BR2 Wireless | 172.16.3.16/29 | 255.255.255.248 | .17–.22 | 172.16.3.17 | BR2 wireless clients |
| VLAN 100 – BR2 Mgmt | 172.16.3.24/30 | 255.255.255.252 | .25–.26 | 172.16.3.25 | BR2 switch/AP management |
| WAN: HQ↔L3SW transit | 172.16.0.0/30 | 255.255.255.252 | .1–.2 | — | HQ-RTR↔HQ-L3SW routed link |
| WAN: HQ↔BR1 **Primary** | 172.16.0.4/30 | 255.255.255.252 | .5–.6 | — | HQ-RTR S0/0/0 ↔ BR1-RTR S0/0/0 |
| WAN: HQ↔BR1 **Backup** | 172.16.0.8/30 | 255.255.255.252 | .9–.10 | — | HQ-RTR S0/0/1 ↔ BR1-RTR S0/0/1 |
| WAN: HQ↔BR2 (static) | 172.16.0.12/30 | 255.255.255.252 | .13–.14 | — | HQ-RTR S0/1/0 ↔ BR2-RTR S0/0/0 |
| WAN: HQ↔ISP (Internet edge) | 203.0.113.0/30 | 255.255.255.252 | .1–.2 | — | HQ-RTR Gi0/1 (outside) ↔ ISP-RTR Gi0/0 |
| ISP↔Internet host | 203.0.113.4/30 | 255.255.255.252 | .5–.6 | — | ISP-RTR Gi0/1 ↔ INET-HOST-PC |
| Bank's public block | 203.0.113.8/29 | 255.255.255.248 | .9–.14 | — | Static NAT / future public use |

**Interface IP assignments:**
- HQ-RTR: Gi0/0 = 172.16.0.1/30, Gi0/1 = 203.0.113.2/30, S0/0/0 = 172.16.0.5/30 (DCE), S0/0/1 = 172.16.0.9/30 (DCE), S0/1/0 = 172.16.0.13/30 (DCE), Lo0 = 1.1.1.1/32
- BR1-RTR: Gi0/0.20 = 172.16.2.1/27, Gi0/0.30 = 172.16.2.33/28, Gi0/0.100 = 172.16.2.49/29, S0/0/0 = 172.16.0.6/30, S0/0/1 = 172.16.0.10/30, Lo0 = 2.2.2.2/32
- BR2-RTR: Gi0/0.20 = 172.16.3.1/28, Gi0/0.30 = 172.16.3.17/29, Gi0/0.100 = 172.16.3.25/30, S0/0/0 = 172.16.0.14/30
- ISP-RTR: Gi0/0 = 203.0.113.1/30, Gi0/1 = 203.0.113.5/30
- HQ-L3SW: Gi0/1 (routed) = 172.16.0.2/30, SVI10 = 172.16.1.65/28, SVI20 = 172.16.1.1/26, SVI30 = 172.16.1.97/27, SVI100 = 172.16.1.129/29, Lo0 (router‑id only) = 3.3.3.3/32
- HQ-ASW1: SVI100 = 172.16.1.130/29
- BR1-SW: SVI100 = 172.16.2.50/29
- BR2-SW: SVI100 = 172.16.3.26/30
- Servers: `HQ-DHCP-SRV` .66, `HQ-DNS-SRV` .67, `HQ-WEB-SRV` .68, `HQ-MAIL-SRV` .69, `HQ-FTP-SRV` .70 (all /28 172.16.1.64 network, gw .65)
- APs (static): `HQ-AP1` 172.16.1.100, `BR1-AP1` 172.16.2.40, `BR2-AP1` 172.16.3.20
- `INET-HOST-PC`: 203.0.113.6/30, gw 203.0.113.5
- `HQ-WEB-SRV` public (Static NAT): **203.0.113.10**

---

## 5. VLAN PLAN

| VLAN ID | Name | DHCP or Static? | Where DHCP Comes From | Why |
|---|---|---|---|---|
| 10 | SERVERS | **Static** | N/A | Servers must have fixed, predictable IPs (DNS/NAT/records all point to them) |
| 20 | STAFF | DHCP | HQ: Server‑PT (`HQ-DHCP-SRV`) via relay · BR1/BR2: router `ip dhcp pool` | Staff PCs are numerous and rotate — DHCP saves admin time |
| 30 | WIRELESS | DHCP | HQ: Server‑PT (`HQ-DHCP-SRV`) via relay · BR1/BR2: router `ip dhcp pool` | Wireless clients are transient (laptops/phones) — DHCP is essential |
| 100 | MGMT | **Static** | N/A | Management plane (switch/AP IPs, VTY access-class source) must be static and tightly controlled for security |
| 999 | NATIVE-UNUSED | N/A (no hosts) | N/A | Dedicated black‑hole native VLAN on every trunk so untagged/VLAN‑hopping traffic has nowhere useful to go (security best practice) |

**Why HQ uses Server‑PT DHCP but branches use router DHCP:** HQ is the central IT site — it already has dedicated servers, so a real DHCP server gives centralized logging, larger scope management, and matches how banks typically operate their head office. The branches are small and don't warrant a dedicated DHCP appliance — the branch router already exists as the default gateway, so `ip dhcp pool` on the router is the more cost‑effective choice with zero extra hardware. Because `HQ-DHCP-SRV` lives in VLAN 10 but must serve VLAN 20 and VLAN 30 clients, **`ip helper-address`** (DHCP relay) is configured on the HQ‑L3SW SVI20 and SVI30 interfaces.

---

## 6. ROUTER CLI (complete, per device)

### `HQ-RTR`
```
enable
configure terminal
hostname HQ-RTR
no ip domain-lookup
ip domain-name hqbank.local
enable secret Cisco12345!
service password-encryption
username admin privilege 15 secret Cisco12345!
crypto key generate rsa
1024
ip ssh version 2
banner motd # AUTHORIZED ACCESS ONLY - HQ-BANK EDGE ROUTER #
!
interface GigabitEthernet0/0
 description LINK-to-HQ-L3SW-TRANSIT
 ip address 172.16.0.1 255.255.255.252
 ip nat inside
 no shutdown
!
interface GigabitEthernet0/1
 description LINK-to-ISP-OUTSIDE
 ip address 203.0.113.2 255.255.255.252
 ip nat outside
 ip access-group 101 in
 no shutdown
!
interface Serial0/0/0
 description PRIMARY-WAN-to-BR1
 ip address 172.16.0.5 255.255.255.252
 ip nat inside
 clock rate 64000
 no shutdown
!
interface Serial0/0/1
 description BACKUP-WAN-to-BR1
 ip address 172.16.0.9 255.255.255.252
 ip nat inside
 ip ospf cost 5000
 clock rate 64000
 no shutdown
!
interface Serial0/1/0
 description WAN-to-BR2-STATIC
 ip address 172.16.0.13 255.255.255.252
 ip nat inside
 clock rate 64000
 no shutdown
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
!
router ospf 1
 router-id 1.1.1.1
 network 172.16.0.0 0.0.0.3 area 0
 network 172.16.0.4 0.0.0.3 area 0
 network 172.16.0.8 0.0.0.3 area 0
 network 1.1.1.1 0.0.0.0 area 0
 redistribute static subnets
 default-information originate
!
ip route 172.16.3.0 255.255.255.0 172.16.0.14
ip route 0.0.0.0 0.0.0.0 203.0.113.1
!
ip access-list standard 10
 permit 172.16.1.128 0.0.0.7
 deny any
!
access-list 101 remark Inbound from Internet
access-list 101 permit tcp any host 203.0.113.10 eq 80
access-list 101 permit tcp any host 203.0.113.2 established
access-list 101 permit icmp any host 203.0.113.2 echo-reply
access-list 101 permit icmp any host 203.0.113.2 time-exceeded
access-list 101 permit icmp any host 203.0.113.10 echo-reply
access-list 101 deny ip any any log
!
access-list 1 permit 172.16.0.0 0.0.3.255
ip nat inside source list 1 interface GigabitEthernet0/1 overload
ip nat inside source static 172.16.1.68 203.0.113.10
!
line con 0
 logging synchronous
 login local
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
!
end
write memory
```

### `BR1-RTR`
```
enable
configure terminal
hostname BR1-RTR
no ip domain-lookup
ip domain-name hqbank.local
enable secret Cisco12345!
service password-encryption
username admin privilege 15 secret Cisco12345!
crypto key generate rsa
1024
ip ssh version 2
banner motd # AUTHORIZED ACCESS ONLY - BRANCH-1 ROUTER #
!
interface GigabitEthernet0/0
 no shutdown
!
interface GigabitEthernet0/0.20
 description STAFF-VLAN
 encapsulation dot1Q 20
 ip address 172.16.2.1 255.255.255.224
!
interface GigabitEthernet0/0.30
 description WIRELESS-VLAN
 encapsulation dot1Q 30
 ip address 172.16.2.33 255.255.255.240
!
interface GigabitEthernet0/0.100
 description MGMT-VLAN
 encapsulation dot1Q 100
 ip address 172.16.2.49 255.255.255.248
!
interface Serial0/0/0
 description PRIMARY-WAN-to-HQ
 ip address 172.16.0.6 255.255.255.252
 no shutdown
!
interface Serial0/0/1
 description BACKUP-WAN-to-HQ
 ip address 172.16.0.10 255.255.255.252
 ip ospf cost 5000
 no shutdown
!
interface Loopback0
 ip address 2.2.2.2 255.255.255.255
!
router ospf 1
 router-id 2.2.2.2
 network 172.16.0.4 0.0.0.3 area 0
 network 172.16.0.8 0.0.0.3 area 0
 network 172.16.2.0 0.0.0.31 area 0
 network 172.16.2.32 0.0.0.15 area 0
 network 172.16.2.48 0.0.0.7 area 0
 network 2.2.2.2 0.0.0.0 area 0
!
ip dhcp excluded-address 172.16.2.1
ip dhcp excluded-address 172.16.2.33 172.16.2.40
!
ip dhcp pool BR1-STAFF
 network 172.16.2.0 255.255.255.224
 default-router 172.16.2.1
 dns-server 172.16.1.67
!
ip dhcp pool BR1-WIRELESS
 network 172.16.2.32 255.255.255.240
 default-router 172.16.2.33
 dns-server 172.16.1.67
!
ip access-list standard 10
 permit 172.16.2.48 0.0.0.7
 permit 172.16.1.128 0.0.0.7
 deny any
!
line con 0
 logging synchronous
 login local
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
!
end
write memory
```

### `BR2-RTR`
```
enable
configure terminal
hostname BR2-RTR
no ip domain-lookup
ip domain-name hqbank.local
enable secret Cisco12345!
service password-encryption
username admin privilege 15 secret Cisco12345!
crypto key generate rsa
1024
ip ssh version 2
banner motd # AUTHORIZED ACCESS ONLY - BRANCH-2 ROUTER #
!
interface GigabitEthernet0/0
 no shutdown
!
interface GigabitEthernet0/0.20
 description STAFF-VLAN
 encapsulation dot1Q 20
 ip address 172.16.3.1 255.255.255.240
!
interface GigabitEthernet0/0.30
 description WIRELESS-VLAN
 encapsulation dot1Q 30
 ip address 172.16.3.17 255.255.255.248
!
interface GigabitEthernet0/0.100
 description MGMT-VLAN
 encapsulation dot1Q 100
 ip address 172.16.3.25 255.255.255.252
!
interface Serial0/0/0
 description WAN-to-HQ-STATIC-ONLY
 ip address 172.16.0.14 255.255.255.252
 no shutdown
!
ip dhcp excluded-address 172.16.3.1
ip dhcp excluded-address 172.16.3.17 172.16.3.20
!
ip dhcp pool BR2-STAFF
 network 172.16.3.0 255.255.255.240
 default-router 172.16.3.1
 dns-server 172.16.1.67
!
ip dhcp pool BR2-WIRELESS
 network 172.16.3.16 255.255.255.248
 default-router 172.16.3.17
 dns-server 172.16.1.67
!
ip route 0.0.0.0 0.0.0.0 172.16.0.13
!
ip access-list standard 10
 permit 172.16.3.24 0.0.0.3
 permit 172.16.1.128 0.0.0.7
 deny any
!
line con 0
 logging synchronous
 login local
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
!
end
write memory
```

### `ISP-RTR`
```
enable
configure terminal
hostname ISP-RTR
enable secret Cisco12345!
service password-encryption
!
interface GigabitEthernet0/0
 description LINK-to-HQ-RTR
 ip address 203.0.113.1 255.255.255.252
 no shutdown
!
interface GigabitEthernet0/1
 description LINK-to-INET-HOST-PC
 ip address 203.0.113.5 255.255.255.252
 no shutdown
!
ip route 203.0.113.8 255.255.255.248 203.0.113.2
!
end
write memory
```
*(ISP-RTR represents the "outside world" — it only needs a route back to the bank's NAT'd public block; it deliberately has no visibility into the bank's private 172.16.0.0/16 network.)*

---

## 7. SWITCH CLI (complete, per device)

### `HQ-L3SW`
```
enable
configure terminal
hostname HQ-L3SW
ip routing
enable secret Cisco12345!
service password-encryption
username admin privilege 15 secret Cisco12345!
!
vlan 10
 name SERVERS
vlan 20
 name STAFF
vlan 30
 name WIRELESS
vlan 100
 name MGMT
vlan 999
 name NATIVE-UNUSED
!
interface GigabitEthernet0/1
 description ROUTED-LINK-to-HQ-RTR
 no switchport
 ip address 172.16.0.2 255.255.255.252
 no shutdown
!
interface GigabitEthernet0/2
 description TRUNK-to-HQ-ASW1
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 20,30,100,999
!
interface range FastEthernet0/1 - 5
 description SERVER-FARM-PORTS
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
!
interface Vlan10
 ip address 172.16.1.65 255.255.255.240
!
interface Vlan20
 ip address 172.16.1.1 255.255.255.192
 ip helper-address 172.16.1.66
 ip access-group 120 in
!
interface Vlan30
 ip address 172.16.1.97 255.255.255.224
 ip helper-address 172.16.1.66
 ip access-group 120 in
!
interface Vlan100
 ip address 172.16.1.129 255.255.255.248
!
interface Loopback0
 ip address 3.3.3.3 255.255.255.255
!
router ospf 1
 router-id 3.3.3.3
 network 172.16.0.0 0.0.0.3 area 0
 network 172.16.1.0 0.0.0.63 area 0
 network 172.16.1.64 0.0.0.15 area 0
 network 172.16.1.96 0.0.0.31 area 0
 network 172.16.1.128 0.0.0.7 area 0
 network 3.3.3.3 0.0.0.0 area 0
!
ip access-list standard 10
 permit 172.16.1.128 0.0.0.7
 deny any
!
access-list 120 remark Wireless VLAN30 -> only DNS+HTTP to server farm
access-list 120 permit udp 172.16.1.96 0.0.0.31 host 172.16.1.67 eq 53
access-list 120 permit tcp 172.16.1.96 0.0.0.31 host 172.16.1.68 eq 80
access-list 120 deny ip 172.16.1.96 0.0.0.31 172.16.1.64 0.0.0.15
access-list 120 permit ip 172.16.1.96 0.0.0.31 any
access-list 120 permit ip any any
!
line con 0
 login local
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
!
end
write memory
```

### `HQ-ASW1`
```
enable
configure terminal
hostname HQ-ASW1
enable secret Cisco12345!
service password-encryption
username admin privilege 15 secret Cisco12345!
!
vlan 20
 name STAFF
vlan 30
 name WIRELESS
vlan 100
 name MGMT
vlan 999
 name NATIVE-UNUSED
!
interface GigabitEthernet0/1
 description TRUNK-to-HQ-L3SW
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 20,30,100,999
!
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
!
interface FastEthernet0/9
 description UPLINK-to-HQ-AP1
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
!
interface Vlan100
 ip address 172.16.1.130 255.255.255.248
 no shutdown
!
ip default-gateway 172.16.1.129
!
ip access-list standard 10
 permit 172.16.1.128 0.0.0.7
 deny any
!
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
!
end
write memory
```

### `BR1-SW`
```
enable
configure terminal
hostname BR1-SW
enable secret Cisco12345!
service password-encryption
username admin privilege 15 secret Cisco12345!
!
vlan 20
 name STAFF
vlan 30
 name WIRELESS
vlan 100
 name MGMT
vlan 999
 name NATIVE-UNUSED
!
interface GigabitEthernet0/1
 description TRUNK-to-BR1-RTR
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 20,30,100,999
!
interface range FastEthernet0/1 - 2
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
!
interface FastEthernet0/5
 description UPLINK-to-BR1-AP1
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
!
interface Vlan100
 ip address 172.16.2.50 255.255.255.248
 no shutdown
!
ip default-gateway 172.16.2.49
!
ip access-list standard 10
 permit 172.16.2.48 0.0.0.7
 permit 172.16.1.128 0.0.0.7
 deny any
!
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
!
end
write memory
```

### `BR2-SW`
```
enable
configure terminal
hostname BR2-SW
enable secret Cisco12345!
service password-encryption
username admin privilege 15 secret Cisco12345!
!
vlan 20
 name STAFF
vlan 30
 name WIRELESS
vlan 100
 name MGMT
vlan 999
 name NATIVE-UNUSED
!
interface GigabitEthernet0/1
 description TRUNK-to-BR2-RTR
 switchport mode trunk
 switchport trunk native vlan 999
 switchport trunk allowed vlan 20,30,100,999
!
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
!
interface FastEthernet0/3
 description UPLINK-to-BR2-AP1
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
!
interface Vlan100
 ip address 172.16.3.26 255.255.255.252
 no shutdown
!
ip default-gateway 172.16.3.25
!
ip access-list standard 10
 permit 172.16.3.24 0.0.0.3
 permit 172.16.1.128 0.0.0.7
 deny any
!
line vty 0 4
 access-class 10 in
 login local
 transport input ssh
!
end
write memory
```

---

## 8. FIREWALL / ACL CLI — Explanation

All ACLs below already appear in Sections 6–7; here is what each one *does* and why.

**ACL 101 — HQ‑RTR `Gi0/1` inbound (Internet‑facing edge firewall):**
- `permit tcp any host 203.0.113.10 eq 80` — allows the outside world to reach the bank's public website (the Static‑NAT'd web server) on port 80, and nothing else on that host.
- `permit tcp any host 203.0.113.2 established` — allows **return** traffic for TCP sessions that internal users initiated through PAT (the ACK bit must be set), so staff can browse the Internet outbound. New inbound TCP connections to the PAT address are not matched, so they're blocked.
- `permit icmp ... echo-reply / time-exceeded` — allows ping replies and traceroute hops back in, for diagnostics, without opening the router to unsolicited inbound ICMP.
- `deny ip any any log` — explicit "default deny" (with logging) closing everything else from the Internet; this is the actual firewall behavior.

**ACL 120 — HQ‑L3SW `Vlan30` inbound (wireless‑to‑server‑farm segmentation):**
- Permits wireless clients to reach the DNS server (UDP/53) and the web server (TCP/80) only.
- Explicitly denies wireless clients from reaching the rest of the server VLAN (blocking FTP and SMTP/POP3 — financial data and internal file transfers should never be reachable from a Wi‑Fi network).
- A final `permit ip any any` lets wireless clients still reach the Internet and other non‑server destinations normally.

**Standard ACL 1 — HQ‑RTR (NAT source ACL):**
- `permit 172.16.0.0 0.0.3.255` matches the summarized private range `172.16.0.0/22` (covers WAN transit + all three site LANs) — this is what's allowed to be PAT'd out to the Internet.

**Standard ACL 10 — on every device's VTY lines:**
- Restricts Telnet/SSH management access to only the local site's MGMT VLAN plus HQ's MGMT VLAN (so HQ network admins can remotely manage branch gear, but nobody else can). Combined with `transport input ssh` and local usernames, this is the core of "secure" management access.

> **Design note (Router ACL vs ASA 5505):** We used router extended ACLs rather than an ASA 5505 because (a) they run natively on the routers we already need for NAT/OSPF, keeping the topology buildable without extra licensing quirks in PT, and (b) they fully demonstrate stateless packet filtering + the "established" trick for pseudo‑statefulness, which meets the lab's teaching goals. An ASA 5505 would add true stateful inspection and would be the natural next upgrade at the HQ edge in a production deployment.

---

## 9. SERVER-PT GUI STEPS

For every server: click the server → **Desktop tab → IP Configuration** → set Static IP/Subnet/Gateway/DNS exactly per Section 4's table (all gateway = `172.16.1.65`, DNS = `172.16.1.67`).

### `HQ-DHCP-SRV` (172.16.1.66)
Config tab → **DHCP** service → toggle **On**. Add two pools using the **+** button:
- Pool `HQ-STAFF`: Default Gateway `172.16.1.1`, DNS Server `172.16.1.67`, Start IP Address `172.16.1.2`, Subnet Mask `255.255.255.192`, Max Users `60`.
- Pool `HQ-WIRELESS`: Default Gateway `172.16.1.97`, DNS Server `172.16.1.67`, Start IP Address `172.16.1.101`, Subnet Mask `255.255.255.224`, Max Users `25` (starting past `.100` skips the AP's static IP).
Delete/disable the default `pool0` example.

### `HQ-DNS-SRV` (172.16.1.67)
Config tab → **DNS** service → toggle **On**. Add A records:
| Name | Type | Address |
|---|---|---|
| www.hqbank.local | A Record | 172.16.1.68 |
| mail.hqbank.local | A Record | 172.16.1.69 |
| ftp.hqbank.local | A Record | 172.16.1.70 |

### `HQ-WEB-SRV` (172.16.1.68)
Config tab → **HTTP** service → toggle **HTTP On**. Edit the default `index.html` (File Manager) to something identifiable, e.g. change the `<h1>` heading to **"HQ Bank — Internal Portal."**

### `HQ-MAIL-SRV` (172.16.1.69)
Config tab → **EMAIL** service → toggle **SMTP** and **POP3** to **On**. Set Domain Name: `hqbank.local`. Add User Accounts: `alice / Cisco12345!` and `bob / Cisco12345!`.
On each client PC → Desktop → **Email** app: Your Name = staff name, Email Address = `alice@hqbank.local`, Incoming Mail Server = `172.16.1.69`, Outgoing Mail Server = `172.16.1.69`, User Name = `alice`, Password = `Cisco12345!` → Save, then Send/Receive to test.

### `HQ-FTP-SRV` (172.16.1.70)
Config tab → **FTP** service → toggle **On**. Add User: `admin / Cisco12345!` with permissions **Write, Read, Delete, Rename, List** checked. From a client: `ftp 172.16.1.70` (or `ftp ftp.hqbank.local` once DNS is working) at the command prompt, log in, `put`/`get` a file to confirm.

---

## 10. WIRELESS CONFIG

| Site | SSID | Security | Passphrase | VLAN/Subnet |
|---|---|---|---|---|
| HQ | `HQ-Bank-Secure` | WPA2-PSK (AES) | `HQb@nk!2026Wifi` | VLAN 30 / 172.16.1.96/27 |
| Branch-1 | `BR1-Bank-Secure` | WPA2-PSK (AES) | `BR1b@nk!2026Wifi` | VLAN 30 / 172.16.2.32/28 |
| Branch-2 | `BR2-Bank-Secure` | WPA2-PSK (AES) | `BR2b@nk!2026Wifi` | VLAN 30 / 172.16.3.16/29 |

**AP configuration (repeat per AP, e.g. `HQ-AP1`):** click the AP → **Config tab → Port 1 (Wireless)** → set SSID field per table above → **Security Mode: WPA2-PSK** → **Encryption: AES** → enter the Pass Phrase → Port 1 stays on VLAN default (the switchport it's plugged into, e.g. Fa0/9 on `HQ-ASW1`, already carries VLAN 30 as an access port, so the AP simply bridges wireless clients into VLAN 30).

**Connecting a wireless client (e.g. `HQ-LAPTOP-WIFI`):**
1. Click the laptop → **Desktop tab → PC Wireless** (laptops ship with a WPC300N wireless NIC pre‑installed, no hardware swap needed).
2. Under **Connect**, select SSID `HQ-Bank-Secure`.
3. Choose **WPA2-PSK**, enter passphrase `HQb@nk!2026Wifi` → Connect.
4. Go to **Desktop → IP Configuration** → select **DHCP** → confirm it receives an address in `172.16.1.101–.126` and gateway `172.16.1.97`.
5. Test with `ping 172.16.1.68` (web server) and by opening the **Web Browser** app to `http://www.hqbank.local`.

*(For `PC-PT` desktops that need wireless instead of a laptop, you'd power the PC off, remove the `PT-PC-NM-1CFE` Ethernet module, add a `WMP300N` wireless module, and power back on — laptops avoid this step entirely, which is why they were chosen for the wireless client role.)*

---

## 11. OSPF + STATIC + REDISTRIBUTION

| Router | Method | Router ID | Why |
|---|---|---|---|
| `HQ-RTR` | OSPF (area 0) + 2 static routes | 1.1.1.1 | Core of the WAN; needs dynamic routing for the redundant BR1 links, plus static routes for the Internet default and the BR2 stub |
| `HQ-L3SW` | OSPF (area 0) | 3.3.3.3 | Advertises all HQ VLAN subnets into the WAN |
| `BR1-RTR` | OSPF (area 0) | 2.2.2.2 | Has two physical WAN paths to HQ — OSPF's cost metric is what makes automatic failover possible |
| `BR2-RTR` | **Static only** (default route to HQ) | — | Single WAN link, no redundancy to react to — a dynamic protocol adds no value and only adds CPU/complexity for a small stub site |

**Networks / wildcard masks used** (all `area 0`):
- HQ-RTR: `172.16.0.0 0.0.0.3`, `172.16.0.4 0.0.0.3`, `172.16.0.8 0.0.0.3`, `1.1.1.1 0.0.0.0`
- HQ-L3SW: `172.16.0.0 0.0.0.3`, `172.16.1.0 0.0.0.63`, `172.16.1.64 0.0.0.15`, `172.16.1.96 0.0.0.31`, `172.16.1.128 0.0.0.7`, `3.3.3.3 0.0.0.0`
- BR1-RTR: `172.16.0.4 0.0.0.3`, `172.16.0.8 0.0.0.3`, `172.16.2.0 0.0.0.31`, `172.16.2.32 0.0.0.15`, `172.16.2.48 0.0.0.7`, `2.2.2.2 0.0.0.0`

**Why redistribution is needed:** `BR2-RTR` deliberately runs no dynamic routing protocol — it just default-routes everything to HQ. But `HQ-RTR`'s route to BR2's LAN (`ip route 172.16.3.0 255.255.255.0 172.16.0.14`) is a **static** route, invisible to OSPF unless explicitly redistributed. `redistribute static subnets` on `HQ-RTR` injects that BR2 route into OSPF so `HQ-L3SW` and `BR1-RTR` also learn how to reach Branch‑2. Separately, `HQ-RTR`'s Internet default route (`ip route 0.0.0.0 0.0.0.0 203.0.113.1`) is propagated with `default-information originate`, so every OSPF‑speaking router (and, via BR2's own static default, Branch‑2 too) has a path to the Internet — all of it funneled back through HQ's NAT/firewall, matching the "centralized breakout" design decision.

**Verification commands:**
```
show ip ospf neighbor
show ip route ospf
show ip route static
show ip protocols
show ip ospf database
traceroute 172.16.3.1
```
On `HQ-RTR`, `show ip route` should show `O E2` (external type‑2) entries for `172.16.3.0/24` and a `O*E2` default route on the other routers, confirming redistribution is working.

---

## 12. WAN REDUNDANCY

**Failover mechanism: OSPF cost metric.** `HQ-RTR Serial0/0/0` ↔ `BR1-RTR Serial0/0/0` is the **primary** link, running at OSPF's default calculated cost (~64, based on the T1 bandwidth). `HQ-RTR Serial0/0/1` ↔ `BR1-RTR Serial0/0/1` is the **backup** link, manually set to `ip ospf cost 5000` on both ends — an intentionally huge cost so OSPF's SPF algorithm will never prefer it while the primary is up.

**Why OSPF cost instead of floating static:** since HQ↔BR1 is already inside the OSPF domain (both routers, both links), using OSPF's own metric keeps everything in one routing protocol with one source of truth, and it reacts to *any* primary‑path failure (not just an interface going down — e.g., a flapping neighbor or a bad line‑protocol state too), which a floating static route wouldn't do as gracefully.

**How to test it in Packet Tracer:**
1. Before the test, run `show ip route` on `HQ-RTR` and confirm the route to `172.16.2.0/27` (BR1 staff) points out `Serial0/0/0`.
2. On `HQ-RTR`, run `interface Serial0/0/0` → `shutdown`.
3. Wait for OSPF's dead timer (default 40 sec) to expire — `show ip ospf neighbor` will show the S0/0/0 neighbor disappear.
4. Run `show ip route` again — the route to `172.16.2.0/27` should now point out `Serial0/0/1` (the backup), automatically.
5. From `HQ-PC1`, `ping 172.16.2.1` (BR1's gateway) — after the brief convergence gap, connectivity is restored over the backup link.
6. Run `no shutdown` on `Serial0/0/0` to restore the primary — OSPF will reconverge back to the lower‑cost path.

*(Optional enhancement to demo faster convergence: `ip ospf hello-interval 1` / `ip ospf dead-interval 4` on both serial interfaces — reduces the failover gap from ~40s to ~4s.)*

---

## 13. TESTING CHECKLIST

| # | Source | Destination | Expected Result | Tool/Command |
|---|---|---|---|---|
| 1 | `HQ-PC1` | — | Gets an IP in 172.16.1.2–.62 from Server‑PT DHCP | `ipconfig /all` |
| 2 | `BR1-PC1` | — | Gets an IP in 172.16.2.2–.30 from router DHCP pool | `ipconfig /all` |
| 3 | `HQ-LAPTOP-WIFI` | `HQ-Bank-Secure` SSID | Associates with WPA2, gets DHCP IP in .101–.126 | PC Wireless app |
| 4 | `HQ-PC1` (VLAN20) | `HQ-DNS-SRV` (VLAN10) | Ping succeeds (inter‑VLAN routing via HQ-L3SW) | `ping 172.16.1.67` |
| 5 | `HQ-PC1` | `www.hqbank.local` | Page loads, resolves via DNS | Web Browser app |
| 6 | Mail client on `alice@hqbank.local` | `bob@hqbank.local` | Email delivered | Email app Send/Receive |
| 7 | `BR1-PC1` | `HQ-FTP-SRV` | Login succeeds, file `put`/`get` works | `ftp 172.16.1.70` |
| 8 | `BR1-RTR` | `HQ-L3SW` VLAN subnets | OSPF neighbor FULL, routes learned | `show ip ospf neighbor`, `show ip route` |
| 9 | `HQ-RTR` | `BR2-RTR` LAN | Static route + redistribution reachable from `BR1-RTR` | `traceroute 172.16.3.1` from BR1-RTR |
| 10 | `HQ-PC1` | `INET-HOST-PC` (203.0.113.6) | Ping succeeds; source shown as 203.0.113.2 on ISP side (PAT) | `ping`, `show ip nat translations` on HQ-RTR |
| 11 | `INET-HOST-PC` | `203.0.113.10` (Static NAT) | HTTP page loads (bank public website) | Web Browser app on INET-HOST-PC |
| 12 | `INET-HOST-PC` | `HQ-FTP-SRV` real IP or via NAT | **Denied** — ACL 101 only permits port 80 | `ftp` attempt should time out |
| 13 | `HQ-LAPTOP-WIFI` (wireless) | `HQ-FTP-SRV` | **Denied** by ACL 120 (only DNS/HTTP allowed) | `ftp 172.16.1.70` should fail |
| 14 | Any non‑MGMT PC | `HQ-RTR` | SSH connection **refused** | `ssh -l admin 172.16.1.1`-style attempt fails |
| 15 | Admin PC in VLAN100 | `HQ-RTR` | SSH connection succeeds | SSH client, login `admin` |
| 16 | — | HQ↔BR1 primary | Shutting `Serial0/0/0` fails traffic over to backup automatically | Section 12 procedure |
| 17 | `BR2-PC1` | Internet (`INET-HOST-PC`) | Ping succeeds via HQ's centralized NAT | `ping` |

---

## 14. TROUBLESHOOTING

| Symptom | Likely Cause | Command to Diagnose | Fix |
|---|---|---|---|
| Serial link line protocol down | Missing `clock rate` on DCE end, or mismatched encapsulation | `show controllers serial0/0/0`, `show interfaces serial0/0/0` | Add `clock rate 64000` on the HQ‑RTR (DCE) end; ensure `no shutdown` on both ends |
| OSPF neighbors won't form | Wildcard mask wrong, area mismatch, or interface down | `show ip ospf interface brief`, `show ip ospf neighbor` | Fix `network` statement math or `no shutdown` the interface |
| VLAN traffic not reaching router-on-a-stick subinterface | Trunk not allowing that VLAN, or native VLAN mismatch | `show interfaces trunk`, `show vlan brief` | Add VLAN to `switchport trunk allowed vlan`, match native VLAN on both ends |
| DHCP clients not getting an address across VLANs | Missing `ip helper-address` on the SVI | `show ip interface vlan20`, `debug ip udp` | Add `ip helper-address 172.16.1.66` to the SVI |
| Static NAT / PAT not translating | Interfaces not marked `ip nat inside`/`outside`, or NAT ACL wrong | `show ip nat translations`, `show ip nat statistics`, `debug ip nat` | Verify every LAN‑facing interface has `ip nat inside`, WAN has `ip nat outside`; verify ACL 1/101 match the right ranges |
| Can't reach a device via SSH | ACL 10 blocking source, or `transport input` wrong | `show access-lists 10`, `show line vty 0 4` | Confirm source IP is in the permitted MGMT subnet; confirm `transport input ssh` |
| Branch‑2 unreachable from Branch‑1 | Redistribution missing on HQ-RTR | `show ip route ospf` on BR1-RTR | Ensure `redistribute static subnets` is present under `router ospf 1` on HQ-RTR |
| Wireless client can't associate | SSID/passphrase mismatch, or AP/client security mode mismatch | AP Config tab, PC Wireless app | Match SSID exactly, confirm WPA2-PSK/AES on both ends |
| General connectivity/config review | — | `show running-config`, `show ip interface brief`, `show cdp neighbors` | Confirm interfaces are `up/up`, IPs match the addressing table |

---

## 15. FINAL ASCII TOPOLOGY DIAGRAM

```
                                   [INET-HOST-PC]
                                   203.0.113.6/30
                                          |
                                   Gi0/1  |  203.0.113.5/30
                                   +--------------+
                                   |   ISP-RTR    |
                                   +--------------+
                                   Gi0/0  203.0.113.1/30
                                          |
                                          | 203.0.113.0/30
                                   Gi0/1  |  203.0.113.2/30 (NAT outside)
                                   +--------------+
                                   |    HQ-RTR    | Lo0: 1.1.1.1
                                   | (2911+2xHWIC)|
                                   +--------------+
                        S0/0/0 primary |  | S0/0/1 backup |  S0/1/0 static
                     172.16.0.5/30     |  |172.16.0.9/30  |  172.16.0.13/30
                                       |  |               |
                    Gi0/0  172.16.0.1/30 |               |
                                       |  |               |
                              +----------------+          |
                              |    HQ-L3SW     | Lo0:3.3.3.3      |
                              | (3560, routes  |                  |
                              |  VLANs 10/20/  |                  |
                              |  30/100)       |                  |
                              +----------------+                  |
                          Fa0/1-5 |        Gi0/2 (trunk)          |
                    +----+----+---+                |              |
                    |    |    |                     |              |
                 [DHCP][DNS][WEB][MAIL][FTP]   +----------+        |
                 SERVER FARM (VLAN10)          | HQ-ASW1  |        |
                                                +----------+        |
                                          Fa0/1-2 |    | Fa0/9      |
                                       [HQ-PC1/2] |  [HQ-AP1] ))) [HQ-LAPTOP-WIFI]
                                        (VLAN20)     (VLAN30 WPA2)
                                                                    |
      +--------------------------------------------+   +----------+-------------+
      |     S0/0/0 primary   S0/0/1 backup          |   |   S0/0/0 static        |
      | 172.16.0.6/30        172.16.0.10/30         |   |   172.16.0.14/30       |
      |            +----------------+               |   |     +----------------+ |
      |            |   BR1-RTR      | Lo0:2.2.2.2    |   |     |   BR2-RTR      | |
      |            | (2911+HWIC)    |                |   |     | (1941+HWIC)    | |
      |            +----------------+                |   |     +----------------+ |
      |         Gi0/0 trunk (.20/.30/.100)            |         Gi0/0 trunk (.20/.30/.100)
      |            +----------+                                    +----------+
      |            | BR1-SW   |                                    | BR2-SW   |
      |            +----------+                                    +----------+
      |        Fa0/1-2 |    | Fa0/5                            Fa0/1 |   | Fa0/3
      |  [BR1-PC1/2] |  [BR1-AP1] ))) [BR1-LAPTOP-WIFI]     [BR2-PC1]|  [BR2-AP1] ))) [BR2-LAPTOP-WIFI]
      |   (VLAN20)      (VLAN30 WPA2)                        (VLAN20)   (VLAN30 WPA2)
      +---------------------- BRANCH-1 --------------------+  +----- BRANCH-2 (no WAN redundancy) -----+
```

---

**This lab is complete and internally consistent end‑to‑end** — every IP in every CLI block, GUI step, and test case traces back to the single VLSM table in Section 4. Build it in the order of Sections 3 → 6 → 7 → 9 → 10, then run the Section 13 checklist top to bottom.