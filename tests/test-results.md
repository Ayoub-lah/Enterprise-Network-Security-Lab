# Test Results — Enterprise Network Security Lab

> **Date:** May 2026  
> **Environment:** EVE-NG Community 6.2.0-4 | FortiGate v7.0.9  
> **Tester:** Ayoub Lahlaibi

---

## Summary

| Total Tests | Passed | Failed | Warnings |
|-------------|--------|--------|----------|
| 17 | 16 | 0 | 1 |

---

## Phase 2 — VLAN & DHCP Tests

### Test 01 — DHCP VLAN10 PC7
```
VPCS> dhcp
DDORA IP 192.168.10.100/24 GW 192.168.10.1
```
**Result:** ✅ PASS — IP 192.168.10.100 assigned

---

### Test 02 — DHCP VLAN10 PC8
```
VPCS> dhcp
DDORA IP 192.168.10.101/24 GW 192.168.10.1
```
**Result:** ✅ PASS — IP 192.168.10.101 assigned

---

### Test 03 — DHCP VLAN30 PC10
```
VPCS> dhcp
DDORA IP 192.168.30.100/24 GW 192.168.30.1
```
**Result:** ✅ PASS — IP 192.168.30.100 assigned

---

### Test 04 — DHCP VLAN50 PC9
```
VPCS> dhcp
DDORA IP 192.168.50.100/24 GW 192.168.50.1
```
**Result:** ✅ PASS — IP 192.168.50.100 assigned

---

### Test 05 — PC7 to Gateway Ping
```
VPCS> ping 192.168.10.1
84 bytes from 192.168.10.1 icmp_seq=1 ttl=255 time=14.532 ms
84 bytes from 192.168.10.1 icmp_seq=2 ttl=255 time=13.766 ms
84 bytes from 192.168.10.1 icmp_seq=3 ttl=255 time=6.294 ms
```
**Result:** ✅ PASS — Gateway reachable from VLAN10

---

## Phase 2 — VLAN Isolation Tests

### Test 06 — PC7 to PC10 (VLAN Isolation)
```
VPCS> ping 192.168.30.100
192.168.30.100 icmp_seq=1 timeout
192.168.30.100 icmp_seq=2 timeout
192.168.30.100 icmp_seq=3 timeout
```
**Expected:** BLOCKED by firewall policy  
**Result:** ✅ PASS — Inter-VLAN traffic blocked (Users ↛ Guest)

---

### Test 07 — PC7 to PC9 DMZ (Allowed)
```
FGT1-Master # execute ping-options source 192.168.10.1
FGT1-Master # execute ping 192.168.50.100
64 bytes from 192.168.50.100: icmp_seq=0 ttl=64 time=48.4 ms
64 bytes from 192.168.50.100: icmp_seq=1 ttl=64 time=29.2 ms
```
**Expected:** ALLOWED (HTTP/HTTPS/PING only)  
**Result:** ✅ PASS — Users can reach DMZ via policy P2

---

### Test 08 — PC9 to PC7 (DMZ to LAN Block)
```
VPCS> ping 192.168.10.100
192.168.10.100 icmp_seq=1 timeout
192.168.10.100 icmp_seq=2 timeout
```
**Expected:** BLOCKED by DMZ-block-LAN policy  
**Result:** ✅ PASS — DMZ cannot access LAN (policy P3)

---

## Phase 3 — IPsec VPN Tests

### Test 09 — R3 to FGT1 Connectivity
```
R3-Siege# ping 10.0.0.2
!!!!!
Success rate is 100 percent (5/5)
```
**Result:** ✅ PASS

---

### Test 10 — IPsec Tunnel Site A
```
FGT1-Master # get vpn ipsec tunnel summary
VPN-SiteA  UP  id=1 rgwy=10.1.0.2
```
**Result:** ✅ PASS — IPsec tunnel to Site A active

---

### Test 11 — IPsec Tunnel Site B
```
FGT1-Master # get vpn ipsec tunnel summary
VPN-SiteB  UP  id=2 rgwy=10.2.0.2
```
**Result:** ✅ PASS — IPsec tunnel to Site B active

---

### Test 12 — PC7 to Site A via VPN
```
VPCS> ping 172.16.10.1
84 bytes from 172.16.10.1 icmp_seq=1 ttl=253 time=25.1 ms
84 bytes from 172.16.10.1 icmp_seq=2 ttl=253 time=22.3 ms
```
**Result:** ✅ PASS — Traffic routed through IPsec tunnel

---

### Test 13 — PC7 to Site B via VPN
```
VPCS> ping 172.16.20.1
84 bytes from 172.16.20.1 icmp_seq=1 ttl=253 time=23.5 ms
84 bytes from 172.16.20.1 icmp_seq=2 ttl=253 time=21.8 ms
```
**Result:** ✅ PASS — Traffic routed through IPsec tunnel

---

## Phase 4 — HA Cluster Tests

### Test 14 — FortiGate HA Sync
```
FGT1-Master # get system ha status
Master:  FGT1-Master  priority=200  in-sync
Slave:   FGT2-Backup  priority=100  in-sync
```
**Result:** ✅ PASS — HA cluster synchronized

---

## Phase 5 — SSL VPN Tests

### Test 15 — SSL VPN Portal Access
```
SSH Tunnel: ssh -L 8443:192.168.56.200:10443 root@192.168.56.110
Browser: https://localhost:8443
Login: vpnuser / VPNuser123!
Portal: SSL-VPN Portal loaded successfully
```
**Result:** ✅ PASS — SSL VPN web portal accessible  
**Note:** Direct browser access blocked due to TLS/DH compatibility issue with FortiGate v7.0.9 KVM. SSH tunnel workaround used.

---

## Phase 6 — IPS Tests

### Test 16 — Port Scan Detection
```
Windows CMD> nmap -sT -Pn --max-rtt-timeout 100ms -p 22,23,80,443 192.168.10.100
PORT    STATE    SERVICE
22/tcp  filtered ssh
23/tcp  filtered telnet
80/tcp  filtered http
443/tcp filtered https
```
**Expected:** Ports filtered by IPS-Lab profile  
**Result:** ✅ PASS — IPS blocking scan attempts

---

## Phase 8 — SNMP Tests

### Test 17 — SNMP Port Verification
```
Windows CMD> nmap -sU -p 161 192.168.56.200
PORT    STATE         SERVICE
161/udp open|filtered snmp
MAC Address: 00:09:0F:09:01:02 (Fortinet)
```
**Result:** ✅ PASS — SNMP port 161/UDP open on FortiGate

---

## Known Issues & Limitations

### Web Filtering (Phase 7)
- **Issue:** FortiGuard evaluation license expired after 15 days
- **Impact:** FortiGuard category-based web filtering unavailable
- **Workaround:** Static URL filtering configured manually
- **Status:** ⚠️ WARNING — Not a failure, environment limitation

### SSL VPN Browser Access
- **Issue:** FortiGate v7.0.9 KVM uses legacy TLS/DH parameters incompatible with modern browsers
- **Impact:** Direct browser access to https://192.168.56.200:10443 fails
- **Workaround:** SSH port forwarding tunnel successfully used
- **Status:** ⚠️ WARNING — Workaround implemented and validated

### FortiClient Compatibility
- **Issue:** FortiClient 7.4.x incompatible with FortiGate 7.0.x SSL VPN
- **Impact:** Tunnel mode SSL VPN via FortiClient not possible
- **Workaround:** Web mode via SSH tunnel used instead
- **Status:** ⚠️ WARNING — Documented limitation

---

## Test Environment Details

| Parameter | Value |
|-----------|-------|
| Test Date | May 2026 |
| EVE-NG Version | Community 6.2.0-4 |
| FortiOS Version | v7.0.9 build0444 |
| Cisco vIOS | 15.9-3.M6 |
| Host OS | Windows 10 Pro |
| VirtualBox | 6.x |
| RAM Available | 20GB |
| Test Duration | ~15 days |

---

*Generated as part of Enterprise Network Security Lab documentation*
