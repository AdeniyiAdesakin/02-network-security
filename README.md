# Module 2: Network Security with ACLs and Port Security

**Extended ACLs | Traffic Filtering | Port Security | Violation Recovery | Interface Hardening**

[← Return to the main project](../README.md)

## Module Overview

This module strengthens the campus access layer. I added dedicated switches for the main user departments, applied extended access control lists to enforce traffic restrictions, secured server-facing interfaces with static MAC addresses, and disabled unused switch ports.

The validation includes both permitted and denied traffic, the directional effect of an ACL, and the observed shutdown and recovery of a protected server port.

## Security Requirements

| Control | Intended Result |
|---|---|
| ACL 100 | Prevent VLAN 30 from reaching VLAN 100 while allowing approved departmental access |
| ACL 110 | Block ICMP echo from VLAN 20 to VLAN 10 while permitting other IP traffic |
| Static port security | Restrict server-facing ports to approved MAC addresses |
| Shutdown violation mode | Disable a secured port when an unauthorized device is connected |
| Unused-port shutdown | Reduce the number of exposed access interfaces |

## Phase 5: Expanded and Secured the Access Layer

### 1. Added Dedicated Access Switches

I added a Cisco 2960 access switch for the Administration building and repeated the design for the Academics and Student buildings.

<p align="center">
  <img src="../images/security/01.png" width="750" alt="Selecting a Cisco 2960 access switch">
</p>

<p align="center">
  <img src="../images/security/02.png" width="750" alt="Renaming the Administration access switch">
</p>

```text
AccessSwitch-AdministrationBuilding(config)# vlan 10
AccessSwitch-AdministrationBuilding(config-vlan)# name AdministrationBuilding
AccessSwitch-AdministrationBuilding(config)# interface range fa0/1-24
AccessSwitch-AdministrationBuilding(config-if-range)# switchport mode access
AccessSwitch-AdministrationBuilding(config-if-range)# switchport access vlan 10
AccessSwitch-AdministrationBuilding(config-if-range)# do write
```

<p align="center">
  <img src="../images/security/03.png" width="750" alt="Configuring the Administration switch for VLAN 10">
</p>

<p align="center">
  <img src="../images/security/04.png" width="750" alt="Configuring the Academics switch for VLAN 20">
</p>

<p align="center">
  <img src="../images/security/05.png" width="750" alt="Configuring the Student switch for VLAN 30">
</p>

### 2. Removed Unused VLAN Assignments

```text
CoreSwitch(config)# interface range fa0/3-5
CoreSwitch(config-if-range)# no switchport access vlan 10
CoreSwitch(config)# interface range fa0/7-9
CoreSwitch(config-if-range)# no switchport access vlan 20
CoreSwitch(config)# interface range fa0/11-14
CoreSwitch(config-if-range)# no switchport access vlan 30
CoreSwitch(config)# do write
CoreSwitch# show vlan brief
```

<p align="center">
  <img src="../images/security/06.png" width="750" alt="Removing obsolete VLAN assignments">
</p>

<p align="center">
  <img src="../images/security/07.png" width="750" alt="Verifying VLAN port assignments">
</p>

### 3. Applied ACL 100 to Protect the Server VLAN

```text
MC_router(config)# access-list 100 deny ip 192.168.30.0 0.0.0.255 192.168.100.0 0.0.0.255
MC_router(config)# access-list 100 permit ip 192.168.10.0 0.0.0.255 192.168.100.0 0.0.0.255
MC_router(config)# access-list 100 permit ip 192.168.20.0 0.0.0.255 192.168.100.0 0.0.0.255
MC_router(config)# interface GigabitEthernet0/1.100
MC_router(config-subif)# ip access-group 100 out
MC_router(config-subif)# do write
```

<p align="center">
  <img src="../images/security/08.png" width="750" alt="Creating and applying ACL 100">
</p>

<p align="center">
  <img src="../images/security/09.png" width="750" alt="Verifying ACL 100">
</p>

> **Scope note:** ACL 100 ends with an implicit deny. Any source that is not explicitly permitted will also be denied access to VLAN 100.

### 4. Validated ACL 100

A host in VLAN 10 successfully reached server `192.168.100.10`.

<p align="center">
  <img src="../images/security/10.png" width="750" alt="Successful VLAN 10-to-server ping">
</p>

A host in VLAN 20 also reached the server.

<p align="center">
  <img src="../images/security/11.png" width="750" alt="Successful VLAN 20-to-server ping">
</p>

Traffic from VLAN 30 to the server was denied.

<p align="center">
  <img src="../images/security/12.png" width="750" alt="Blocked VLAN 30-to-server ping">
</p>

### 5. Applied ACL 110 to Restrict ICMP

```text
MC_router(config)# access-list 110 deny icmp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 echo
MC_router(config)# access-list 110 permit ip any any
MC_router(config)# interface GigabitEthernet0/1.10
MC_router(config-subif)# ip access-group 110 out
MC_router(config-subif)# do write
```

<p align="center">
  <img src="../images/security/13.png" width="750" alt="Creating and applying ACL 110">
</p>

The VLAN 20-to-VLAN 10 ping failed as intended.

<p align="center">
  <img src="../images/security/14.png" width="750" alt="Blocked VLAN 20-to-VLAN 10 ping">
</p>

The reverse-direction test succeeded.

<p align="center">
  <img src="../images/security/15.png" width="750" alt="Successful reverse-direction ping">
</p>

### 6. Configured Static Port Security

```text
CoreSwitch(config)# interface fa0/16
CoreSwitch(config-if)# switchport mode access
CoreSwitch(config-if)# switchport port-security
CoreSwitch(config-if)# switchport port-security mac-address <SERVER_MAC_ADDRESS>
CoreSwitch(config-if)# switchport port-security violation shutdown
CoreSwitch(config-if)# do write
```

<p align="center">
  <img src="../images/security/16.png" width="750" alt="Configuring switch port security">
</p>

<p align="center">
  <img src="../images/security/17.png" width="750" alt="Verifying port-security status">
</p>

### 7. Triggered and Recovered from a Violation

I moved the servers to each other’s ports and generated traffic. The affected links changed to a down state.

<p align="center">
  <img src="../images/security/18.png" width="750" alt="Server link down after a port-security violation">
</p>

```text
CoreSwitch(config)# interface fa0/16
CoreSwitch(config-if)# shutdown
CoreSwitch(config-if)# no shutdown
```

<p align="center">
  <img src="../images/security/19.png" width="750" alt="Resetting the secured interface">
</p>

<p align="center">
  <img src="../images/security/20.png" width="750" alt="Server link restored">
</p>

### 8. Disabled Unused Ports

<p align="center">
  <img src="../images/security/21.png" width="750" alt="Reviewing switch interface status">
</p>

```text
CoreSwitch(config)# interface range fa0/3-5
CoreSwitch(config-if-range)# shutdown
CoreSwitch(config)# interface range fa0/7-9
CoreSwitch(config-if-range)# shutdown
CoreSwitch(config)# interface range fa0/11-14
CoreSwitch(config-if-range)# shutdown
CoreSwitch(config)# interface range fa0/17-24
CoreSwitch(config-if-range)# shutdown
```

<p align="center">
  <img src="../images/security/22.png" width="750" alt="Shutting down unused interfaces">
</p>

<p align="center">
  <img src="../images/security/23.png" width="750" alt="Verifying administratively disabled ports">
</p>

## Module Validation Summary

| Control | Result | Evidence |
|---|---|---|
| Dedicated access switches | Configured | VLAN-specific switch configuration captured |
| ACL 100 permitted traffic | Passed | VLANs 10 and 20 reached the server network |
| ACL 100 denied traffic | Passed | VLAN 30 could not reach the server network |
| ACL 110 directional restriction | Passed | VLAN 20-to-VLAN 10 echo failed while the reverse test passed |
| Port-security configuration | Configured | Secure-up state and one secure address displayed |
| Port-security behavior | Observed | Link shutdown and recovery captured |
| Port-security violation counter | Not captured | No post-test output showed a nonzero counter |
| Unused-port shutdown | Passed | Selected interfaces shown administratively down |

## Technical Notes

- ACL 100 ends with an implicit deny.
- Extended ACL placement should reflect the intended traffic direction.
- Each secured port must use a unique server MAC address.
- A post-violation `show port-security interface` capture would provide stronger evidence.
- Unused ports should also be placed in an unused VLAN before shutdown.

## Module Outcome

This stage introduced controls at both Layer 3 and Layer 2. The ACL tests demonstrated selective network access, while port security and interface shutdown reduced the risk of unauthorized access through the switching infrastructure.

## Continue the Project

[← Previous: VLAN Segmentation and Inter-VLAN Routing](../01-vlan-routing/README.md)  
[Next: Multi-Site Routing and Layer 3 Switching →](../03-multisite-routing/README.md)
