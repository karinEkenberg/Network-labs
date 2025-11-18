
# Network Security Lab: STP Broadcast Storm Mitigation
<img width="443" height="336" alt="stp-loop-issue" src="https://github.com/user-attachments/assets/93025716-8f66-430f-be3f-691c96ac4fdf" />

## Overview
Troubleshooting a Layer 2 loop scenario. The objective was to identify and patch a Spanning Tree Protocol (STP) misconfiguration that risked creating broadcast storms.

## Observation & Challenge
* **Scenario:** A redundant link was introduced between switches.
* **Simulation Behavior:** Unlike the theoretical lab scenario, the broadcast storm was not visually observable in runtime. The switches' internal protection mechanisms transitioned the port to `Blocking` state faster than the CLI could capture the `Forwarding` state.
* **Pivot:** Since runtime diagnostics (`show spanning-tree`) showed a healthy state, I shifted to **Configuration Audit** (Static Analysis) to find the root cause.

## Root Cause Analysis
Reviewing the running configuration revealed a critical security flaw on the inter-switch link:
cisco
interface GigabitEthernet0/2
 spanning-tree portfast  ! <--- VULNERABILITY detected


**Risk:** `PortFast` on a trunk link bypasses the listening/learning states. While the switch eventually blocked the port, this configuration creates a race condition where a loop occurs temporarily during every link-up event.

## Remediation

Removed the unsafe configuration to ensure proper STP convergence.

**Patch:**

cisco
config t
interface GigabitEthernet0/2
 no spanning-tree portfast
end


## Verification

  * **Audit:** Confirmed removal of the command via `show run`.
  * **Runtime:** Verified that `show spanning-tree vlan 1` maintains the port in `Alternate Blocking` state, securing the topology against future loops.

