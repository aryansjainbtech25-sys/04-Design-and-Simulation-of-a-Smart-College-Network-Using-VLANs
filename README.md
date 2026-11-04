# Design & Simulation of a Smart College Network Using VLANs

## Students

| # | Name | USN |
|---|---|---|
| 1 | Aryan Inchal | 1RUA25CSE0084 |
| 2 | Bellumkonda Sandilya | 1RUA25CSE0108 |
| 3 | Bhuvan Tej U | 1RUA25CSE0119 |
| 4 | Aryan S Jain | 1RUA25CSE0087 |

**Under the guidance of:** Prof. Evlin Vidyu Latha P  
**School of Computer Science and Engineering, RV University, Bangalore**  
**Course:** CS2403 Computer Networks - Mini Project Report

---

## Project Overview
A secure, segmented college campus network using VLANs, Router-on-a-Stick, Core MLS 3560 Inter-VLAN routing, DHCP Relay, WPA2-PSK Wireless, ACLs, NAT/PAT Internet, and real-traffic analysis with Wireshark.

**Two Topologies Built:**

**Topology A - Basic Star (Day 1-5):**
- **Switch:** College Core Switch 2960-24TT (Star Topology)
- **Router:** College Core Router ISR4331 (Router-on-a-Stick)
- **IP Method:** Static IP (No DHCP server)

**Topology B - Enterprise (Day 6+ / Phase 2-11):**
- **Core:** 3560-24PS MLS Core-MLS-3560 (IP Routing, SVIs, DHCP Relay, ACLs)
- **Edge:** ISR4331 Edge-Router (10.0.0.2/30 to Core, 200.1.1.1/30 to ISP)
- **Access:** 6x 2960 - SW-Admin, SW-Student, SW-Faculty, SW-Guest-WiFi, SW-IT-Lab, SW-Medical
- **Servers:** Server-DHCP 192.168.99.10, Server-Web 192.168.99.20, Server-Internet 8.8.8.8
- **ISP:** ISP-Router 200.1.1.2/30 + 8.8.8.1/24
- **Wireless:** AP-Guest Guest-WiFi WPA2-PSK Nopass@123, AP-IT IT-WiFi WPA2-PSK Cisco123!

- **Simulation Tool:** Cisco Packet Tracer 8.x
- **Analysis Tool:** Wireshark 4.x on Linux (HP Pavilion Laptop 16-af0xxx) + Packet Tracer Simulation Mode
- **Host Interface:** `wlo1` (Linux Wireless) via Mobile Hotspot `172.16.222.0/24`
- **IP Method:** Enterprise uses DHCP Pools (6 VLANs) with DNS 192.168.99.20

---

## Day 2: VLAN & IP Addressing Plan (Topology A - Basic)

| VLAN ID | Name | Network | Gateway (Router Subinterface) | Device - IP - Switch Port |
|---|---|---|---|---|
| 10 | STUDENTS | 192.168.10.0/24 | 192.168.10.1 (g0/0/0.10) | Student PC - 192.168.10.10 - Fa0/1 |
| 20 | FACULTY | 192.168.20.0/24 | 192.168.20.1 (g0/0/0.20) | Faculty PC - 192.168.20.10 - Fa0/2 |
| 30 | ADMIN | 192.168.30.0/24 | 192.168.30.1 (g0/0/0.30) | Admin PC - 192.168.30.10 - Fa0/3, Server - 192.168.30.100 - Fa0/5 |
| 40 | GUESTS | 192.168.40.0/24 | 192.168.40.1 (g0/0/0.40) | Guest PC - 192.168.40.10 - Fa0/4 |

Mask: 255.255.255.0 for all. DNS: 8.8.8.8

## Day 3: Trunk & Inter-VLAN Routing (Basic)

- Switch: `interface Gig0/1` -> `switchport mode trunk` -> allowed vlan 10,20,30,40
- Router: Physical `g0/0/0` up, no IP. 4 subinterfaces with `encapsulation dot1Q`

```
interface g0/0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
interface g0/0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
interface g0/0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
interface g0/0/0.40
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.0
```

## Day 4: Connectivity Tests (Basic - Before ACL)

Gateway tests: All 5 devices ping their gateway - PASS
Same-VLAN: Admin 30.10 -> Server 30.100 - PASS (Fixed wiring to Fa0/3, Fa0/5)
Inter-VLAN: Student->Faculty, Student->Admin, Guest->Student etc - All PASS after Router-on-a-Stick

**Simulation Note:** Inter-VLAN PC first ARPs for its Gateway (e.g., Who has 192.168.10.1?), not destination. To see ARP+ICMP, clear cache: `arp -d` on PC + `clear arp-cache` on Router, filter only ARP+ICMP in Simulation Mode.

---

## Day 5 - Task 1: ACL Configuration (Basic - 45 mins)

### Policy
- Students -> College Server (30.100) = Allow
- Students -> Admin network (30.0/24) = Deny (Server host exception)
- Guests -> Internal networks (10,20,30) = Deny
- Faculty -> All = Allow

### Configuration

```
ip access-list extended STUDENT-ACL
 remark Allow Students to College Server only
 permit ip 192.168.10.0 0.0.0.255 host 192.168.30.100
 remark Deny Students to rest of Admin network
 deny ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
 permit ip 192.168.10.0 0.0.0.255 any

ip access-list extended GUEST-ACL
 deny ip 192.168.40.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny ip 192.168.40.0 0.0.0.255 192.168.20.0 0.0.0.255
 deny ip 192.168.40.0 0.0.0.255 192.168.30.0 0.0.0.255
 permit ip 192.168.40.0 0.0.0.255 any

interface g0/0/0.10
 ip access-group STUDENT-ACL in
interface g0/0/0.40
 ip access-group GUEST-ACL in
```

Verification: `show access-lists` + `show ip interface g0/0/0.10 | include access list` + `show ip interface g0/0/0.40 | include access list`

### ACL Test Results (Basic)

| # | Source | Destination | Expected | Actual | Reason |
|---|---|---|---|---|---|
| 1 | Student 10.10 | Server 30.100 | Allow | Reply | Rule 10 permit host |
| 2 | Student 10.10 | Admin PC 30.10 | Deny | Timeout | Rule 20 deny 30.0 |
| 3 | Student 10.10 | Faculty 20.10 | Allow | Reply | Rule 30 permit any |
| 4 | Guest 40.10 | Student 10.10 | Deny | Timeout | GUEST-ACL deny 40->10 |
| 5 | Guest 40.10 | Faculty 20.10 | Deny | Timeout | deny 40->20 |
| 6 | Guest 40.10 | Server 30.100 | Deny | Timeout | deny 40->30 |
| 7 | Faculty 20.10 | Server 30.100 | Allow | Reply | No ACL on Faculty |

**Order Matters:** Host permit must be before network deny.

---

## Day 5 - Task 2: Wireshark Packet Analysis (Basic - 45 mins)

### Limitation
> Wireshark cannot capture Packet Tracer internal simulated traffic. Packet Tracer Simulation Mode is used for virtual packets. Wireshark is used separately for real traffic from host test environment (wlo1). Both show same protocols.

### Real Environment
- OS: Linux HP Pavilion 16-af0xxx, user aryansjain18
- Interface: `wlo1` (Linux Wireless LAN, equivalent to Wi-Fi on Windows)
- Connection: Mobile Hotspot
- Host IP: `172.16.222.235/24` on wlo1 (`ip a | grep wlo1`)
- Gateway (Phone): `172.16.222.1` (`ip r | grep wlo1` shows default via)
- Note: Android 12+ hotspot uses 172.16.x.x, not 192.168.43.x

### Capture Steps
1. Install: `sudo apt install wireshark -y` + `sudo usermod -a -G wireshark $USER` (re-login) or `sudo wireshark`
2. Select `wlo1`, start capture
3. Generate:
   ```
   ping -c 4 172.16.222.1
   ping -c 4 8.8.8.8
   nslookup google.com
   # browse google.com
   ```
4. Stop, Save as `Day5_Wireshark_wlo1_172.16.pcapng`

### Filters

| Filter | Displays | Observation |
|---|---|---|
| `arp` | ARP | Who has 172.16.222.1? Tell 172.16.222.235 + Phone MAC |
| `icmp` | ICMP ping | Type 8 Request to 172.16.222.1, Type 0 Reply. External 8.8.8.8 may be blocked by carrier, so gateway ping used |
| `dns` | DNS | Query A google.com, Response 142.250.x.x |
| `tcp` | TCP | SYN, SYN-ACK, ACK to 142.250.x.x:443 |
| `udp` | UDP | UDP 53 for DNS |

Deliverable: `.pcapng` + screenshots of filters + packet details (Ethernet, IP, ICMP layers expanded)

---

## Enterprise Topology B - Smart College Network (Phase 2 onwards)

### Phase 2.2-2.3: L2/L3 Base - Core MLS 3560

**VLANs:** 10 STUDENTS, 20 FACULTY, 30 ADMIN, 40 GUESTS, 50 IT_LAB, 60 MEDICAL, 99 SERVERS, 1 Native

**IP Plan:**
- Core Gig0/1: 10.0.0.1/30, Edge Gig0/0/0: 10.0.0.2/30
- SVIs: Vlan10 192.168.10.1, Vlan20 192.168.20.1, Vlan30 192.168.30.1, Vlan40 192.168.40.1, Vlan50 192.168.50.1, Vlan60 192.168.60.1, Vlan99 192.168.99.1
- Servers Static: DHCP 192.168.99.10, Web 192.168.99.20, GW 192.168.99.1

**Core Config:**
```
ip routing
vlan 10,20,30,40,50,60,99
interface range fa0/1-6
 switchport trunk encapsulation dot1q
 switchport mode trunk
interface Vlan10 ... Vlan99
 ip address ... + ip helper-address 192.168.99.10 (Vlan10-60)
interface Gig0/1
 no switchport
 ip address 10.0.0.1 255.255.255.252
ip route 0.0.0.0 0.0.0.0 10.0.0.2
```

**Access Switches (All 2960s):**
- SW-Admin Fa0/1-2 VLAN30, Fa0/3-4 VLAN99
- SW-Student Fa0/1-2 VLAN10
- SW-Faculty Fa0/1-2 VLAN20
- SW-Guest-WiFi Fa0/1 VLAN40 (to AP-Guest)
- SW-IT-Lab Fa0/1-3 VLAN50 (to AP-IT + PCs)
- SW-Medical Fa0/1 VLAN60

**Verification:**
```
Core# show interfaces trunk
Fa0/1-6 on 802.1q trunking 1,10,20,30,40,50,60,99 forwarding
Core# show ip route
C 192.168.10.0/24 Vlan10 ... C 192.168.99.0/24 Vlan99
S* 0.0.0.0/0 via 10.0.0.2
```

**Evidence:**
- `results/enterprise/day 1/phase 2.3 - topology.png`

---

### Phase 5.1-5.3: DHCP Server + Relay

**Task 5.1: Server-DHCP (192.168.99.10) - Services -> DHCP ON**

| Pool Name | Gateway | Start IP | Mask | Max Users | DNS |
|---|---|---|---|---|---|
| STUDENT_POOL | 192.168.10.1 | 192.168.10.100 | /24 | 50 | 192.168.99.20 |
| FACULTY_POOL | 192.168.20.1 | 192.168.20.100 | /24 | 20 | 192.168.99.20 |
| ADMIN_POOL | 192.168.30.1 | 192.168.30.100 | /24 | 20 | 192.168.99.20 |
| GUEST_POOL | 192.168.40.1 | 192.168.40.100 | /24 | 50 | 192.168.99.20 |
| IT_POOL | 192.168.50.1 | 192.168.50.100 | /24 | 20 | 192.168.99.20 |
| MEDICAL_POOL | 192.168.60.1 | 192.168.60.100 | /24 | 20 | 192.168.99.20 |

Note: Default `serverPool` and `pool1` removed/disabled (PT bug: set Start IP 0.0.0.0 Max 0 to disable if Remove fails)

**Task 5.2: DHCP Relay on Core-MLS-3560**
```
interface Vlan10-60
 ip helper-address 192.168.99.10
```
`show running-config | include helper` -> 6 lines

**Task 5.3: PCs to DHCP**
- Student-PC1 -> Desktop -> IP Configuration -> DHCP -> Request successful
- Results: Student 10.100, Faculty 20.100, Admin 30.100, Guest-Laptop 40.101 Wireless0, IT 50.100, Medical 60.101

**Evidence:**
- `results/enterprise/day 2/phase5.1-server-dhcp-pools.png`
- `results/enterprise/day 2/phase5.1-student-pc1-dhcp-configuration.png`
- `results/enterprise/day 2/phase5.1-faculty-pc1-dhcp-configuration.png`
- `results/enterprise/day 2/phase5.1-guest-laptop-dhcp-configuration.png.png` (Wireless0 40.101)
- `results/enterprise/day 2/phase5.1-it-pc1-dhcp-configuration.png`
- `results/enterprise/day 2/phase5.1-medical-pc1-dhcp-configuration.png`
- `results/enterprise/day 2/phase5.1-topology.png`

---

### Phase 6.1: Wireless Security - WPA2-PSK

**AP-Guest:**
- Port 1 SSID: `Guest-WiFi`, Auth: `WPA2-PSK`, PSK: `Nopass@123`, Encryption: `AES`, Channel: 6, Coverage: 140

**AP-IT:**
- Port 1 SSID: `IT-WiFi`, Auth: `WPA2-PSK`, PSK: `Cisco123!`, Encryption: `AES`, Channel: 6

**Laptop Reconnect:**
- PC Wireless -> Connect -> Refresh -> Select SSID -> Pre-shared Key -> Associated
- IP Config -> DHCP Renew -> Still gets 40.101 / 50.x

**Evidence:**
- `results/enterprise/day 3/phase6.1-ap-guest-wifi-username-and-authentication.png`
- `results/enterprise/day 3/phase6.1-ap-it-wifi-username-and-authentication.png`
- `results/enterprise/day 3/phase6.1-topology.png` (dotted wireless links)

---

### Phase 6.2-6.3: DNS and HTTP - college.local Portal

**Server-Web (192.168.99.20) - Services -> DNS ON**
- `college.local A 192.168.99.20`
- `smartcollege.local A 192.168.99.20`
- `server-dhcp.local A 192.168.99.10`
- `internet.local A 8.8.8.8` (added in Phase 9)

**HTTP ON:**
- index.html replaced with full Enterprise Portal:
  - Header: SMART COLLEGE NETWORK Enterprise VLAN Design
  - Status: 6 VLANs Active Trunk Up, 7 Access Switches Forwarding, DHCP Relay Active, WPA2-PSK Secured
  - Cards: VLAN 10-60 details with Network/Gateway/DHCP Pool/Switch Port
  - Core Infrastructure: VLAN 99, Core-MLS, Edge-Router
  - Services Status: DHCP, DNS, HTTP, Wireless, Inter-VLAN Routing

**Verification:**
```
C:\> nslookup college.local
Server: [192.168.99.20]
Name: college.local Address: 192.168.99.20

Web Browser http://college.local -> Portal loads
Web Browser http://smartcollege.local -> Same portal
Web Browser http://192.168.99.20 -> Same portal
```

**Evidence:**
- `results/enterprise/day 3/phase6.3-college.local-student-pc1-live-portal.png`
- `results/enterprise/day 3/phase6.3-smartcollege.local-student-pc1-live-portal.png`
- `results/enterprise/day 3/phase6.3-student-pc1-nslookup.png`

---

### Phase 6.4: Extended ACLs - Security (Enterprise)

**Objective:** Guests only DNS/DHCP/Web, not internal PCs. Students not Admin.

**Core-MLS-3560 - Corrected ACLs (Permit DNS/Web first, then deny):**

```
! === GUEST ACL ===
ip access-list extended GUEST_RESTRICT
 permit udp any any eq 67
 permit udp any any eq 68
 permit udp any host 192.168.99.20 eq 53
 permit udp any host 192.168.99.10 eq 67
 permit udp any host 192.168.99.10 eq 68
 permit tcp any host 192.168.99.20 eq 80
 permit tcp any host 192.168.99.20 eq 443
 permit tcp any host 192.168.99.20 eq 53
 deny ip 192.168.40.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny ip 192.168.40.0 0.0.0.255 192.168.20.0 0.0.0.255
 deny ip 192.168.40.0 0.0.0.255 192.168.30.0 0.0.0.255
 deny ip 192.168.40.0 0.0.0.255 192.168.50.0 0.0.0.255
 deny ip 192.168.40.0 0.0.0.255 192.168.60.0 0.0.0.255
 deny ip 192.168.40.0 0.0.0.255 192.168.99.0 0.0.0.255
 permit ip any any

! === STUDENT ACL ===
ip access-list extended STUDENT_RESTRICT
 permit udp any host 192.168.99.20 eq 53
 permit tcp any host 192.168.99.20 eq 80
 permit tcp any host 192.168.99.20 eq 53
 permit tcp any host 192.168.99.20 eq 443
 permit udp any host 192.168.99.10 eq 67
 permit udp any host 192.168.99.10 eq 68
 deny ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
 deny ip 192.168.10.0 0.0.0.255 192.168.99.0 0.0.0.255
 permit ip any any

interface Vlan40
 ip access-group GUEST_RESTRICT in
interface Vlan10
 ip access-group STUDENT_RESTRICT in
```

**Note:** Packet Tracer 3560 bug - `show ip interface VlanX | include access` always says `not set` even when applied. Trust `show run | include access-group` which shows both.

**Verification - Real ACL Test Results (Enterprise):**

| Source | Destination | Expected | Actual | Reason |
|---|---|---|---|---|
| Guest-Laptop 40.100 Wireless0 | Student-PC1 10.100 | Blocked | Destination host unreachable (100% loss) | deny 40->10 |
| Guest-Laptop 40.100 | 99.20 ping | Blocked ICMP | Destination host unreachable | deny 40->99 for ICMP, but TCP 80 permitted |
| Guest-Laptop 40.100 | 99.20 Web http://college.local | Allow | Portal loads! | permit tcp 40->99.20 eq 80 |
| Student-PC1 10.100 | Admin 30.100 | Blocked | Destination host unreachable | deny 10->30 |
| Student-PC1 10.100 | Faculty 20.100 | Allow | Reply TTL 127 | permit any |

This proves port-specific security: ICMP blocked, HTTP/DNS allowed.

**Evidence:**
- `results/enterprise/day 3/phase6.4-student-pc1-acl-ping-tests.png`
- `results/enterprise/day 3/phase6.4-guest-laptop-acl-ping-tests.png`
- `results/enterprise/day 3/phase6.4-guest-laptop-live-portal-acl-test.png`

---

### Phase 9.1-9.3: ISP, Internet Server, and NAT PAT

**Topology Wiring:**
- Edge-Router Gig0/0/1 -> ISP-Router Gig0/0/0 (Copper Straight)
- ISP-Router Gig0/0/1 -> Server-Internet Fa0
- ISP-Router: 200.1.1.2/30 public, 8.8.8.1/24 internet LAN
- Server-Internet: 8.8.8.8/24 GW 8.8.8.1 HTTP ON

**Task 9.1: ISP-Router**
```
hostname ISP-Router
interface Gig0/0/0 description Link to Edge-Router ip 200.1.1.2 255.255.255.252 no shut
interface Gig0/0/1 description Simulated Internet LAN ip 8.8.8.1 255.255.255.0 no shut
ip route 192.168.0.0 255.255.0.0 200.1.1.1
ip route 10.0.0.0 255.255.255.0 200.1.1.1
```

**Edge-Router Public:**
```
interface Gig0/0/1 ip 200.1.1.1 255.255.255.252 no shut
ip route 0.0.0.0 0.0.0.0 200.1.1.2
```

**Verification:**
```
Edge# ping 200.1.1.2 80% (4/5)
Edge# ping 8.8.8.8 80%
Student ping 8.8.8.8 Reply 15ms TTL 125 (before NAT 75%, after 100%)
```

**Internet Portal - White Text Fix:**
- Server-Internet index.html with black cards #141b2d, text forced white #ffffff !important
- Header gradient blue->pink WELCOME TO THE INTERNET
- Shows request path: Student 10.101 -> SW-Student -> Core 10.1 -> Edge 10.0.0.2 -> 200.1.1.1 -> ISP 200.1.1.2 -> 8.8.8.8

**Task 9.2: NAT Overload (PAT)**
```
interface Gig0/0/1 ip nat outside
interface Gig0/0/0 ip nat inside
access-list 1 permit 192.168.0.0 0.0.255.255
access-list 1 permit 10.0.0.0 0.0.0.3
ip nat inside source list 1 interface Gig0/0/1 overload
ip route 0.0.0.0 0.0.0.0 200.1.1.2
ip route 192.168.0.0 255.255.0.0 10.0.0.1
```

**Verification:**
```
Edge# show ip nat translations
icmp 200.1.1.1:10 192.168.10.101:10 8.8.8.8:10 8.8.8.8:10
icmp 200.1.1.1:11 192.168.10.101:11 8.8.8.8:11 8.8.8.8:11

Student# nslookup internet.local Server 192.168.99.20 -> 8.8.8.8
Student# ping 8.8.8.8 Reply 1ms 0% loss
Browser http://8.8.8.8 -> Internet portal
Browser http://internet.local -> Same portal via DNS
```

**Task 9.3: Core Verification**
```
Core# show ip route
S* 0.0.0.0/0 via 10.0.0.2
```
Already present from Phase 2.3 - No change needed, just verified.

**Evidence:**
- `results/enterprise/day 4/phase9.1-8.8.8.8-live-portal-run-student-pc1.jpeg` (white text portal)
- `results/enterprise/day 4/phase9.1-edge-router-ping-test-to-student-pc1-and-internet.png`
- `results/enterprise/day 4/phase9.1-student-pc1-ping-test-to-internet.jpeg`
- `results/enterprise/day 4/phase9.2-student-pc1-ping-tests.png` (0% loss after NAT)
- `results/enterprise/day 4/phase9.2-student-pc1-nslookup-internet-live-portal.png`
- `results/enterprise/day 4/phase9.2-guest-laptop-internet-live-portal.png`

---

### Phase 11: Wireshark / Simulation Mode Captures

**Simulation Mode Controls:**
- Realtime tab = Normal mode
- Simulation tab = Capture mode
- Edit Filters -> Show All/None -> Check needed protocol
- Play = Auto, Capture/Forward = One packet at a time (use for DORA/ARP), Reset Simulation = Clear
- Stop = Click Realtime tab

**Task 11.1: DHCP DORA**
- Filter: ONLY DHCP
- Trigger: Student-PC1 Static 0.0.0.0 -> DHCP Request
- Capture: Click Capture/Forward 4 times
- Expected in Event List:
  1. Discover 0.0.0.0 -> 255.255.255.255
  2. Offer 192.168.99.10 -> 192.168.10.101
  3. Request 192.168.10.101 -> 192.168.99.10
  4. ACK 192.168.99.10 -> 192.168.10.101

**Evidence:** `results/enterprise/day 4/phase11.1-wireshark-dhcp.png` (Filter shows DHCP only, Event List yellow DHCP packets)

**Task 11.2: TCP 3-Way Handshake**
- Filter: TCP + HTTP (NOT DHCP)
- Trigger: Student-PC1 Browser http://college.local -> Go, but use Play/Capture
- Expected: SYN 10.101:1025 -> 99.20:80, SYN-ACK 99.20:80 -> 10.101:1025, ACK 10.101 -> 99.20, then HTTP GET

**Evidence:** `results/enterprise/day 4/phase11.2-wireshark-tcp-http.png` (Filter TCP, Event List green TCP packets: Student-PC1 -> SW-Student -> Core -> SW-Admin -> Web-Server -> back)

**Task 11.3: ARP Resolution**
- Filter: ARP + ICMP
- Trigger: Admin-PC1 `arp -d` then `ping 192.168.30.1`
- Expected: ARP Request broadcast Who has 30.1? Tell 30.100, ARP Reply unicast 30.1 is at MAC, then ICMP Echo Request/Reply

**Evidence:** `results/enterprise/day 4/phase11.3-wireshark-icmp-arp.png` (Filter ARP, ICMP, Event List green ARP + black ICMP)

**Note:** Real Wireshark on host wlo1 interface (172.16.222.0/24) captured separately for report: arp, icmp, dns, tcp filters show same protocols as Simulation Mode. Simulation proves VLAN topology internal traffic.

---

## Day 5 - Task 3: Test the Complete Network (30 mins) - Basic Topology

### IP Addressing Note
> **No DHCP implemented in Basic Topology.** All PCs and Server use Static IP addressing manually configured (IP, Mask, Gateway, DNS). Therefore, "Student receives valid address" is verified via `ipconfig` showing static config, not DHCP lease. Enterprise Topology uses DHCP.

### Final Test Results Table - Basic Deliverable

| Test | Command / Check | Expected result | Actual Result | Status | Evidence |
|---|---|---|---|---|---|
| **Student PC receives a valid address** | `ipconfig` on Student PC | Valid IP in 192.168.10.0/24, Mask 255.255.255.0, Gateway 192.168.10.1 | Static: 192.168.10.10/24, GW 192.168.10.1, DNS 8.8.8.8 - Manually configured | **Pass** | `27_Student_IPConfig_Static.png` |
| **Student reaches its gateway** | `ping 192.168.10.1` from Student | Reply | Reply from 192.168.10.1 bytes=32 TTL=255 | **Pass** | `28_Student_Gateway_Ping.png` |
| **Permitted student-to-server communication** | `ping 192.168.30.100` from Student | Pass - STUDENT-ACL Rule 10 permit host 30.100 | Reply from 192.168.30.100 | **Pass** | `29_Student_Server_Allow.png` |
| **Student attempts to access admin devices** | `ping 192.168.30.10` from Student | **Blocked** - STUDENT-ACL Rule 20 deny 30.0 | Request timed out / Destination unreachable. Hit count on `show access-lists` increases | **Blocked (Expected)** | `30_Student_Admin_Blocked.png` |
| **Guest attempts to access internal resources** | `ping 192.168.10.10` `20.10` `30.100` from Guest | **Blocked** - GUEST-ACL deny 10,20,30 | All 3: Request timed out. Hits on GUEST-ACL 10,20,30 | **Blocked (Expected)** | `31_Guest_Internal_Blocked.png` |
| **Faculty accesses permitted resources** | `ping 192.168.30.100` from Faculty | Pass - No ACL on g0/0/0.20 | Reply from 192.168.30.100 | **Pass** | `32_Faculty_Server_Allow.png` |

## Enterprise Final Test Results - Deliverable

| Test | Command / Check | Expected | Actual | Status | Evidence |
|---|---|---|---|---|---|
| **Enterprise Student receives DHCP** | `ipconfig` Student-PC1 | 192.168.10.100/24 GW 10.1 DNS 99.20 | DHCP: 192.168.10.100/24 GW 10.1 DNS 99.20 DHCP Server 99.10 | **Pass** | `day 2/phase5.1-student-pc1-dhcp-configuration.png` |
| **Enterprise Guest Wireless DHCP** | `ipconfig` Guest-Laptop Wireless0 | 192.168.40.100/24 GW 40.1 | DHCP: 40.101 Wireless0 GW 40.1 DNS 99.20 | **Pass** | `day 2/phase5.1-guest-laptop-dhcp-configuration.png.png` |
| **Core Trunk** | `show interfaces trunk` | Fa0/1-6 trunking 1,10,20,30,40,50,60,99 forwarding | 1,10,20,30,40,50,60,99 active and forwarding | **Pass** | `phase 2.3 topology` |
| **DHCP Relay** | `show run | include helper` | 6 helper-address | 6x ip helper-address 192.168.99.10 | **Pass** | CLI |
| **WPA2-PSK** | AP Config Port 1 | Guest-WiFi Nopass@123, IT-WiFi Cisco123! | SSID Guest-WiFi WPA2-PSK Nopass@123 AES, IT-WiFi Cisco123! AES | **Pass** | `day 3/phase6.1-ap-*.png` |
| **DNS college.local** | `nslookup college.local` | 192.168.99.20 | Server 99.20 -> 99.20 | **Pass** | `day 3/phase6.3-student-pc1-nslookup.png` |
| **HTTP college.local** | `http://college.local` Browser | Portal loads | SMART COLLEGE NETWORK portal loads | **Pass** | `day 3/phase6.3-college.local-*.png` |
| **HTTP smartcollege.local** | `http://smartcollege.local` | Portal loads | Same portal loads | **Pass** | `day 3/phase6.3-smartcollege.local-*.png` |
| **Student to Admin ACL** | `ping 30.100` Student | Blocked | Destination host unreachable | **Blocked Expected** | `day 3/phase6.4-student-pc1-acl-ping-tests.png` |
| **Student to Faculty ACL** | `ping 20.100` Student | Allow | Reply TTL 127 | **Pass** | Same |
| **Guest to Student ACL** | `ping 10.100` Guest | Blocked | Destination host unreachable | **Blocked Expected** | `day 3/phase6.4-guest-laptop-acl-ping-tests.png` |
| **Guest to Web ACL** | `http://college.local` Guest | Allow (TCP 80) | Portal loads despite ping blocked | **Pass** | `day 3/phase6.4-guest-laptop-live-portal-acl-test.png` |
| **ISP Link** | `ping 200.1.1.2` Edge | 80% | 80% (4/5) | **Pass** | `day 4/phase9.1-edge-router-ping-test*` |
| **Internet Reach** | `ping 8.8.8.8` Student | Reply | Reply 1ms 0% loss after NAT | **Pass** | `day 4/phase9.1-student-pc1-ping-test*` |
| **NAT PAT** | `show ip nat translations` | 192.168.10.101 -> 200.1.1.1 | icmp 200.1.1.1:10 192.168.10.101:10 8.8.8.8:10 | **Pass** | `day 4/phase9.2-student-pc1-ping-tests.png` |
| **DNS internet.local** | `nslookup internet.local` | 8.8.8.8 | Server 99.20 -> 8.8.8.8 | **Pass** | `day 4/phase9.2-student-pc1-nslookup-internet-live-portal.png` |
| **HTTP internet.local** | `http://internet.local` Guest | Internet portal | WELCOME TO THE INTERNET white text portal | **Pass** | `day 4/phase9.2-guest-laptop-internet-live-portal.png` |
| **DHCP DORA Simulation** | Simulation DHCP filter | Discover/Offer/Request/ACK | Event List yellow DHCP 4 packets | **Pass** | `day 4/phase11.1-wireshark-dhcp.png` |
| **TCP Handshake Simulation** | Simulation TCP filter | SYN/SYN-ACK/ACK | Event List green TCP Student->Web-Server | **Pass** | `day 4/phase11.2-wireshark-tcp-http.png` |
| **ARP Simulation** | Simulation ARP+ICMP | ARP Request/Reply + ICMP | Event List green ARP + black ICMP | **Pass** | `day 4/phase11.3-wireshark-icmp-arp.png` |

## Troubleshooting Log

| Issue | Cause | Fix |
|---|---|---|
| Same-VLAN timeout | Cables on Fa0/6, Fa0/8 not Fa0/3, Fa0/5 | Port labels enabled, rewired to match show vlan brief |
| Router flapping | Gig0/1 not trunk | switchport mode trunk |
| show interfaces trunk invalid on Router | Switch-only command | Used show ip interface brief on Router |
| Simulation only ICMP | ARP cache cached | arp -d + clear arp-cache + filter ARP+ICMP |
| Simulation only ARP | Simple PDU expired after gateway ARP | Used ping inside Simulation Mode, not Simple PDU |
| Wireshark no Wi-Fi | Linux uses wlo1 not Wi-Fi | Used wlo1 interface |
| Wireshark permission | Not in wireshark group | usermod -a -G wireshark + re-login or sudo wireshark |
| Hotspot IP 172.16.222.x not 192.168.43.x | Android 12+ hotspot range | Documented as observation |
| 3560 vlan 10,20,30,40,50,60,99 invalid | 3560 IOS doesn't support comma in vlan command | Created vlans one by one: vlan 10, vlan 20... |
| 2960 switchport trunk encapsulation dot1q invalid | 2960 doesn't support encapsulation command | Removed, used only switchport mode trunk |
| DHCP serverPool cannot be removed | PT bug default pool | Disabled: Start IP 0.0.0.0 Max 0 |
| show ip interface VlanX no access-group | PT 3560 display bug | Trust show run | include access-group, test via ping not show |
| Guest laptop DHCP failed | AP SSID Default, not Guest-WiFi + distance | Set AP SSID Guest-WiFi + move laptop closer + WPA2-PSK key |
| Laptop module swap | Laptop-PT already wireless, no need Physical tab | Used PC Wireless Connect tab directly |
| Edge ip route 8.8.8.0 255.255.0.0 inconsistent | Mask mismatch | Used 0.0.0.0/0 default route instead |
| Guest ping 99.20 fails but web loads | ACL blocks ICMP but permits TCP 80 | Correct enterprise behavior - ping blocked, web allowed |
| Simulation shows STP + DHCP | Filter has many protocols | Edit Filters -> Show All/None -> Check only needed (DHCP only, TCP+HTTP, ARP+ICMP) |

```
04-Design-and-Simulation-of-a-Smart-College-Network-Using-VLANs/
├── README.md
├── packet tracer/
│   ├── basic/
│   │   ├── smart-college-network-basic-v1.pkt
│   │   ├── smart-college-network-basic-v2.pkt
│   │   ├── smart-college-network-basic-v3.pkt
│   │   ├── smart-college-network-basic-v4.pkt
│   │   └── smart-college-network-basic-v5.pkt
│   └── enterprise/
│       ├── 192.168.99.20/
│       │   └── index.html (college.local portal)
│       ├── 8.8.8.8/
│       │   └── index.html (internet.local white text portal)
│       ├── smart-college-wifi-phase 2.2-enterprise.pkt
│       ├── smart-college-wifi-phase 2.3-enterprise.pkt
│       ├── smart-college-wifi-phase 5.1-enterprise.pkt
│       ├── smart-college-wifi-phase 6.1-enterprise.pkt
│       ├── smart-college-wifi-phase 6.2-enterprise.pkt
│       ├── smart-college-wifi-phase 6.3-enterprise.pkt.pkt
│       ├── smart-college-wifi-phase 6.4-enterprise-final.pkt.pkt
│       ├── smart-college-wifi-phase 6.4-enterprise.pkt
│       ├── smart-college-wifi-phase 9.1-enterprise.pkt
│       └── smart-college-wifi-phase 9.2-enterprise.pkt
├── presentation/
│   └── v1.2 presentation.pptx
├── report/
│   └── v1.2 report.pdf
├── results/
│   ├── day 1/
│   │   └── topology.png
│   ├── day 2/
│   │   ├── Admin PC ipconfig.png
│   │   ├── Faculty PC ipconfig.png
│   │   ├── Guest PC ipconfig.png
│   │   ├── Student PC ipconfig.png
│   │   ├── ipconfig & ping test results.png
│   │   └── show vlan brief.png
│   ├── day 3/
│   │   ├── Admin PC ping test results.png
│   │   ├── Admin PC to Server same VLAN ping test.png
│   │   ├── Core Router ping test results.png
│   │   ├── ICMP & ARP Simulation.png
│   │   ├── gateway tests/
│   │   │   ├── Admin PC gateway ping test.png
│   │   │   ├── Faculty PC gateway ping test.png
│   │   │   ├── Guest PC gateway ping test.png
│   │   │   ├── Server gateway ping test.png
│   │   │   └── Student PC gateway ping test.png
│   │   └── inter-vlan tests/
│   │       ├── Admin PC -> Guest PC.png
│   │       ├── Faculty PC -> Server.png
│   │       ├── Guest PC -> Student PC.png
│   │       └── Student PC inter-vlan ping tests.png
│   ├── day 4/
│   │   ├── ACL configurations.png
│   │   ├── acl/
│   │   │   ├── Faculty PC -> Server.png
│   │   │   ├── Guest PC post-ACL ping tests.png
│   │   │   └── Student PC post-ACL ping tests.png
│   │   └── wireshark/
│   │       ├── linux-terminal-ip-ping test.png
│   │       ├── wireshark-arp-wlo1.png
│   │       ├── wireshark-dns-wlo1.png
│   │       └── wireshark-icmp-wlo1.png
│   ├── day 5/
│   │   ├── Faculty PC ip tests.png
│   │   ├── Guest PC ip tests.png
│   │   └── Student PC ip tests.png
│   ├── day 6/
│   │   └── topology validation/
│   │       ├── acl-audit.md
│   │       ├── router-audit.md
│   │       └── switch-audit.md
│   └── enterprise/
│       ├── day 1/
│       │   └── phase 2.3 - topology.png
│       ├── day 2/
│       │   ├── phase5.1-faculty-pc1-dhcp-configuration.png
│       │   ├── phase5.1-guest-laptop-dhcp-configuration.png.png (Wireless0 40.101)
│       │   ├── phase5.1-it-pc1-dhcp-configuration.png
│       │   ├── phase5.1-medical-pc1-dhcp-configuration.png
│       │   ├── phase5.1-server-dhcp-pools.png (6 pools + disabled serverPool)
│       │   ├── phase5.1-student-pc1-dhcp-configuration.png
│       │   └── phase5.1-topology.png (all green + wireless dotted)
│       ├── day 3/
│       │   ├── phase6.1-ap-guest-wifi-username-and-authentication.png (WPA2-PSK Nopass@123)
│       │   ├── phase6.1-ap-it-wifi-username-and-authentication.png (WPA2-PSK Cisco123!)
│       │   ├── phase6.1-topology.png
│       │   ├── phase6.3-college.local-student-pc1-live-portal.png (http://college.local)
│       │   ├── phase6.3-smartcollege.local-student-pc1-live-portal.png (http://smartcollege.local)
│       │   ├── phase6.3-student-pc1-nslookup.png (college.local + smartcollege.local -> 99.20)
│       │   ├── phase6.4-guest-laptop-acl-ping-tests.png (Guest->Student blocked, Guest->99.20 ping blocked)
│       │   ├── phase6.4-guest-laptop-live-portal-acl-test.png (Guest->college.local web allowed despite ping blocked)
│       │   └── phase6.4-student-pc1-acl-ping-tests.png (Student->Admin blocked, Student->Faculty allowed)
│       └── day 4/
│           ├── phase11.1-wireshark-dhcp.png (DHCP DORA Discover/Offer/Request/ACK Simulation)
│           ├── phase11.2-wireshark-tcp-http.png (TCP SYN/SYN-ACK/ACK to Web-Server)
│           ├── phase11.3-wireshark-icmp-arp.png (ARP Request/Reply + ICMP)
│           ├── phase9.1-8.8.8.8-live-portal-run-student-pc1.jpeg (white text black boxes internet portal)
│           ├── phase9.1-edge-router-ping-test-to-student-pc1-and-internet.png (80% to 200.1.1.2 and 8.8.8.8)
│           ├── phase9.1-student-pc1-ping-test-to-internet.jpeg (15ms TTL 125)
│           ├── phase9.1-topology.png
│           ├── phase9.2-guest-laptop-internet-live-portal.png (http://internet.local white portal)
│           ├── phase9.2-student-pc1-nslookup-internet-live-portal.png (internet.local -> 8.8.8.8)
│           └── phase9.2-student-pc1-ping-tests.png (ping 8.8.8.8 0% loss, NAT translations)
└── wireshark/
    └── wiresharkv1.pcapng
```
