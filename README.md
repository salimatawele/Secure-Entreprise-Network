# 🔐 Secure Enterprise Network

> Enterprise Network Infrastructure & Security Project — Cisco Packet Tracer

## 📌 Project Overview

This project focuses on the design, implementation, and progressive security hardening of a small enterprise network using **Cisco Packet Tracer**.

The objective is to build a structured, segmented, redundant, and secure network infrastructure while applying practical networking and cybersecurity concepts.

The project is being developed progressively, with each phase documented and tested before moving to the next one.

---

## 🎯 Objectives

The main objectives of this project are to:

- Design a realistic enterprise network topology
- Segment the network using VLANs
- Configure 802.1Q trunking
- Implement inter-VLAN routing using multilayer switches
- Provide gateway redundancy using HSRP
- Implement dynamic routing using OSPF
- Configure EtherChannel and Spanning Tree Protocol
- Implement DHCP and NAT/PAT
- Secure network devices using SSH and authentication mechanisms
- Implement Layer 2 and Layer 3 security controls
- Control network traffic using ACLs
- Perform connectivity and security testing
- Document the complete infrastructure and its configurations

---

# 🏗️ Network Architecture

The network is designed using a hierarchical architecture consisting of:

- 2 Edge Routers
- 2 Multilayer Core Switches
- 2 Access Switches
- 1 ISP Router
- 1 Internet/Cloud representation
- 8 Client PCs
- 1 Server

### High-Level Topology

```text
                         INTERNET
                            |
                          [ISP]
                         /     \
                        /       \
                  [R1-EDGE]   [R2-EDGE]
                     |           |
                     |           |
                [SW1-CORE]====[SW2-CORE]
                   ||             ||
                   ||             ||
             [SW3-ACCESS]   [SW4-ACCESS]
              /  /  |  \     / / | \  \
             PC PC PC PC   PC PC PC PC SRV1
