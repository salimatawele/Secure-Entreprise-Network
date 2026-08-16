# Day 1 — Network Foundation

> **Project:** Secure Enterprise Network  
> **Platform:** Cisco Packet Tracer  
> **Phase:** Day 1 — Network Foundation  
> **Status:** Completed

---

## 1. Objective

The objective of Day 1 was to establish the foundation of the enterprise network infrastructure.

The main focus was to implement the basic network architecture, create logical network segmentation using VLANs, configure trunk links, enable inter-VLAN routing, and provide gateway redundancy using HSRP.

At the end of this phase, the network should provide a functional and redundant Layer 2 and Layer 3 infrastructure.

---

## 2. Network Topology

The Day 1 topology consists of:

- 2 multilayer core switches
- 2 access switches
- 2 edge routers
- 1 ISP router
- 8 client PCs
- 1 server
- 1 Internet/Cloud representation

The core switches provide Layer 3 routing and gateway redundancy, while the access switches provide connectivity to end devices.

---

## 3. VLAN Implementation

The network was segmented into multiple VLANs according to departments and network functions.

| VLAN | Name | Network | Virtual Gateway |
|------|------|---------|-----------------|
| 10 | ADMIN | 192.168.10.0/24 | 192.168.10.1 |
| 20 | RH | 192.168.20.0/24 | 192.168.20.1 |
| 30 | IT | 192.168.30.0/24 | 192.168.30.1 |
| 40 | SERVERS | 192.168.40.0/24 | 192.168.40.1 |
| 50 | GUEST | 192.168.50.0/24 | 192.168.50.1 |
| 99 | MANAGEMENT | 192.168.99.0/24 | 192.168.99.1 |

### VLAN Roles

- **VLAN 10 — ADMIN:** Administration department
- **VLAN 20 — RH:** Human Resources department
- **VLAN 30 — IT:** IT department
- **VLAN 40 — SERVERS:** Internal server network
- **VLAN 50 — GUEST:** Guest network
- **VLAN 99 — MANAGEMENT:** Network infrastructure management

---

## 4. Access Port Configuration

Access ports were assigned to the appropriate VLAN according to the role of each connected device.

Examples include:

- Administration PCs → VLAN 10
- Human Resources PCs → VLAN 20
- IT PCs → VLAN 30
- Server → VLAN 40
- Guest PCs → VLAN 50

This configuration provides logical separation between the different departments.

---

## 5. Trunk Configuration

802.1Q trunking was configured on the links connecting the switches.

The main trunk connections are:

```text
SW1-CORE ↔ SW2-CORE
SW1-CORE ↔ SW3-ACCESS
SW2-CORE ↔ SW4-ACCESS
