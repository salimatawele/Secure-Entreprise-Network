# Secure Enterprise Network


## Project Overview

This project consists of designing, implementing, testing, and progressively securing a realistic enterprise network infrastructure using **Cisco Packet Tracer**.

The objective is to simulate the network infrastructure of a small-to-medium-sized organization with multiple departments, internal servers, guest users, redundant network equipment, and controlled Internet access.

The project is developed progressively in multiple phases. Each phase introduces new networking or cybersecurity technologies and is documented with configuration files, screenshots, tests, and technical reports.

The final objective is to obtain a complete enterprise network that is:

* Segmented
* Redundant
* Routable
* Scalable
* Secure
* Tested
* Fully documented

---

# Project Objectives

The main objectives of this project are to:

* Design a realistic enterprise network topology.
* Implement network segmentation using VLANs.
* Configure 802.1Q trunking.
* Implement inter-VLAN routing.
* Provide gateway redundancy using HSRP.
* Implement Layer 2 redundancy using EtherChannel and STP.
* Implement dynamic routing using OSPF.
* Configure DHCP services.
* Implement NAT/PAT for Internet connectivity.
* Secure network devices using SSH and local authentication.
* Implement Layer 2 security mechanisms.
* Implement Layer 3 traffic filtering using ACLs.
* Perform connectivity and failover testing.
* Perform controlled security tests.
* Document configurations and network behavior.
* Produce a professional cybersecurity/networking portfolio project.

---

# Enterprise Scenario

The simulated organization contains several departments with different network requirements.

The network is divided into the following logical segments:

| Department / Function | VLAN |
| --------------------- | ---: |
| Administration        |   10 |
| Human Resources       |   20 |
| IT                    |   30 |
| Servers               |   40 |
| Guest Network         |   50 |
| Network Management    |   99 |

The architecture is designed so that departments are logically separated while controlled communication between them can be implemented through Layer 3 routing and ACLs.

The guest network will eventually be restricted from accessing sensitive internal resources while still allowing Internet access.

---

# Network Architecture

The network uses a hierarchical architecture composed of:

* 1 ISP router
* 2 edge routers
* 2 multilayer core switches
* 2 access switches
* 1 server
* 8 client PCs
* 1 Internet/Cloud representation

### High-Level Topology

```text
                              INTERNET
                            /          \
                           /            \
                        [ISP1]       [ISP2]
                          |             |
                          |             |
                       [R1-EDGE]   [R2-EDGE]
                          |             |
                          |             |
                    [SW1-CORE]======[SW2-CORE]
                       ||                ||
                       ||                ||
                 [SW3-ACCESS]      [SW4-ACCESS]
                  /   /  |  \       /  /  |  \   \
                 PC  PC  PC  PC    PC PC  PC  PC  SRV1
```

---

# Network Devices

| Device        | Model           | Quantity | Role                    |
| ------------- | --------------- | -------: | ----------------------- |
| ISP Router    | Cisco 2911      |        1 | ISP / external network  |
| Edge Router   | Cisco 2911      |        2 | Edge routing            |
| Core Switch   | Cisco 3560-24PS |        2 | Layer 3 core            |
| Access Switch | Cisco 2960      |        2 | End-device connectivity |
| Server        | Server-PT       |        1 | Internal server         |
| PC            | PC-PT           |        8 | End users               |
| Cloud         | Cloud-PT        |        1 | Internet representation |

---

# 🔌 Interface & Connection Plan

## ISP1

| Interface | Connected Device |
| --------- | ---------------- |
|  S0/0/0   | serial0 / Cloud  |
|  S0/0/1   |     R1-EDGE      |

## ISP2

| Interface | Connected Device |
| --------- | ---------------- |
|  S0/0/0   | serial1 / Cloud  |
|  S0/0/1   |     R2-EDGE      |

## R1-EDGE

| Interface | Connected Device |
| --------- | ---------------- |
| S0/0/1    | ISP1             |
| G0/1      | SW1-CORE G0/1    |
| G0/2      | R2-EDGE G0/2     |

## R2-EDGE

| Interface | Connected Device |
| --------- | ---------------- |
| S0/0/1    | ISP2             |
| G0/1      | SW2-CORE G0/1    |
| G0/2      | R1-EDGE G0/2     |

## SW1-CORE

| Interface | Connected Device |
| --------- | ---------------- |
| G0/1      | R1-EDGE G0/1     |
| G0/2      | SW2-CORE G0/2    |
| Fa0/23    | SW2-CORE Fa0/23  |
| Fa0/1     | SW3-ACCESS Fa0/1 |
| Fa0/2     | SW3-ACCESS Fa0/2 |

## SW2-CORE

| Interface | Connected Device |
| --------- | ---------------- |
| G0/1      | R2-EDGE G0/1     |
| G0/2      | SW1-CORE G0/2    |
| Fa0/23    | SW1-CORE Fa0/23  |
| Fa0/1     | SW4-ACCESS Fa0/1 |
| Fa0/2     | SW4-ACCESS Fa0/2 |

## SW3-ACCESS

| Interface | Connected Device |
| --------- | ---------------- |
| Fa0/1     | SW1-CORE Fa0/1   |
| Fa0/2     | SW1-CORE Fa0/2   |
| Fa0/3     | PC-ADMIN1        |
| Fa0/4     | PC-ADMIN2        |
| Fa0/5     | PC-RH1           |
| Fa0/6     | PC-RH2           |

## SW4-ACCESS

| Interface | Connected Device |
| --------- | ---------------- |
| Fa0/1     | SW2-CORE Fa0/1   |
| Fa0/2     | SW2-CORE Fa0/2   |
| Fa0/3     | PC-IT1           |
| Fa0/4     | PC-IT2           |
| Fa0/5     | PC-GUEST1        |
| Fa0/6     | PC-GUEST2        |
| Fa0/7     | SRV1             |

---

# VLAN Architecture

| VLAN | Name       | Network         | Virtual Gateway |
| ---: | ---------- | --------------- | --------------- |
|   10 | ADMIN      | 192.168.10.0/24 | 192.168.10.1    |
|   20 | RH         | 192.168.20.0/24 | 192.168.20.1    |
|   30 | IT         | 192.168.30.0/24 | 192.168.30.1    |
|   40 | SERVERS    | 192.168.40.0/24 | 192.168.40.1    |
|   50 | GUEST      | 192.168.50.0/24 | 192.168.50.1    |
|   99 | MANAGEMENT | 192.168.99.0/24 | 192.168.99.1    |

### VLAN 10 — ADMIN

Used by the Administration department.

### VLAN 20 — RH

Used by the Human Resources department.

### VLAN 30 — IT

Used by the IT department.

### VLAN 40 — SERVERS

Dedicated to internal servers.

### VLAN 50 — GUEST

Dedicated to guest devices and isolated from sensitive internal resources.

### VLAN 99 — MANAGEMENT

Dedicated to the management of network infrastructure.

---

# IP Addressing Plan

## VLAN Interfaces

| VLAN | SW1-CORE        | SW2-CORE        | HSRP Virtual IP |
| ---: | --------------- | --------------- | --------------- |
|   10 | 192.168.10.2/24 | 192.168.10.3/24 | 192.168.10.1    |
|   20 | 192.168.20.2/24 | 192.168.20.3/24 | 192.168.20.1    |
|   30 | 192.168.30.2/24 | 192.168.30.3/24 | 192.168.30.1    |
|   40 | 192.168.40.2/24 | 192.168.40.3/24 | 192.168.40.1    |
|   50 | 192.168.50.2/24 | 192.168.50.3/24 | 192.168.50.1    |
|   99 | 192.168.99.2/24 | 192.168.99.3/24 | 192.168.99.1    |

## Routed Links

| Link               | Network     |
| ------------------ | ----------- |
| R1-EDGE ↔ R2-EDGE  | 10.0.0.0/30 |
| R1-EDGE ↔ SW1-CORE | 10.0.0.4/30 |
| R2-EDGE ↔ SW2-CORE | 10.0.0.8/30 |

---

# Routing

## Inter-VLAN Routing

Inter-VLAN routing is performed by the multilayer core switches.

The 3560 switches use Switch Virtual Interfaces (SVIs) for each VLAN.

Routing is enabled using:

```text
ip routing
```

---

# HSRP

**Hot Standby Router Protocol (HSRP)** is used to provide gateway redundancy.

Each VLAN has:

* One IP address on SW1-CORE
* One IP address on SW2-CORE
* One virtual IP shared through HSRP

Example:

```text
VLAN 10

SW1-CORE       192.168.10.2
SW2-CORE       192.168.10.3
HSRP Virtual   192.168.10.1
```

The virtual IP is used as the default gateway by end devices.

If the active core switch becomes unavailable, the standby core switch can assume the virtual gateway role.

---

# Trunking

802.1Q trunking is used to transport multiple VLANs between switches.

Main trunk links:

* SW1-CORE ↔ SW2-CORE
* SW1-CORE ↔ SW3-ACCESS
* SW2-CORE ↔ SW4-ACCESS

The trunk links allow the required VLANs to traverse the infrastructure.

---

# EtherChannel

EtherChannel will be implemented between redundant switch links.

The objective is to:

* Increase available bandwidth.
* Provide link redundancy.
* Reduce the impact of individual link failures.
* Create a logical link from multiple physical links.

Planned implementation:

```text
SW1-CORE
   ║
   ║ EtherChannel
   ║
SW2-CORE
```

---

# Spanning Tree Protocol

STP will be used to prevent Layer 2 switching loops.

The project will include STP optimization and security mechanisms such as:

* PortFast
* BPDU Guard
* Appropriate root bridge selection

---

# OSPF

OSPF will be implemented as the dynamic routing protocol.

The objective is to allow routers and multilayer switches to dynamically exchange routing information.

The implementation will include:

* OSPF process configuration
* Router IDs
* Network advertisements
* Appropriate passive interfaces
* Routing table verification

---

# DHCP

DHCP will be implemented to provide automatic IP configuration to end devices.

DHCP will provide:

* IP address
* Subnet mask
* Default gateway
* DNS information

Different DHCP scopes will be created for the appropriate VLANs.

---

# NAT / PAT

NAT/PAT will be implemented on the edge routers to allow private internal networks to access the external network.

The implementation will include:

* Inside interfaces
* Outside interfaces
* NAT rules
* PAT using the external interface
* Connectivity testing

---

# Network Security

Security will be progressively implemented at both Layer 2 and Layer 3.

## Device Hardening

Planned controls include:

* Local user accounts
* Enable secret
* Password encryption
* SSH
* Secure management access
* Login protection
* MOTD banner
* Unused port shutdown

---

## Port Security

Port Security will be implemented on access ports to restrict unauthorized devices.

Planned controls include:

* Maximum allowed MAC addresses
* Sticky MAC addresses
* Violation actions
* Verification of learned MAC addresses

---

## DHCP Snooping

DHCP Snooping will be implemented to protect the network against unauthorized DHCP servers.

The project will distinguish between:

* Trusted interfaces
* Untrusted interfaces

---

## BPDU Guard

BPDU Guard will be used on appropriate edge ports to protect the STP topology against unauthorized switches.

---

# Access Control Lists

ACLs will be used to control communication between network segments.

The security policy will include restrictions such as:

```text
GUEST → Internal Servers       DENY
GUEST → Administration        DENY
GUEST → Human Resources       DENY
GUEST → IT                    DENY
GUEST → Internet              ALLOW
```

Additional ACL rules will be implemented according to the final security requirements.

---

# Testing & Validation

Each implementation phase will be tested before moving to the next phase.

## VLAN Verification

```text
show vlan brief
```

## Trunk Verification

```text
show interfaces trunk
```

## SVI Verification

```text
show ip interface brief
```

## HSRP Verification

```text
show standby brief
```

## Routing Verification

```text
show ip route
```

## OSPF Verification

```text
show ip ospf neighbor
```

## EtherChannel Verification

```text
show etherchannel summary
```

## STP Verification

```text
show spanning-tree
```

## Connectivity Testing

```text
ping <destination-ip>
```

## Path Testing

```text
traceroute <destination-ip>
```

---

# Security Testing

After implementing the security controls, controlled tests will be performed to verify that:

* Unauthorized devices cannot access protected switch ports.
* Guest devices cannot access restricted internal networks.
* Unauthorized DHCP servers are blocked.
* STP manipulation attempts are mitigated.
* Management access is restricted to authorized users.
* ACL rules correctly allow and deny traffic.

All tests will be performed within the simulated laboratory environment.

---

# Project Roadmap

## Day 1 — Network Foundation

* [x] Network topology
* [x] Device deployment
* [x] VLAN creation
* [x] Access port configuration
* [x] Trunk configuration
* [x] SVI configuration
* [x] Inter-VLAN routing
* [x] HSRP
* [x] Initial IP addressing
* [x] Initial connectivity tests

## Day 2 — Layer 2 Redundancy

* [ ] EtherChannel
* [ ] STP
* [ ] Root bridge configuration
* [ ] PortFast
* [ ] BPDU Guard
* [ ] Layer 2 verification

## Day 3 — Dynamic Routing

* [ ] OSPF
* [ ] Router IDs
* [ ] OSPF neighbors
* [ ] Routing table verification
* [ ] Route failover testing

## Day 4 — Network Services

* [ ] DHCP
* [ ] DHCP relay
* [ ] DNS / server configuration
* [ ] NAT/PAT
* [ ] Internet connectivity

## Day 5 — Device Security

* [ ] SSH
* [ ] Local authentication
* [ ] Password protection
* [ ] Secure management
* [ ] Unused-port shutdown

## Day 6 — Layer 2 Security

* [ ] Port Security
* [ ] DHCP Snooping
* [ ] BPDU Guard
* [ ] PortFast
* [ ] Security verification

## Day 7 — Traffic Security

* [ ] ACL design
* [ ] Inter-VLAN filtering
* [ ] Guest isolation
* [ ] Server protection
* [ ] ACL testing

## Final Phase — Testing & Documentation

* [ ] Full connectivity testing
* [ ] HSRP failover testing
* [ ] EtherChannel failure testing
* [ ] OSPF failure testing
* [ ] Security testing
* [ ] Configuration backup
* [ ] Final topology
* [ ] Final documentation
* [ ] Final security assessment

---

# Repository Structure

```text
secure-enterprise-network/
│
├── README.md
│
├── packet-tracer/
│   ├── secure-enterprise-network-day1.pkt
│   ├── secure-enterprise-network-day2.pkt
│   ├── secure-enterprise-network-day3.pkt
│   └── secure-enterprise-network-final.pkt
│
├── configs/
│   ├── day1/
│   │   ├── R1-EDGE.txt
│   │   ├── R2-EDGE.txt
│   │   ├── SW1-CORE.txt
│   │   ├── SW2-CORE.txt
│   │   ├── SW3-ACCESS.txt
│   │   └── SW4-ACCESS.txt
│   │
│   ├── day2/
│   ├── day3/
│   └── final/
│
├── docs/
│   ├── architecture/
│   ├── day1/
│   ├── day2/
│   ├── day3/
│   └── final/
│
├── screenshots/
│   ├── topology/
│   ├── vlan/
│   ├── hsrp/
│   ├── ospf/
│   └── security/
│
└── tests/
    ├── day1/
    ├── day2/
    ├── day3/
    └── final/
```

---

# Technologies

### Networking

* Cisco IOS
* Cisco Packet Tracer
* VLAN
* 802.1Q
* Inter-VLAN Routing
* HSRP
* EtherChannel
* STP
* OSPF
* DHCP
* NAT/PAT

### Cybersecurity

* SSH
* Device Hardening
* Port Security
* DHCP Snooping
* BPDU Guard
* ACL
* Network Segmentation
* Access Control
* Security Testing

---

# Skills Demonstrated

This project demonstrates practical skills in:

* Network architecture
* Cisco device configuration
* VLAN segmentation
* Layer 2 switching
* Layer 3 routing
* High availability
* Network redundancy
* Dynamic routing
* Network services
* Network security
* Troubleshooting
* Documentation
* Technical testing

---

# Project Progress

| Component           | Status    |
| ------------------- | --------- |
| Network Topology    | Completed |
| VLAN Segmentation   | Completed |
| Trunking            | Completed |
| SVI Configuration   | Completed |
| Inter-VLAN Routing  | Completed |
| HSRP                | Completed |
| EtherChannel        | Planned   |
| STP                 | Planned   |
| OSPF                | Planned   |
| DHCP                | Planned   |
| NAT/PAT             | Planned   |
| SSH Hardening       | Planned   |
| Port Security       | Planned   |
| DHCP Snooping       | Planned   |
| BPDU Guard          | Planned   |
| ACL                 | Planned   |
| Security Testing    | Planned   |
| Final Documentation | Planned   |

---

# Evidence

Screenshots and verification outputs are stored in the repository to document the implementation and testing of each phase.

Examples include:

* Network topology
* VLAN configuration
* Trunk status
* HSRP status
* Routing tables
* OSPF neighbors
* EtherChannel status
* STP status
* ACL verification
* Security test results

---

# Author

**Salimata WELE**

Telecommunications & IT Student
Networking & Cybersecurity

---

# Project Status

**In Progress**

Day 1 — Network Foundation completed.

The project will progressively evolve from a basic segmented network into a redundant, routed, secured, and fully documented enterprise infrastructure.

---
