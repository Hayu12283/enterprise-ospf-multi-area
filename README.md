# enterprise-ospf-multi-area
# Enterprise Network with OSPF Multi-Area Routing

## Executive Overview
This project models an enterprise multi-area OSPFv2 infrastructure designed to connect a corporate headquarters (`HQ-RTR`), a central Data Center (`DC-RTR`), and two remote branch offices (`BR1-RTR` and `BR2-RTR`). 

By enforcing a multi-area hierarchy, link-state advertisement (LSA) flooding is isolated within specific areas, minimizing memory overhead, optimizing SPF calculations, and securing routing updates across the backbone.

---

## Architectural Topology & Addressing Matrix

### OSPF Area Topology
* **Area 0 (Backbone):** Connects `HQ-RTR`, `DC-RTR`, `BR1-RTR`, and `BR2-RTR`.
* **Area 10 (Data Center):** Connects core server subnets behind `DC-RTR` (ABR).
* **Area 20 (Branch 1):** Connects user subnets behind `BR1-RTR` (ABR).
* **Area 30 (Branch 2):** Connects user subnets behind `BR2-RTR` (ABR).

### Subnet Addressing Table

| Device Name | Interface | IP Address / Mask | OSPF Area | Description / Role |
| :--- | :--- | :--- | :--- | :--- |
| **`HQ-RTR`** | `Loopback0`<br/>`Gi0/0/0`<br/>`Gi0/0/1` | `1.1.1.1/32`<br/>`10.0.0.1/30`<br/>`10.0.0.5/30` | Area 0<br/>Area 0<br/>Area 0 | Router ID / Core Backbone<br/>Link to DC-RTR<br/>Link to BR1-RTR |
| **`DC-RTR`** | `Loopback0`<br/>`Gi0/0/0`<br/>`Gi0/0/1`<br/>`Gi0/0/2` | `2.2.2.2/32`<br/>`10.0.0.2/30`<br/>`10.0.0.9/30`<br/>`172.16.10.1/24` | Area 0<br/>Area 0<br/>Area 0<br/>Area 10 | Router ID / Backbone ABR<br/>Link to HQ-RTR<br/>Link to BR2-RTR<br/>Data Center Server LAN Gateway |
| **`BR1-RTR`** | `Loopback0`<br/>`Gi0/0/0`<br/>`Gi0/0/1` | `3.3.3.3/32`<br/>`10.0.0.6/30`<br/>`192.168.10.1/24` | Area 0<br/>Area 0<br/>Area 20 | Router ID / Branch 1 ABR<br/>Link to HQ-RTR<br/>Branch 1 User LAN Gateway |
| **`BR2-RTR`** | `Loopback0`<br/>`Gi0/0/0`<br/>`Gi0/0/1` | `4.4.4.4/32`<br/>`10.0.0.10/30`<br/>`192.168.20.1/24` | Area 0<br/>Area 0<br/>Area 30 | Router ID / Branch 2 ABR<br/>Link to DC-RTR<br/>Branch 2 User LAN Gateway |

---

## Core Engineering Features Implemented

1. **MD5 Message-Digest Authentication:** Enforced across all Area 0 transit links (`ip ospf message-digest-key 1 md5`) to block rogue router insertion.
2. **Passive Interfaces:** Configured on user and server LAN gateways (`passive-interface Gi0/0/1` / `Gi0/0/2`) to suppress unnecessary OSPF multicast Hello packets and prevent network recon.
3. **Route Summarization (ABR):** Implemented inter-area route aggregation on `DC-RTR` (`area 10 range 172.16.0.0 255.255.0.0`) to keep Area 0 routing tables compact.
4. **Deterministic Router IDs:** Hardcoded via dedicated `/32` Loopback interfaces for reliable adjacency formation and election predictability.

---

## Operational Verification

### 1. Verify OSPF Neighbor Adjacency
Executed on `HQ-RTR`:
```text
HQ-RTR# show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           0   FULL/ -         00:00:34    10.0.0.2        GigabitEthernet0/0/0
3.3.3.3           0   FULL/ -         00:00:38    10.0.0.6        GigabitEthernet0/0/1
