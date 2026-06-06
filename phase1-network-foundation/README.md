# 🛡️ Enterprise Cybersecurity Homelab

![Phase](https://img.shields.io/badge/Phase-1%20Complete-brightgreen?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-VMware%20Workstation%20Pro-blue?style=for-the-badge&logo=vmware)
![Firewall](https://img.shields.io/badge/Firewall-pfSense%202.8.1-red?style=for-the-badge)
![Cost](https://img.shields.io/badge/Cost-Free%20%2F%20Open%20Source-success?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Active%20Build-orange?style=for-the-badge)

> A fully segmented, enterprise-grade cybersecurity homelab built on a single laptop — designed to mirror the network architecture, security tooling, and operational processes of multinational corporations (MNCs). Built to demonstrate hands-on competency for **SOC Analyst**, **Security Engineer**, **Junior Pentester**, and **Cloud Security Engineer** roles.

---

## 📋 Table of Contents

- [Project Overview](#-project-overview)
- [Architecture Diagram](#-architecture-diagram)
- [Phase Roadmap](#-phase-roadmap)
- [Phase 1 — Network Foundation](#-phase-1--network-foundation)
  - [Hardware & Software](#hardware--software)
  - [Network Design](#network-design)
  - [VMware Configuration](#vmware-configuration)
  - [pfSense Configuration](#pfsense-configuration)
  - [Firewall Aliases](#firewall-aliases)
  - [Firewall Rules](#firewall-rules)
  - [Validation](#validation)
- [MNC Job Relevance](#-mnc-job-relevance)
- [Coming in Phase 2](#-coming-in-phase-2)

---

## 🎯 Project Overview

Most security candidates have theoretical knowledge from certifications. This lab provides **operational experience** — the ability to describe, from direct experience, how enterprise networks are designed, segmented, monitored, and defended.

| Property | Detail |
|----------|--------|
| **Host Machine** | Single Windows 11 laptop |
| **RAM** | 16 GB |
| **Storage** | 500 GB |
| **Hypervisor** | VMware Workstation Pro (free personal use) |
| **Total Cost** | $0 — entirely free and open source |
| **Goal** | MNC security role — SOC / Security Engineer / Pentester / Cloud Security |

---

## 🗺️ Architecture Diagram

```
                          ┌─────────────────────────────────────────────┐
                          │           INTERNET / WAN                    │
                          │     VMware NAT (VMnet8) — 192.168.139.x     │
                          └──────────────────┬──────────────────────────┘
                                             │
                          ┌──────────────────▼──────────────────────────┐
                          │              pfSense 2.8.1                  │
                          │         Firewall + Router + DNS             │
                          │  em0=WAN  em1=LAN  em2=DMZ  em3=BRANCH     │
                          │  em4=SOC  em5=MGMT  em6=ATTACK             │
                          └──┬───────┬────────┬────────┬───────┬────────┘
                             │       │        │        │       │
              ┌──────────────▼─┐ ┌───▼──┐ ┌──▼───┐ ┌─▼───┐ ┌─▼──────┐
              │  LAN / Corp    │ │ DMZ  │ │BRANCH│ │ SOC │ │  MGMT  │
              │  10.10.0.0/24  │ │10.20 │ │10.30 │ │10.40│ │ 10.50  │
              │  AD, DNS, DHCP │ │NGINX │ │ DC   │ │Wazuh│ │Grafana │
              │  App Server    │ │ WAF  │ │DHCP  │ │ ELK │ │Ansible │
              │  DB Server     │ │Honey │ │Win10 │ │Hive │ │NetBox  │
              └────────────────┘ └──────┘ └──────┘ └─────┘ └────────┘
                                                                    
              ┌─────────────────────────────────────────────────────────┐
              │        ATTACK LAB — 10.60.0.0/24 — ISOLATED            │
              │   Kali Linux │ Metasploitable 3 │ DVWA │ Vuln Windows  │
              │         NO ROUTE TO PRODUCTION — DATA DIODE            │
              └─────────────────────────────────────────────────────────┘
```

---

## 🗓️ Phase Roadmap

| Phase | Focus | Key Tools | MNC Role |
|-------|-------|-----------|----------|
| **✅ Phase 1** | Network Foundation | pfSense, VMware, Kali | All roles — baseline |
| **🔄 Phase 2** | Detection & Monitoring | Wazuh SIEM, ELK, Suricata | SOC Analyst L1/L2 |
| **⏳ Phase 3** | Identity & Access | Windows AD, OpenVPN, Teleport | Security / IAM Engineer |
| **⏳ Phase 4** | Attack & Defend | Kali, Metasploitable, TheHive, MISP | Junior Pentester |
| **⏳ Phase 5** | Automation & DevSecOps | Shuffle SOAR, Ansible, Docker, Falco | DevSecOps Engineer |
| **⏳ Phase 6** | Cloud Security | AWS, GuardDuty, Security Hub, CloudTrail | Cloud Security Engineer |

---

## ✅ Phase 1 — Network Foundation

### Hardware & Software

| Component | Value |
|-----------|-------|
| Host OS | Windows 11 |
| Hypervisor | VMware Workstation Pro (free personal license) |
| Firewall | pfSense 2.8.1-RELEASE (Netgate CE) |
| Admin VM | Kali Linux (VMware pre-built image) |
| Total VMs in Phase 1 | 2 (pfSense + Kali) |

---

### Network Design

#### IP Addressing Scheme

A clean, consistent pattern was chosen: **`10.X0.0.0/24`** where the middle octet identifies the zone. Every gateway ends in `.1` and every DHCP pool runs `.100–.200`. All subnets are `/24`.

| # | Zone | Network | Gateway | DHCP Range | VMnet | Interface |
|---|------|---------|---------|------------|-------|-----------|
| 0 | WAN | VMware NAT (auto) | VMware | pfSense only | VMnet8 | em0 |
| 1 | Corporate Core (LAN) | `10.10.0.0/24` | `10.10.0.1` | `.100–.200` | VMnet2 | em1 |
| 2 | DMZ | `10.20.0.0/24` | `10.20.0.1` | `.100–.200` | VMnet3 | em2 |
| 3 | Branch Office | `10.30.0.0/24` | `10.30.0.1` | `.100–.200` | VMnet4 | em3 |
| 4 | SOC | `10.40.0.0/24` | `10.40.0.1` | `.100–.200` | VMnet5 | em4 |
| 5 | Management (MGMT) | `10.50.0.0/24` | `10.50.0.1` | `.100–.200` | VMnet6 | em5 |
| 6 | Attack Lab (ATTACK) | `10.60.0.0/24` | `10.60.0.1` | `.100–.200` | VMnet7 | em6 |

#### Zone Descriptions

**🏢 Corporate Core (LAN) — 10.10.0.0/24**

The primary internal network representing corporate headquarters. Hosts Active Directory, DNS, DHCP, application servers, and database servers. The most populated zone in a real enterprise. Admin VMs operate here to manage lab infrastructure.

**🌐 DMZ — 10.20.0.0/24**

The Demilitarised Zone hosts public-facing services — NGINX reverse proxy, WAF, mail gateway, OpenVPN, Pi-hole DNS sinkhole, and honeypots (Cowrie, Dionaea). Can receive internet connections and initiate outbound connections, but must **never** initiate connections to internal LAN. This is enforced by firewall rules.

**🏬 Branch Office — 10.30.0.0/24**

Simulates a remote office connecting back to HQ. Contains Branch DC, DHCP, and Windows clients. Wazuh agents ship logs to the SOC zone for centralised monitoring — exactly as MNCs monitor remote offices.

**🔍 SOC — 10.40.0.0/24**

Security Operations Center. Hosts Wazuh SIEM, ELK Stack, OpenVAS, TheHive, MISP, and Shuffle SOAR. Has read access to every zone for log collection and investigation. Cannot reach the Attack Lab.

**⚙️ Management (MGMT) — 10.50.0.0/24**

Most privileged zone. Hosts Grafana, Prometheus, Ansible, NetBox, and the Bastion/PAM host. All infrastructure management flows through this zone only — implements Privileged Access Workstation (PAW) model used in MNC environments.

**⚔️ Attack Lab (ATTACK) — 10.60.0.0/24 — ISOLATED**

Completely isolated offensive security environment. Contains Kali Linux, Metasploitable 3, DVWA, and vulnerable Windows VMs. **Zero routes to any internal zone.** Enforced by explicit firewall block as the first rule on the ATTACK interface.

> `NO ROUTE TO PRODUCTION — COMPLETELY ISOLATED — ONE-WAY DATA DIODE`

---

### VMware Configuration

#### Virtual Network Editor Settings

> All lab VMnets are **Host-only** with **VMware DHCP disabled**. pfSense provides DHCP per zone.

| VMnet | Label | Type | Subnet | Mask | DHCP |
|-------|-------|------|--------|------|------|
| VMnet1 | (default) | Host-only | 192.168.42.0 | /24 | ON — leave alone |
| VMnet2 | LAN | Host-only | `10.10.0.0` | /24 | **OFF** |
| VMnet3 | DMZ | Host-only | `10.20.0.0` | /24 | **OFF** |
| VMnet4 | BRANCH | Host-only | `10.30.0.0` | /24 | **OFF** |
| VMnet5 | SOC | Host-only | `10.40.0.0` | /24 | **OFF** |
| VMnet6 | MGMT | Host-only | `10.50.0.0` | /24 | **OFF** |
| VMnet7 | ATTCK | Host-only | `10.60.0.0` | /24 | **OFF** |
| VMnet8 | WAN (NAT) | NAT | 192.168.139.0 | /24 | ON — VMware manages |

#### pfSense VM Hardware

| Component | Value | Reason |
|-----------|-------|--------|
| Memory | 1024 MB | Minimum for stable multi-interface operation |
| Processors | 2 vCPUs | Required for Suricata IDS in Phase 2 |
| Hard Disk | 20 GB SCSI | Sufficient for pfSense + logs + packages |
| Network Adapter 1 | NAT (VMnet8) — WAN | em0 — Internet facing |
| Network Adapter 2 | Custom: VMnet2 — LAN | em1 — Corporate Core |
| Network Adapter 3 | Custom: VMnet3 — DMZ | em2 — DMZ Zone |
| Network Adapter 4 | Custom: VMnet4 — BRANCH | em3 — Branch Office |
| Network Adapter 5 | Custom: VMnet5 — SOC | em4 — Security Operations |
| Network Adapter 6 | Custom: VMnet6 — MGMT | em5 — Management Network |
| Network Adapter 7 | Custom: VMnet7 — ATTACK | em6 — Isolated Attack Lab |

> ⚠️ **Adapter order is critical.** VMware assigns `em0–em6` in the order adapters appear in VM Settings. Inserting adapters in the wrong position causes interface mismatches.

---

### pfSense Configuration

#### System General Settings

| Setting | Value |
|---------|-------|
| Hostname | `pfSense` |
| Domain | `lab.local` |
| Primary DNS | `8.8.8.8` |
| Secondary DNS | `8.8.4.4` |
| DNS Server Override | Enabled |
| Timezone | `Asia/Kolkata` |
| Time Server | `2.pfsense.pool.ntp.org` |
| GUI Protocol | HTTPS (port 443) |

#### Interface Assignments

| pfSense Name | em Port | IP Address | Zone |
|-------------|---------|------------|------|
| WAN | em0 | DHCP — `192.168.139.x/24` | Internet / VMware NAT |
| LAN | em1 | `10.10.0.1/24` | Corporate Core |
| DMZ | em2 | `10.20.0.1/24` | DMZ Zone |
| BRANCH | em3 | `10.30.0.1/24` | Branch Office |
| SOC | em4 | `10.40.0.1/24` | Security Operations |
| MGMT | em5 | `10.50.0.1/24` | Management Network |
| ATTACK | em6 | `10.60.0.1/24` | Isolated Attack Lab |

#### DNS Resolver (Unbound)

| Setting | Value |
|---------|-------|
| Enable DNS Resolver | ✅ Enabled |
| Listen Port | 53 |
| Network Interfaces | All |
| Outgoing Network Interfaces | All |
| DNSSEC | ✅ Enabled |
| DNS Query Forwarding | ✅ Enabled → forwards to 8.8.8.8 / 8.8.4.4 |
| DHCP Registration | ✅ Enabled — VMs resolve as `hostname.lab.local` |
| Static DHCP Registration | ✅ Enabled |

---

### Firewall Aliases

> Aliases are named network objects used in firewall rules. This makes rules readable, maintainable, and self-documenting.

| Alias | Type | Value | Purpose |
|-------|------|-------|---------|
| `RFC1918` | Network | `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` | All private IP space — used inverted (`!RFC1918`) to mean "internet only" |
| `LAB_LAN` | Network | `10.10.0.0/24` | Corporate Core |
| `LAB_DMZ` | Network | `10.20.0.0/24` | DMZ Zone |
| `LAB_BRANCH` | Network | `10.30.0.0/24` | Branch Office |
| `LAB_SOC` | Network | `10.40.0.0/24` | SOC Zone |
| `LAB_MGMT` | Network | `10.50.0.0/24` | Management Zone |
| `LAB_ATTACK` | Network | `10.60.0.0/24` | Isolated Attack Lab |
| `ALL_INTERNAL` | Network | All 6 lab subnets combined | Used in broad block rules |

---

### Firewall Rules

> Rules are processed **top to bottom — first match wins**. Explicit block rules generate log entries visible to Wazuh SIEM. All interfaces follow **implicit deny** — anything not explicitly allowed is dropped.

#### WAN

| # | Action | Source | Destination | Description |
|---|--------|--------|-------------|-------------|
| 1 | 🔴 BLOCK | RFC1918 networks | Any | Block spoofed private IPs — ingress filtering |
| 2 | 🔴 BLOCK | Bogon / unallocated IPs | Any | Block IANA-unassigned space — auto-managed |

#### LAN (Corporate Core)

| # | Action | Source | Destination | Port | Description |
|---|--------|--------|-------------|------|-------------|
| 1 | 🟢 PASS | Any | LAN Address | 443/80 | Anti-Lockout Rule — pfSense auto-created |
| 2 | 🟢 PASS | LAN subnets | `!RFC1918` | Any | LAN to Internet only |
| 3 | 🟢 PASS | LAN subnets | `LAB_DMZ` | Any | LAN to DMZ |
| 4 | 🟢 PASS | LAN subnets | `LAB_SOC` | Any | LAN to SOC |
| 5 | 🟢 PASS | LAN subnets | `LAB_BRANCH` | Any | LAN to Branch |
| 6 | 🔴 BLOCK | LAN subnets | `LAB_ATTACK` | Any | LAN blocked from Attack Lab |
| 7 | 🔴 BLOCK | LAN subnets | `LAB_MGMT` | Any | LAN blocked from Management — least privilege |
| 8 | 🟢 PASS | LAN subnets | This Firewall | 443 | LAN access to pfSense GUI |
| 9 | 🟢 PASS | LAN subnets | This Firewall | 53 TCP/UDP | LAN DNS queries to pfSense resolver |

#### DMZ

| # | Action | Source | Destination | Port | Description |
|---|--------|--------|-------------|------|-------------|
| 1 | 🟢 PASS | DMZ subnets | `!RFC1918` | Any | DMZ to Internet — server updates |
| 2 | 🟢 PASS | DMZ subnets | `LAB_SOC` | Any | DMZ to SOC — log shipping to Wazuh |
| 3 | 🔴 BLOCK | DMZ subnets | `ALL_INTERNAL` | Any | DMZ blocked from all internal zones |

#### Branch Office

| # | Action | Source | Destination | Port | Description |
|---|--------|--------|-------------|------|-------------|
| 1 | 🟢 PASS | BRANCH subnets | `!RFC1918` | Any | Branch to Internet |
| 2 | 🟢 PASS | BRANCH subnets | `LAB_LAN` | Any | Branch to HQ resources |
| 3 | 🟢 PASS | BRANCH subnets | `LAB_SOC` | Any | Branch to SOC — Wazuh agent log shipping |
| 4 | 🔴 BLOCK | BRANCH subnets | `LAB_MGMT` | Any | Branch blocked from Management |
| 5 | 🔴 BLOCK | BRANCH subnets | `LAB_ATTACK` | Any | Branch blocked from Attack Lab |
| 6 | 🔴 BLOCK | BRANCH subnets | `LAB_DMZ` | Any | Branch blocked from DMZ |

#### SOC

| # | Action | Source | Destination | Port | Description |
|---|--------|--------|-------------|------|-------------|
| 1 | 🟢 PASS | SOC subnets | `!RFC1918` | Any | SOC to Internet — threat intel feeds (MISP) |
| 2 | 🟢 PASS | SOC subnets | `LAB_LAN` | Any | SOC to LAN — log collection |
| 3 | 🟢 PASS | SOC subnets | `LAB_DMZ` | Any | SOC to DMZ — monitor public-facing services |
| 4 | 🟢 PASS | SOC subnets | `LAB_BRANCH` | Any | SOC to Branch — monitor remote office |
| 5 | 🟢 PASS | SOC subnets | `LAB_MGMT` | Any | SOC to MGMT — monitor management infra |
| 6 | 🟢 PASS | SOC subnets | This Firewall | 443 | SOC access to pfSense GUI — firewall log review |
| 7 | 🔴 BLOCK | SOC subnets | `LAB_ATTACK` | Any | SOC blocked from Attack Lab |

#### Management (MGMT)

> ⚠️ Rule 2 (ATTACK block) must appear **above** Rule 3 (ALL_INTERNAL allow). pfSense reads top-to-bottom — if ALL_INTERNAL matched first it would allow ATTACK traffic before the block fires.

| # | Action | Source | Destination | Port | Description |
|---|--------|--------|-------------|------|-------------|
| 1 | 🟢 PASS | MGMT subnets | `!RFC1918` | Any | MGMT to Internet — updates and patches |
| 2 | 🔴 BLOCK | MGMT subnets | `LAB_ATTACK` | Any | MGMT blocked from Attack Lab — above ALL_INTERNAL |
| 3 | 🟢 PASS | MGMT subnets | `ALL_INTERNAL` | Any | MGMT to all zones — full admin access |
| 4 | 🟢 PASS | MGMT subnets | This Firewall | 443 | MGMT to pfSense GUI — primary admin interface |
| 5 | 🟢 PASS | MGMT subnets | This Firewall | 22 | MGMT to pfSense SSH — emergency console access |

#### Attack Lab (ATTACK)

> 🔒 Strictest policy in the lab. Block comes first — no exceptions for any internal zone.

| # | Action | Source | Destination | Port | Description |
|---|--------|--------|-------------|------|-------------|
| 1 | 🔴 BLOCK | ATTACK subnets | `ALL_INTERNAL` | Any | ATTACK blocked from ALL internal zones — complete isolation |
| 2 | 🟢 PASS | ATTACK subnets | `!RFC1918` | Any | ATTACK to Internet — tool updates only |

---

### Validation

All tests performed from Kali Linux on the LAN zone (`10.10.0.x`).

| Test | Command | Result | Confirms |
|------|---------|--------|----------|
| Internet connectivity | `ping 8.8.8.8` | ✅ PASS | WAN routing and NAT working |
| DNS resolution | `ping google.com` | ✅ PASS | DNS resolver and port 53 rule working |
| LAN to DMZ gateway | `ping 10.20.0.1` | ✅ PASS | LAN → DMZ firewall rule working |
| LAN blocked from ATTACK | `ping 10.60.0.1` | ✅ BLOCKED | Attack Lab isolation enforced |
| pfSense GUI access | `https://10.10.0.1` | ✅ PASS | GUI accessible from LAN on HTTPS |
| Direct DNS query | `nslookup google.com 10.10.0.1` | ✅ PASS | Unbound responding on all interfaces |

---

## 💼 MNC Job Relevance

| Skill Demonstrated | Maps To |
|-------------------|---------|
| Multi-zone network segmentation with pfSense | Network Security Engineer |
| Zero trust firewall policy design | Security Architect |
| Explicit deny rules with logging | SOC Analyst — alert generation |
| RFC1918 + bogon ingress filtering | Perimeter Security Engineer |
| Internal DNS infrastructure | Security / Network Engineer |
| Firewall alias management | Firewall Administrator |
| Per-zone DHCP and IPAM | Network Engineer |
| Policy validation and testing | Any security role |

---

## 🔄 Coming in Phase 2

**Detection & Monitoring — SOC Toolchain**

| Tool | Role | Zone |
|------|------|------|
| **Wazuh SIEM** | Primary SOC platform — log collection, alerts, dashboards | SOC (10.40.0.0/24) |
| **ELK Stack** | Log storage, Elasticsearch indexing, Kibana dashboards | SOC |
| **Suricata IDS** | Network intrusion detection — signature-based alerts | LAN / pfSense inline |
| **OpenVAS** | Vulnerability scanner — CVE identification | SOC |

After Phase 2 you will have a working SOC environment demonstrable in interviews — live alerts, log correlation, and real-time intrusion detection.

---

## 📁 Repository Structure

```
homelab/
├── README.md                    ← You are here (Phase 1)
├── phase1-network-foundation/
│   ├── README.md                ← This document
│   ├── network-diagram.png
│   └── configs/
│       └── pfsense-aliases.xml
├── phase2-soc-monitoring/       ← Coming soon
├── phase3-identity-access/      ← Coming soon
├── phase4-attack-defend/        ← Coming soon
├── phase5-devsecops/            ← Coming soon
└── phase6-cloud-security/       ← Coming soon
```

---

## 📜 License

This project is for educational purposes. All tools used are free and open source.

---

**Built with 🔒 for hands-on enterprise security experience**

*Every component maps to a real MNC job requirement*
