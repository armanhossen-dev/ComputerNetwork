# 🏦 Secure Multi-Branch Banking Network Infrastructure

A simulated multi-branch banking network built in **Cisco Packet Tracer**, connecting a Head Office with three branch offices using VLAN segmentation, inter-VLAN routing, centralized DHCP/DNS/HTTP services, and dynamic routing via RIP v2.

> Lab Project — **CSE322: Computer Networks Lab**
> Department of Computer Science and Engineering, Daffodil International University (DIU)
> Supervised by **Mr. Tanvirul Islam**, Lecturer, Dept. of CSE, DIU

---

## 📌 Overview

This project models the network infrastructure of a national banking corporation with a Head Office and three branches, each with its own set of departments (Management, Employees, Finance, IT, ATM). All branches are connected over WAN links and share centralized DHCP, DNS, and HTTP services hosted at the Head Office.

### Objectives
- Connect the Head Office with three branches over WAN links
- Separate departments using VLANs
- Implement inter-VLAN routing (Router-on-a-Stick)
- Provide centralized DHCP service via DHCP relay
- Configure RIP v2 for dynamic routing between sites
- Provide DNS and HTTP services from a central server
- Verify end-to-end communication across all branches

---

## 🧱 Network Architecture

**Devices used:**
- 4× Cisco 2911 Routers (`R1-HQ`, `R2-BRANCH1`, `R3-BRANCH2`, `R4-BRANCH3`)
- 4× Cisco 2960 Switches (`SW1-HQ`, `SW2-BRANCH1`, `SW3-BRANCH2`, `SW4-BRANCH3`)
- 1× Centralized Banking Server (DHCP + DNS + HTTP)
- PCs, wireless access points, and mobile devices per site

**VLANs:**

| VLAN | Name | Purpose |
|---|---|---|
| 10 | MANAGEMENT | Admin/management devices |
| 20 | EMPLOYEES | General staff access |
| 30 | FINANCE | Finance department |
| 40 | IT | IT department (HQ only) |
| 50 | ATM | ATM services |
| 60 | SERVERS | Centralized server (HQ only) |

**WAN Links (/30):**

| Link | Subnet |
|---|---|
| R1-HQ ↔ R2-BRANCH1 | 172.16.12.0/30 |
| R2-BRANCH1 ↔ R3-BRANCH2 | 172.16.23.0/30 |
| R3-BRANCH2 ↔ R4-BRANCH3 | 172.16.34.0/30 |

**Routing Protocol:** RIP v2 (no auto-summary)

**Centralized Server:** `10.1.60.10` — DHCP, DNS (`bank.local`), HTTP (banking portal homepage)

---

## ⚙️ Services Implemented

- **VLAN Segmentation** — isolates departmental traffic per site
- **Inter-VLAN Routing** — Router-on-a-Stick using sub-interfaces with 802.1Q trunking
- **Centralized DHCP** — via `ip helper-address` relay to `10.1.60.10`
- **DNS** — resolves `bank.local` / `www.bank.local` to the central server
- **HTTP** — hosts a basic banking portal homepage
- **Dynamic Routing** — RIP v2 across HQ and all branches

---

## 🧪 Verification & Testing

The network was verified using:
- `ping` — connectivity between routers, branches, and the central server
- `show ip interface brief` — interface status and IP assignment
- `show ip protocols` — RIP configuration and routing sources
- `show vlan brief` — VLAN membership per switch
- `show interfaces trunk` — trunk link status and allowed VLANs

---

## 📂 Repository Contents

- `Project_Report.pdf` — full lab report (architecture, IP addressing, screenshots, discussion)
- `banking_network.pkt` — Cisco Packet Tracer project file
- Configuration exports for routers and switches

---

## ⚠️ Limitations & Future Work

This is a simulated environment only — it does not implement production-grade security. Planned improvements:
- Firewall protection and ACLs
- VPN connectivity between sites
- Stronger access control
- Support for additional branches and users

---

## 👥 Team

| Name | Student ID | GitHub |
|---|---|---|
| Md. Arman Hossen Ripon | 241-15-883 | [@ArmanHossenRipon](https://github.com/ArmanHossenRipon) <!-- update with your actual GitHub username --> |
| Md. Hasibur Rahman | 241-15-806 | [@username](https://github.com/username) <!-- add GitHub username --> |
| Md. Wahidur Rahman | 241-15-865 | [@username](https://github.com/username) <!-- add GitHub username --> |
| Md Sabbir Hossine | 241-15-673 | [@username](https://github.com/username) <!-- add GitHub username --> |

*Dept. of Computer Science and Engineering, Daffodil International University*

---

## 📚 References

1. Kurose & Ross, *Computer Networking: A Top-Down Approach*, Pearson
2. Forouzan, *Data Communications and Networking*, McGraw-Hill
3. Wendell Odom, *CCNA 200-301 Official Cert Guide, Volume 1*, Cisco Press
4. Cisco Networking Academy — Cisco Packet Tracer
5. Cisco Systems — Cisco IOS Software Documentation
