```
Router# show access-lists
Extended IP access list STUDENT-ACL
    10 permit ip 192.168.10.0 0.0.0.255 host 192.168.30.100
    20 deny ip 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
    30 permit ip 192.168.10.0 0.0.0.255 any
Extended IP access list GUEST-ACL
    10 deny ip 192.168.40.0 0.0.0.255 192.168.10.0 0.0.0.255
    20 deny ip 192.168.40.0 0.0.0.255 192.168.20.0 0.0.0.255
    30 deny ip 192.168.40.0 0.0.0.255 192.168.30.0 0.0.0.255
    40 permit ip 192.168.40.0 0.0.0.255 any
```

```
Router# show ip interface GigabitEthernet0/0/0.10
GigabitEthernet0/0/0.10 is up, line protocol is up (connected)
  Internet address is 192.168.10.1/24
  Broadcast address is 255.255.255.255
  Address determined by setup command
  MTU is 1500 bytes
  Helper address is not set
  Directed broadcast forwarding is disabled
  Outgoing access list is not set
  Inbound  access list is STUDENT-ACL
  Proxy ARP is enabled
  Security level is default
  Split horizon is enabled
  ICMP redirects are always sent
  ICMP unreachables are always sent
  ICMP mask replies are never sent
  IP fast switching is disabled
  IP fast switching on the same interface is disabled
  IP Flow switching is disabled
  IP Fast switching turbo vector
  IP multicast fast switching is disabled
  IP multicast distributed fast switching is disabled
  Router Discovery is disabled
  IP output packet accounting is disabled
  IP access violation accounting is disabled
  TCP/IP header compression is disabled
  RTP/IP header compression is disabled
  Probe proxy name replies are disabled
  Policy routing is disabled
  Network address translation is disabled
  BGP Policy Mapping is disabled
  Input features: MCI Check
  WCCP Redirect outbound is disabled
  WCCP Redirect inbound is disabled
  WCCP Redirect exclude is disabled
  ```

  ```
  Router# show ip interface GigabitEthernet0/0/0.40
GigabitEthernet0/0/0.40 is up, line protocol is up (connected)
  Internet address is 192.168.40.1/24
  Broadcast address is 255.255.255.255
  Address determined by setup command
  MTU is 1500 bytes
  Helper address is not set
  Directed broadcast forwarding is disabled
  Outgoing access list is not set
  Inbound  access list is GUEST-ACL
  Proxy ARP is enabled
  Security level is default
  Split horizon is enabled
  ICMP redirects are always sent
  ICMP unreachables are always sent
  ICMP mask replies are never sent
  IP fast switching is disabled
  IP fast switching on the same interface is disabled
  IP Flow switching is disabled
  IP Fast switching turbo vector
  IP multicast fast switching is disabled
  IP multicast distributed fast switching is disabled
  Router Discovery is disabled
  IP output packet accounting is disabled
  IP access violation accounting is disabled
  TCP/IP header compression is disabled
  RTP/IP header compression is disabled
  Probe proxy name replies are disabled
  Policy routing is disabled
  Network address translation is disabled
  BGP Policy Mapping is disabled
  Input features: MCI Check
  WCCP Redirect outbound is disabled
  WCCP Redirect inbound is disabled
  WCCP Redirect exclude is disabled
  ```