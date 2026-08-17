# Multi-Protocol Route Redistribution Lab (OSPF ↔ EIGRP ↔ RIPv2 via BGP)

A Cisco Packet Tracer lab demonstrating route redistribution across three separate routing domains — **OSPF**, **EIGRP**, and **RIPv2** — through a central router acting as a redistribution point running **BGP**.


## Overview

This project simulates a realistic enterprise/ISP-style scenario where three independently-managed routing domains, each running a different Interior Gateway Protocol (IGP), need to exchange routes through a common core router. Rather than running one IGP everywhere, the core router (`r0`) redistributes routes between OSPF, EIGRP, and RIPv2, using BGP as its own routing process, so that end-to-end reachability exists between all three LANs.

**Goals demonstrated:**
- Configuring and verifying OSPF, EIGRP, and RIPv2 in isolated domains
- Configuring BGP on a central router
- Redistributing routes between four routing protocols (OSPF, EIGRP, RIPv2, BGP) on a single router
- Setting appropriate seed metrics for each redistribution (since OSPF, EIGRP, and RIP calculate metrics differently and none of them can auto-derive a metric from BGP or from each other)
- Verifying full end-to-end connectivity across all three domains

## Topology

| Router | Hostname | Routing Protocol | Connects To | LAN |
|---|---|---|---|---|
| Router 1 | `OSPF` | OSPF | r0 (Gig0/0 ↔ Gig0/1) | Switch (Sw) — PC0 |
| Router 2 | `EIGRP` | EIGRP | r0 (Gig0/1 ↔ Gig0/1) | Switch0 — PC1 |
| Router 3 | `RIPversion2` | RIPv2 | r0 (Gig0/2 ↔ Gig0/1) | Switch2 — PC2 |
| Router 4 (core) | `r0` | BGP + redistribution | All three edge routers | — |

`r0` is the ASBR-equivalent of this lab: it runs BGP as its own process and redistributes routes into and out of OSPF, EIGRP, and RIPv2 so that PC0, PC1, and PC2 (each on a different IGP) can all reach one another.

## IP Addressing

| Device | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| OSPF router | Gig0/0 (LAN) | 192.168.0.1 | 255.255.255.0 |
| OSPF router | Gig0/1 (to r0) | 10.0.13.2 | 255.255.255.252 |
| EIGRP router | Gig0/0 (LAN) | 192.168.1.1 | 255.255.255.0 |
| EIGRP router | Gig0/1 (to r0) | 10.0.12.2 |255.255.255.252 |
| RIPversion2 router | Gig0/0 (LAN) | 192.168.2.1 | 255.255.255.0 |
| RIPversion2 router | Gig0/1 (to r0) | 10.0.14.2 | 255.255.255.252 |
| r0 | Gig0/0 (to OSPF) | 10.0.13.1  | 255.255.255.252 |
| r0 | Gig0/1 (to EIGRP) | 10.0.12.1 | 255.255.255.252 |
| r0 | Gig0/2 (to RIPv2) | 10.0.14.1 | 255.255.255.252 |
| PC0 / PC1 / PC2 | Fa0 | | |

## Configuration Summary

### Edge routers (single protocol each)
- **OSPF router** — `router ospf 1`, network statements for its LAN and the link to r0
- **EIGRP router** — `router eigrp 200` (or named EIGRP), network statements for its LAN and the link to r0
- **RIPversion2 router** — `router rip`, `version 2`, `no auto-summary`, network statements for its LAN and the link to r0

### Core router (r0) — redistribution
```
router ospf 1
 log-adjacency-changes
 redistribute rip subnets 
 redistribute eigrp 200 metric 30 subnets 
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.13.0 0.0.0.3 area 0
 network 10.0.14.0 0.0.0.3 area 0

router eigrp 200
 redistribute rip metric 10000 1 255 1 1500 
 redistribute ospf 1 metric 10000 100 255 1 1500 
 network 10.0.12.0 0.0.0.3
 network 10.0.13.0 0.0.0.3
 network 10.0.14.0 0.0.0.

router rip
 version 2
 redistribute eigrp 200 metric 5 
 redistribute ospf 1 metric 3 
 network 10.0.0.0
 no auto-summary
```
> Replace with your actual redistribution commands, seed metrics, and any route-maps or distribute-lists used to prevent redistribution/routing loops.

## Verification

Commands used to confirm the redistribution worked end-to-end:

```
show ip route
show ip protocols
ping <remote-LAN-PC-IP>
tracert <remote-LAN-PC-IP>
```

- [ ] PC0 (OSPF domain) can ping PC1 (EIGRP domain)
- [ ] PC0 (OSPF domain) can ping PC2 (RIPv2 domain)
- [ ] PC1 (EIGRP domain) can ping PC2 (RIPv2 domain)
- [ ] `show ip route` on each edge router shows routes learned from the other two domains via redistribution

## Files

| File | Description |
|---|---|
| `redistribution.pkt` | Full Packet Tracer project file |
| `topology.png` | Topology screenshot |
| `configs/` | Exported running-configs for each router (`.txt`) |

## Tools Used

- Cisco Packet Tracer
- OSPF, EIGRP, RIPv2, BGP

## Author

Jetlee Muriuki — Nairobi, Kenya
