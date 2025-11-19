<img width="850" height="692" alt="nat-lab" src="https://github.com/user-attachments/assets/4cbcda03-30e1-4490-9b76-6d5d1e8c5731" />


# NAT Pool Exhaustion Lab


## Overview
This lab simulates a Denial of Service (DoS) scenario caused by a misconfigured NAT pool. The objective was to diagnose
network connectivity issues where internal users were intermittently blocked from accessing the internet, identify the
bottleneck, and implement a scalable solution.

## 🛠 Technologies
* Cisco IOS
* Cisco Packet Tracer
* Network Address Translation (NAT/PAT)

## The Scenario
* Topology: 10 Internal PCs connected via Switch to a Router.
* The Problem: The NAT pool was configured with only 5 public IP addresses for 10 users.
* Symptom: After 5 simultaneous connections, the 6th user received `Request timed out`.
* Diagnosis: Verified via `show ip nat statistics` showing 100% allocation and rising misses.

## The Solution
The issue was resolved by expanding the address pool and enabling Port Address Translation (PAT) using the `overload` keyword.
This allows multiple internal users to share a single public IP using unique port numbers.

### Key Configuration Commands
```cisco
! 1. Remove the restrictive configuration
no ip nat inside source list 1 pool INTERNET-POOL

! 2. Expand the NAT Pool (From 5 to 12 addresses)
ip nat pool INTERNET-POOL 203.0.113.2 203.0.113.13 netmask 255.255.255.240

! 3. Enable PAT (Overload) to prevent future exhaustion
ip nat inside source list 1 pool INTERNET-POOL overload


## Verification

  * Stress Test: Successfully initiated continuous pings (`ping -n 1000`) from all 10 PCs simultaneously.
  * Statistics: `show ip nat statistics` confirmed 0 misses.
  * Translation Table: `show ip nat translations` verified usage of Port Address Translation (multiple internal IPs mapped to the same global IP with different ports).

