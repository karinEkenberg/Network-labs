# ACL & TCP Return Traffic Analysis

<img width="629" height="187" alt="acl-blocking-traffic" src="https://github.com/user-attachments/assets/bda78f95-44c1-4d98-9aa1-3fe324cae7e9" />




## Overview
This lab focuses on Access Control Lists (ACLs) and the handling of TCP return traffic in a stateless environment.
The objective was to configure a router to allow external HTTP/HTTPS access to an internal web server while ensuring
that the server's reply packets are explicitly permitted based on the TCP connection state.

## Technologies
* Cisco IOS
* Extended ACLs
* TCP Three-Way Handshake
* Network Security Principles

## The Scenario
* Topology: Internal Web Server (192.168.1.100) behind a Router connected to the Internet.
* The Problem: Standard ACLs are stateless. While an inbound rule might permit traffic *to* port 80, it does not
automatically account for the return traffic (the server's reply) if strict filtering is applied.
* Security Goal: To securely filter traffic without breaking legitimate TCP sessions initiated from outside.

## The Solution
The solution involved adding the `established` keyword to the ACL. This creates a rule that checks the TCP flags (ACK or RST).
If these flags are set, it indicates the packet is part of an already existing connection and should be allowed.

### Configuration
The following Access List (ACL) was applied to the external interface (inbound):

```cisco
! Base rules allowing access TO the server
permit tcp any host 192.168.1.100 eq 80
permit tcp any host 192.168.1.100 eq 443

! The FIX: Explicitly allowing return traffic for established sessions
permit tcp any any established
````

## Verification

By inspecting the Access List after generating web traffic, we confirmed that the established rule matches the return packets.

Router Output:

```text
R1# show access-lists
Extended IP access list INTERNET-IN
    5 permit tcp any any established
    10 permit tcp any host 192.168.1.100 eq www (20 match(es))
    20 permit tcp any host 192.168.1.100 eq 443
    30 permit icmp any any (4 match(es))
    40 deny ip any any
```

*Note: Rule 5 ensures that the router acts somewhat like a stateful firewall by acknowledging the TCP session state.*

