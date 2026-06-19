# 42 - NetPractice

> A practical networking exercise to understand TCP/IP addressing, subnetting, and routing.

---

## Table of Contents

- [What is NetPractice?](#what-is-netpractice)
- [Key Concepts](#key-concepts)
  - [IP Address](#ip-address)
  - [Subnet Mask](#subnet-mask)
  - [Network Address & Broadcast Address](#network-address--broadcast-address)
  - [CIDR Notation](#cidr-notation)
  - [Usable Hosts](#usable-hosts)
  - [Default Gateway](#default-gateway)
  - [Routing Table](#routing-table)
  - [Switch vs Router](#switch-vs-router)
- [Subnetting Cheat Sheet](#subnetting-cheat-sheet)
- [How to Approach Each Level](#how-to-approach-each-level)
- [Common Mistakes](#common-mistakes)

---

## What is NetPractice?

NetPractice is a 42 school project consisting of 10 levels, each presenting a network diagram with misconfigured or incomplete devices. The goal is to configure IP addresses, subnet masks, and routing tables so that all devices in the diagram can communicate correctly.

There is no coding involved — the project is purely about understanding how TCP/IP networks work.

---

## Key Concepts

### IP Address

An IP address (IPv4) is a 32-bit number divided into four octets, written in dotted-decimal notation (e.g. `192.168.1.1`). Each octet ranges from 0 to 255.

An IP address has two parts:
- The **network part** — identifies which network the device belongs to.
- The **host part** — identifies the specific device within that network.

The boundary between these two parts is defined by the subnet mask.

---

### Subnet Mask

A subnet mask is also a 32-bit number. It uses consecutive `1` bits to mark the network part and `0` bits for the host part.

| Decimal | Binary |
|---|---|
| `255.255.255.0` | `11111111.11111111.11111111.00000000` |
| `255.255.0.0` | `11111111.11111111.00000000.00000000` |
| `255.255.255.128` | `11111111.11111111.11111111.10000000` |

To find whether two IP addresses are on the **same network**, apply a bitwise AND between each address and the subnet mask. If the results are identical, they share the same network.

---

### Network Address & Broadcast Address

Given an IP address and a subnet mask, two addresses are reserved and cannot be assigned to a host:

- **Network address** — the first address of the range. All host bits are `0`. It identifies the network itself.
- **Broadcast address** — the last address of the range. All host bits are `1`. Used to send a message to all hosts on the network.

**Example** with `192.168.1.0 / 255.255.255.0`:

| Address | Value |
|---|---|
| Network address | `192.168.1.0` |
| First usable host | `192.168.1.1` |
| Last usable host | `192.168.1.254` |
| Broadcast address | `192.168.1.255` |

---

### CIDR Notation

CIDR (Classless Inter-Domain Routing) notation expresses the subnet mask as a suffix indicating how many bits are set to `1`.

| CIDR | Subnet Mask | Number of addresses |
|---|---|---|
| `/24` | `255.255.255.0` | 256 |
| `/25` | `255.255.255.128` | 128 |
| `/26` | `255.255.255.192` | 64 |
| `/27` | `255.255.255.224` | 32 |
| `/28` | `255.255.255.240` | 16 |
| `/30` | `255.255.255.252` | 4 |

`/30` is commonly used for point-to-point links between two routers, as it provides exactly 2 usable host addresses.

---

### Usable Hosts

The number of usable host addresses in a subnet is calculated as:

**2^(32 − prefix) − 2**

The `− 2` accounts for the network address and the broadcast address.

| CIDR | Total addresses | Usable hosts |
|---|---|---|
| `/24` | 256 | 254 |
| `/25` | 128 | 126 |
| `/26` | 64 | 62 |
| `/27` | 32 | 30 |
| `/28` | 16 | 14 |
| `/30` | 4 | 2 |

---

### Default Gateway

The default gateway is the IP address of the router interface that a device uses to reach destinations outside its own subnet. It must be on the same network as the device it serves.

A device sends traffic to its default gateway whenever the destination IP does not belong to its local subnet. The router then forwards the packet toward its destination.

---

### Routing Table

A routing table is a set of rules used by a router (or a host) to determine where to forward packets. Each entry contains:

- **Destination** — the target network (in CIDR or address/mask notation). Use `0.0.0.0/0` as a catch-all default route.
- **Next hop / Gateway** — the IP address of the next router to send the packet to, or `0.0.0.0` if the destination is directly reachable.

Routers match the destination address against their routing table entries. The most specific matching route (longest prefix) wins. If no specific route matches, the default route (`0.0.0.0/0`) is used.

---

### Switch vs Router

| Device | Role |
|---|---|
| **Switch** | Connects multiple devices within the **same network**. Operates at Layer 2 (MAC addresses). Does not separate networks. |
| **Router** | Connects **different networks** together. Operates at Layer 3 (IP addresses). Forwards packets between networks based on routing tables. |

In NetPractice, a switch simply groups devices — it does not require IP configuration. A router has one interface per connected network, and each interface must have an IP address belonging to that network.

---

## Subnetting Cheat Sheet

To quickly find the network address from any IP and mask:

1. Convert both the IP and the mask to binary.
2. Perform a bitwise AND operation bit by bit.
3. The result is the network address.

To find the broadcast address:

1. Take the network address in binary.
2. Set all host bits (the `0` bits of the mask) to `1`.
3. Convert back to decimal.

**Quick reference — last octet subnetting:**

| CIDR suffix | Mask last octet | Block size | Network boundaries |
|---|---|---|---|
| `/25` | 128 | 128 | 0, 128 |
| `/26` | 192 | 64 | 0, 64, 128, 192 |
| `/27` | 224 | 32 | 0, 32, 64, 96, 128, 160, 192, 224 |
| `/28` | 240 | 16 | 0, 16, 32, 48, … |
| `/29` | 248 | 8 | 0, 8, 16, 24, … |
| `/30` | 252 | 4 | 0, 4, 8, 12, … |

The **block size** is `256 − mask last octet`. Network addresses are always multiples of the block size.

---

## How to Approach Each Level

1. **Identify all networks** — each link between a router interface and a group of devices is a separate network.
2. **Assign subnet masks first** — decide how many hosts each network needs and pick an appropriate mask.
3. **Set IP addresses** — make sure every device on the same network shares the same network address, and that no two devices have the same IP.
4. **Set default gateways** — each host must point to the router interface on its own network.
5. **Fill in routing tables** — for each router, add routes to all networks it cannot reach directly. Use `0.0.0.0/0` as a default route when a single exit point handles all unknown traffic.
6. **Verify connectivity** — mentally trace the path of a packet between two hosts and confirm every hop has a valid route.

---

## Common Mistakes

- **Overlapping subnets** — two different networks using the same address range. Always verify that no two subnets share any addresses.
- **Wrong default gateway** — the gateway IP must be the router interface on the same subnet as the host, not on another network.
- **Assigning a network or broadcast address to a host** — the first and last addresses of a subnet are reserved and cannot be used by a device.
- **Missing return routes** — a router can send a packet to a destination but the destination has no route back. Both directions must be configured.
- **Interfaces on the same router in the same subnet** — each router interface must belong to a different network.
- **Forgetting the default route** — when a router has a single uplink, use `0.0.0.0/0` pointing to that uplink instead of listing every possible destination.

---

*Project completed as part of the 42 school curriculum — NetPractice.*
