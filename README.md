# 🏢 Enterprise Network Security Lab

<div align="center">

![FortiGate](https://img.shields.io/badge/FortiGate-v7.0.9-red?style=for-the-badge&logo=fortinet)
![EVE-NG](https://img.shields.io/badge/EVE--NG-Community%206.2.0-blue?style=for-the-badge)
![Cisco](https://img.shields.io/badge/Cisco-vIOS%2015.9-blue?style=for-the-badge&logo=cisco)
![VirtualBox](https://img.shields.io/badge/VirtualBox-6.x-orange?style=for-the-badge&logo=virtualbox)
![Status](https://img.shields.io/badge/Status-Completed-green?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)

**A complete enterprise-grade network security infrastructure simulation built on EVE-NG**

*FortiGate HA Cluster · IPsec Site-to-Site VPN · SSL VPN · IPS/IDS · VLAN Segmentation · SNMP Monitoring*

[Features](#-features) · [Architecture](#-architecture) · [Technologies](#-technologies) · [Lab Setup](#-lab-setup) · [Configuration](#-configuration) · [Testing](#-testing-validation) · [Results](#-results)

</div>

---

## 📖 Overview

This project simulates a **production-ready enterprise network infrastructure** using industry-standard tools and technologies. The lab demonstrates real-world network engineering and security concepts including firewall high availability, encrypted tunnels, VLAN segmentation, intrusion prevention, and infrastructure monitoring.



---

## ✨ Features

| Feature | Technology | Status |
|---------|-----------|--------|
| Network Simulation | EVE-NG Community 6.2.0-4 | ✅ |
| Next-Gen Firewall | FortiGate v7.0.9 | ✅ |
| High Availability | FortiGate Active-Passive HA | ✅ |
| WAN Routing | Cisco vIOS Router (R3-Siege) | ✅ |
| VLAN Segmentation | 6 VLANs (Users, Servers, Guest, DMZ, WAN) | ✅ |
| DHCP Services | FortiGate DHCP per VLAN | ✅ |
| IPsec VPN | Site-to-Site (2 remote sites) | ✅ |
| SSL VPN | Web Mode + Tunnel Mode | ✅ |
| Intrusion Prevention | IPS-Lab Profile (32 signatures) | ✅ |
| Web Filtering | WebFilter-Lab Profile | ⚠️ |
| SNMP Monitoring | SNMPv2c Community | ✅ |
| Firewall Policies | 9 rules with NAT & IPS | ✅ |
| Attack/Defense Scenarios | Port scan, VLAN isolation tests | ✅ |

> ⚠️ Web Filtering requires active FortiGuard license. Configuration documented but FortiGuard categories unavailable in evaluation environment.

---

## 🗺️ Architecture

### Network Topology

```
                        ┌─────────────┐
                        │   Internet  │
                        └──────┬──────┘
                               │
                        ┌──────┴──────┐
                        │  R3-Siege   │  WAN Router
                        │ 10.0.0.1/30 │
                        │ 10.0.0.5/30 │
                        └──┬──────┬───┘
                           │      │
              WAN1          │      │         WAN2
         10.0.0.0/30       │      │    10.0.0.4/30
                    ┌──────┘      └──────┐
                    │                    │
             ┌──────┴──────┐    ┌────────┴──────┐
             │ FGT1-Master │◄──►│  FGT2-Backup  │
             │  Priority:  │ HA │   Priority:   │
             │     200     │    │      100      │
             └──────┬──────┘    └───────┬───────┘
                    │   Trunk dot1q      │
                    └────────┬───────────┘
                             │
                      ┌──────┴──────┐
                      │   SW-Core   │  Cisco vIOS-L2
                      │  L2 Switch  │
                      └──┬──┬──┬──┬─┘
                         │  │  │  │
               ┌─────────┘  │  │  └─────────┐
               │            │  │             │
          VLAN10        VLAN30  VLAN50    VLAN100/200
          Users         Guest    DMZ       WAN Sites
        192.168.10     .30.0   .50.0    ──► R4, R5
          PC7,PC8      PC10     PC9
```

### Physical Connections

| Device | Interface | Connected To | Interface |
|--------|-----------|--------------|-----------|
| R3 | Gi0/0 | FGT1 | port1 |
| R3 | Gi0/1 | FGT2 | port1 |
| R3 | Gi0/2 | SW | Gi2/0 (VLAN100) |
| R3 | Gi0/3 | SW | Gi2/1 (VLAN200) |
| FGT1 | port2 | SW | Gi0/0 (trunk) |
| FGT2 | port3 | SW | Gi0/1 (trunk) |
| FGT1 | port4 | FGT2 | port4 (HA) |
| FGT1 | port3 | Cloud0 | pnet0 (Mgmt) |
| SW | Gi0/2 | R4 | Gi0/0 (VLAN100) |
| SW | Gi0/3 | R5 | Gi0/0 (VLAN200) |
| SW | Gi1/0 | PC7 | eth0 (VLAN10) |
| SW | Gi1/1 | PC8 | eth0 (VLAN10) |
| SW | Gi1/2 | PC9 | eth0 (VLAN50) |
| SW | Gi1/3 | PC10 | eth0 (VLAN30) |

---

## 🛠️ Technologies

### Hardware (Virtualized)

| Device | Model | Role |
|--------|-------|------|
| FGT1 | FortiGate-VM64-KVM | Primary Firewall (Active) |
| FGT2 | FortiGate-VM64-KVM | Backup Firewall (Passive) |
| R3 | Cisco vIOS 15.9-3.M6 | WAN Router / Siege |
| R4 | Cisco vIOS 15.9-3.M6 | Remote Site A Router |
| R5 | Cisco vIOS 15.9-3.M6 | Remote Site B Router |
| SW | Cisco vIOS-L2 | Core L2 Switch |
| PC7-PC10 | VPCS | End-user Clients |

### Software Stack

| Tool | Version | Purpose |
|------|---------|---------|
| EVE-NG Community | 6.2.0-4 | Network Emulation Platform |
| VirtualBox | 6.x | Hypervisor (Windows Host) |
| FortiOS | v7.0.9 build0444 | Firewall OS |
| PuTTY | Latest | SSH/Console Access |
| Nmap | 7.95 | Network Scanning Tests |

---

## 📡 IP Addressing Plan

### WAN Links

| Link | Network | R3 IP | FGT IP |
|------|---------|-------|--------|
| WAN1 (FGT1) | 10.0.0.0/30 | 10.0.0.1 | 10.0.0.2 |
| WAN2 (FGT2) | 10.0.0.4/30 | 10.0.0.5 | 10.0.0.6 |

### Inter-Site Links

| Link | Network | R3 IP | Router IP |
|------|---------|-------|-----------|
| To Site A (R4) | 10.1.0.0/24 | 10.1.0.1 | 10.1.0.2 |
| To Site B (R5) | 10.2.0.0/24 | 10.2.0.1 | 10.2.0.2 |

### VLANs — Headquarters

| VLAN | Name | Network | Gateway | DHCP Pool |
|------|------|---------|---------|-----------|
| 10 | Users | 192.168.10.0/24 | 192.168.10.1 | .100 – .200 |
| 20 | Servers | 192.168.20.0/24 | 192.168.20.1 | .100 – .200 |
| 30 | Guest | 192.168.30.0/24 | 192.168.30.1 | .100 – .200 |
| 50 | DMZ | 192.168.50.0/24 | 192.168.50.1 | .100 – .200 |
| 100 | WAN-SiteA | 10.1.0.0/24 | — | — |
| 200 | WAN-SiteB | 10.2.0.0/24 | — | — |

### Remote Sites LANs

| Site | Network | Gateway |
|------|---------|---------|
| Site A (R4) | 172.16.10.0/24 | 172.16.10.1 |
| Site B (R5) | 172.16.20.0/24 | 172.16.20.1 |

### VPN Pools

| Type | Range |
|------|-------|
| SSL VPN Pool | 10.10.10.10 – 10.10.10.100 |
| Management | 192.168.56.200/24 |

---

## 🔧 Lab Setup

### Prerequisites

- Windows 10/11 host machine (minimum 20GB RAM recommended)
- VirtualBox 6.x or higher
- EVE-NG Community Edition OVA
- FortiGate QEMU image (fortinet-FGT-v7.0.9)
- Cisco vIOS image (vios-adventerprisek9-m.SPA.159-3.M6)
- Cisco vIOS-L2 image (viosl2-adventerprisek9-m.ssa.high_iron_20200929)

### EVE-NG Installation

```bash
# 1. Import EVE-NG OVA into VirtualBox
# Network Adapter 1: NAT
# Network Adapter 2: Host-Only (Promiscuous Mode: Allow All)  ← CRITICAL

# 2. Access EVE-NG
Web UI:  http://192.168.56.110  (admin/eve)
SSH:     ssh root@192.168.56.110  (root/eve)
```

### Image Installation

```bash
# SSH into EVE-NG
ssh root@192.168.56.110

# Upload and install FortiGate image
cd /opt/unetlab/addons/qemu/
mkdir fortinet-FGT-v7.0.9
# Upload virtioa.qcow2 to this directory

# Fix permissions
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
```

### Important VirtualBox Configuration

> ⚠️ **Critical**: Set Promiscuous Mode to **Allow All** on the Host-Only adapter of the EVE-NG VM. Without this, management traffic to FortiGate (192.168.56.200) will not reach the host.

---

## ⚙️ Configuration

### FortiGate HA Configuration

**FGT1-Master:**
```
config system ha
    set group-name "FGT-HA"
    set mode a-p
    set password "ha-secret"
    set priority 200
    set hbdev "port4" 50
end
```

**FGT2-Backup:**
```
config system ha
    set group-name "FGT-HA"
    set mode a-p
    set password "ha-secret"
    set priority 100
    set hbdev "port4" 50
end
```

### IPsec VPN — Site A (FGT1 side)

```
config vpn ipsec phase1-interface
    edit "VPN-SiteA"
        set interface "port1"
        set ike-version 1
        set peertype any
        set proposal des-sha256
        set dhgrp 14
        set remote-gw 10.1.0.2
        set psksecret "VPN-SiteA-Secret123"
    next
end
```

### SSL VPN Configuration

```
config vpn ssl settings
    set servercert "Fortinet_Factory"
    set tunnel-ip-pools "SSLVPN_TUNNEL_ADDR1"
    set source-interface "port3"
    set source-address "all"
    set default-portal "full-access"
    set port 10443
    set port-precedence disable
end
```

### IPS Profile

```
config ips sensor
    edit "IPS-Lab"
        set comment "IPS profile for enterprise lab"
        config entries
            edit 1
                set rule SCAN
                set action block
                set log enable
                set packet-log enable
            next
        end
        set block-malicious-url enable
    next
end
```

### SNMP Configuration

```
config system snmp sysinfo
    set status enable
    set description "FGT1-Master Enterprise Lab"
    set contact-info "admin@lab.local"
    set location "EVE-NG Lab"
end

config system snmp community
    edit 1
        set name "LabSNMP"
        set status enable
        config hosts
            edit 1
                set ip 192.168.56.0 255.255.255.0
            next
        end
        set events cpu-high mem-low log-full intf-ip vpn-tun-up vpn-tun-down
    next
end
```

### Firewall Policies Summary

| ID | Name | Source | Destination | Service | Action | NAT | IPS |
|----|------|--------|-------------|---------|--------|-----|-----|
| 1 | Users-to-Internet | vlan10 | port1 | ALL | ACCEPT | ✅ | IPS-Lab |
| 2 | Users-to-DMZ | vlan10 | vlan50 | HTTP/HTTPS/PING | ACCEPT | ❌ | IPS-Lab |
| 3 | DMZ-block-LAN | vlan50 | vlan10 | ALL | DENY | ❌ | ❌ |
| 4 | Guest-Internet-only | vlan30 | port1 | ALL | ACCEPT | ✅ | ❌ |
| 5 | LAN-to-SiteA | vlan10 | VPN-SiteA | ALL | ACCEPT | ❌ | ❌ |
| 6 | SiteA-to-LAN | VPN-SiteA | vlan10 | ALL | ACCEPT | ❌ | ❌ |
| 7 | LAN-to-SiteB | vlan10 | VPN-SiteB | ALL | ACCEPT | ❌ | ❌ |
| 8 | SiteB-to-LAN | VPN-SiteB | vlan10 | ALL | ACCEPT | ❌ | ❌ |
| 9 | SSLVPN-to-LAN | ssl.root | vlan10 | ALL | ACCEPT | ❌ | IPS-Lab |

---

## 🧪 Testing & Validation

### Connectivity Tests

```bash
# Test 1 — DHCP Verification (from PC7 console in EVE-NG)
VPCS> dhcp
DDORA IP 192.168.10.100/24 GW 192.168.10.1  ✅

# Test 2 — Gateway Ping
VPCS> ping 192.168.10.1
84 bytes from 192.168.10.1 icmp_seq=1 ttl=255 time=14.532 ms  ✅

# Test 3 — IPsec VPN to Site A
VPCS> ping 172.16.10.1
84 bytes from 172.16.10.1 icmp_seq=1 ttl=253 time=25.1 ms  ✅

# Test 4 — IPsec VPN to Site B
VPCS> ping 172.16.20.1
84 bytes from 172.16.20.1 icmp_seq=1 ttl=253 time=22.4 ms  ✅
```

### Security Tests

```bash
# Test 5 — VLAN Isolation (PC7 → PC10, should be BLOCKED)
VPCS> ping 192.168.30.100
192.168.30.100 icmp_seq=1 timeout  ✅ BLOCKED

# Test 6 — DMZ to LAN (PC9 → PC7, should be BLOCKED)
VPCS> ping 192.168.10.100
192.168.10.100 icmp_seq=1 timeout  ✅ BLOCKED

# Test 7 — Port Scan (from Windows host via nmap)
nmap -sS -Pn -p 1-1000 192.168.56.200
PORT    STATE  SERVICE
22/tcp  open   ssh
80/tcp  open   http
443/tcp open   https  ✅

# Test 8 — IPS Port Filtering
nmap -sT -Pn --max-rtt-timeout 100ms -p 22,23,80,443 192.168.10.100
PORT    STATE    SERVICE
22/tcp  filtered ssh
80/tcp  filtered http    ✅ Filtered by IPS-Lab

# Test 9 — SNMP (from Windows host)
nmap -sU -p 161 --script snmp-info --script-args snmpcommunity=LabSNMP 192.168.56.200
161/udp open|filtered snmp  ✅
```

### HA Failover Test

```bash
# Verify HA Status on FGT1
FGT1-Master # get system ha status
Model: FortiGate-VM64-KVM
Mode: a-p
Group: 0
Debug: 0
ses_pickup: disable, ses_pickup_delay=disable
Master:
  FGT1-Master(0) HA cluster index = 0, priority = 200, override = disable
Slave:
  FGT2-Backup(1) HA cluster index = 1, priority = 100, override = disable
number of vcluster: 1
vcluster 1: work 192.168.56.200
```

### SSL VPN Access

```bash
# Create SSH tunnel (Windows CMD)
ssh -L 8443:192.168.56.200:10443 root@192.168.56.110

# Access SSL VPN portal
https://localhost:8443
# Login: vpnuser / VPNuser123!  ✅
```

---

## 📊 Results

### Test Summary

| Test | Description | Expected | Result |
|------|-------------|----------|--------|
| 01 | DHCP VLAN10 PC7 | 192.168.10.100 | ✅ |
| 02 | DHCP VLAN10 PC8 | 192.168.10.101 | ✅ |
| 03 | DHCP VLAN30 PC10 | 192.168.30.100 | ✅ |
| 04 | DHCP VLAN50 PC9 | 192.168.50.100 | ✅ |
| 05 | PC7 → Gateway ping | OK | ✅ |
| 06 | PC7 → PC10 VLAN isolation | BLOCKED | ✅ |
| 07 | PC7 → PC9 DMZ access | ALLOWED | ✅ |
| 08 | PC9 → PC7 DMZ→LAN block | BLOCKED | ✅ |
| 09 | FortiGate HA sync | In-Sync | ✅ |
| 10 | R3 → FGT1 ping | OK | ✅ |
| 11 | IPsec VPN Site A | Tunnel active | ✅ |
| 12 | IPsec VPN Site B | Tunnel active | ✅ |
| 13 | PC7 → Site A (172.16.10.1) | OK via VPN | ✅ |
| 14 | PC7 → Site B (172.16.20.1) | OK via VPN | ✅ |
| 15 | SSL VPN Web Mode | Portal accessible | ✅ |
| 16 | IPS Port Scan detection | Ports filtered | ✅ |
| 17 | SNMP port 161 | Open | ✅ |

### Performance Metrics

| Metric | Value |
|--------|-------|
| FortiGate vCPUs | 1/1 (100%) |
| Allocated RAM | 2GB/2GB (98%) |
| Firmware | v7.0.9 build0444 (Mature) |
| HA Sync Status | In-Sync ✅ |
| Active IPsec Tunnels | 2 (SiteA + SiteB) |
| Firewall Policies | 9 active rules |
| IPS Signatures | 32 (SCAN category) |

---


---

## 🔑 Credentials Reference

> ⚠️ These credentials are for **lab/educational purposes only**. Never use these in production.

| Device | Username | Password |
|--------|----------|----------|
| EVE-NG SSH | root | eve |
| EVE-NG Web | admin | eve |
| FGT1-Master | admin | Admin1234! |
| FGT2-Backup | admin | Admin1234! |
| SSL VPN User | vpnuser | VPNuser123! |

---

## 🚧 Known Limitations

1. **Web Filtering** — FortiGuard category-based filtering requires an active FortiGuard subscription. In this lab, the evaluation license expired after 15 days, limiting FortiGuard cloud services.

2. **SSL VPN Browser Access** — FortiGate v7.0.9 KVM uses older TLS/DH parameters incompatible with modern browsers (Chrome, Firefox, Edge). Workaround: SSH tunnel (`ssh -L 8443:192.168.56.200:10443 root@192.168.56.110`).

3. **VPCS Limitations** — VPCS clients cannot initiate inter-VLAN traffic tests directly in some scenarios. Tests validated via FortiGate CLI pings.

4. **FortiClient Compatibility** — FortiClient 7.4.x is not compatible with FortiGate 7.0.x for SSL VPN tunnel mode. Web mode via SSH tunnel was used as an alternative.

---

## 🎯 Skills Demonstrated

- ✅ **Network Design** — Enterprise topology with redundancy
- ✅ **Firewall Administration** — FortiGate NGFW configuration
- ✅ **High Availability** — Active-Passive HA cluster setup
- ✅ **VPN Engineering** — IPsec site-to-site + SSL remote access
- ✅ **Network Segmentation** — VLAN design and inter-VLAN routing
- ✅ **Security Policies** — Stateful firewall rules with UTM profiles
- ✅ **Intrusion Prevention** — IPS signature-based detection/blocking
- ✅ **Network Monitoring** — SNMPv2c configuration and testing
- ✅ **Troubleshooting** — Systematic diagnosis with packet captures
- ✅ **Documentation** — Technical reports, diagrams, and GitHub

---

## 📚 References

- [FortiGate Administration Guide v7.0](https://docs.fortinet.com/product/fortigate/7.0)
- [EVE-NG Documentation](https://www.eve-ng.net/index.php/documentation/)
- [Cisco IOS Configuration Guide](https://www.cisco.com/c/en/us/support/routers/index.html)
- [RFC 4301 — IPsec Architecture](https://www.rfc-editor.org/rfc/rfc4301)
- [RFC 6071 — IPsec and IKE Documents](https://www.rfc-editor.org/rfc/rfc6071)

---

## 👤 Author

<div align="center">

**Ayoub Lahlaibi**

Master's Student — IT Security & Big Data (SIT & Big Data)
Faculté des Sciences et Techniques de Tanger
Université Abdelmalek Essaâdi, Morocco

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?style=for-the-badge&logo=linkedin)](https://linkedin.com/in/ayoub-lahlaibi)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-black?style=for-the-badge&logo=github)](https://github.com/ayoub-lahlaibi)

</div>

---


```

```

---

<div align="center">

⭐ **If this project was helpful, please consider giving it a star!** ⭐

*Built with ❤️ for learning and portfolio purposes*

</div>
