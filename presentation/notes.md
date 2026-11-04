# Presentation Notes

## 1. VLANs & Trunks

### Core - MLS - 3560 (CLI)

``` PuTTy
show vlan brief
show interfaces trunk
```

### Switches (All - CLI)

``` PuTTy
show vlan brief
show interfaces fa0/1 switchport
show mac-address table
```

## 2. Inter - VLAN Routing + DHCP Relay

### Core - MLS - 3560 (CLI)

``` PuTTy
show ip interface brief
show ip route
show run | include helper
```

### DHCP Demo

1. **Web-Service -> Services -> DHCP** shows the DHCP Pool.

2. **Web-Service -> Services -> HTTP -> index.html** shows the live portal.

### Student PC1 (cmd)

``` bash
ipconfig
nslookup college.local
ping 192.168.99.20
```

### Security Check via ACL

1. Realtime Mode -> Simulation Mode

2. Select ICMP + HTTP + DNS

3. Drag PDU from Student-PC1 to Faculty-PC1 (Success)

4. Drag PDU from Student-PC1 to Admin-PC1 (Success)

5. Guest Isolation (only website)

6. Simulation -> Realtime

## Commands

### VLANs

```
show vlan brief
show interfaces trunk
show interfaces switchport
show interfaces fa0/1 switchport
```

### Routers

```
show ip interface brief
show ip route
show ip route static
show running-config | include helper
```

### DHCP - Server (GUI)

```
show ip dhcp binding
```

### DHCP - Server (CLI)

```
show run | include helper-address
show ip interface Vlan10
```

### Core - MLS - 3560 (CLI)

```
enable
show access-lists
show run | include access-group
show access-lists GUEST_RESTRICT
show access-lists STUDENT_RESTRICT
show ip interface Vlan40
show ip interface Vlan10
show ip interface brief
show ip route
```

### Switches (All - CLI)

```
show vlan brief
show interfaces trunk
```

### ISP - Router (CLI)

```
show ip interface brief
show ip route
```