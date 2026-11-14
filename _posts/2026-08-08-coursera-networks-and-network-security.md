---
title: "Google Cybersecurity: Connect and Protect — Networks and Network Security"
author: parthpandey
categories: [Coursera, Google-Cybersecurity]
tags: [course-notes, network-security, tcp-ip, firewalls, vpn, network-protocols]
render_with_liquid: false
date: 2026-08-08 14:15:00 +0530
---

## Course Overview

| Detail | Value |
|---|---|
| **Platform** | Coursera |
| **Certificate** | Google Cybersecurity Professional Certificate |
| **Course** | 3 of 8 — Connect and Protect: Networks and Network Security |
| **Duration** | ~14 hours |
| **Status** | ✅ Completed |

---

## Executive Summary

Course 3 explores the transmission media, protocols, and security devices that govern digital communications across enterprise networks and the public Internet. Through hands-on analysis of packet traffic in Wireshark and tcpdump, I examined the mechanics of the **TCP/IP vs. OSI 7-Layer models**, inspected packet encapsulation and header fields, evaluated perimeter network segmentation (DMZ, Zero Trust microsegmentation), and audited stateful firewalls, proxies, and Intrusion Detection Systems (IDS/IPS).

---

## Technical Foundations: The Protocol Stack

### 1. OSI 7-Layer vs. TCP/IP 4-Layer Architecture

```text
┌───────────────────────────┬─────────────────────────────────────────────────┐
│     OSI 7-LAYER MODEL     │                TCP/IP 4-LAYER MODEL             │
├───────────────────────────┼─────────────────────────────────────────────────┤
│ Layer 7: Application      │                                                 │
│ Layer 6: Presentation     │  Application Layer (HTTP/S, DNS, SSH, DHCP)     │
│ Layer 5: Session          │                                                 │
├───────────────────────────┼─────────────────────────────────────────────────┤
│ Layer 4: Transport        │  Transport Layer (TCP, UDP)                     │
├───────────────────────────┼─────────────────────────────────────────────────┤
│ Layer 3: Network          │  Internet Layer (IPv4, IPv6, ICMP, ARP, IPsec)  │
├───────────────────────────┼─────────────────────────────────────────────────┤
│ Layer 2: Data Link        │  Network Access / Link Layer                    │
│ Layer 1: Physical         │  (Ethernet, 802.11 Wi-Fi, MAC addresses)        │
└───────────────────────────┴─────────────────────────────────────────────────┘
```

#### Protocol Data Units (PDUs) & Encapsulation:
$$\text{Data (L7)} \xrightarrow{\text{Add TCP Header}} \text{Segment (L4)} \xrightarrow{\text{Add IP Header}} \text{Packet (L3)} \xrightarrow{\text{Add MAC Header/Trailer}} \text{Frame (L2)} \xrightarrow{\text{Encoding}} \text{Bits (L1)}$$

---

### 2. Deep Dive: Core Protocols and Attack Vectors

* **Address Resolution Protocol (ARP)**:
  * Translates Layer 3 IP addresses to Layer 2 MAC addresses via broadcast requests.
  * *Vulnerability*: Lacks cryptographic validation, enabling **ARP Spoofing / Cache Poisoning** (allowing MitM traffic interception on local broadcast domains).
  * *Defense*: Dynamic ARP Inspection (DAI) on managed switches and static ARP binding.
* **Domain Name System (DNS)**:
  * Resolves hostnames to IP addresses over UDP port 53.
  * *Vulnerability*: Cleartext queries allow DNS spoofing, cache poisoning, and covert command-and-control (C2) data exfiltration via DNS tunneling.
  * *Defense*: DNSSEC (cryptographic signing of zone records), DNS-over-HTTPS (DoH), and SIEM entropy checks on subdomain query lengths.
* **Transmission Control Protocol (TCP)**:
  * Connection-oriented protocol providing reliable, ordered byte-stream delivery via the **Three-Way Handshake**:
    $$\text{Client} \xrightarrow{\text{SYN}} \text{Server} \xrightarrow{\text{SYN-ACK}} \text{Client} \xrightarrow{\text{ACK}} \text{Server}$$
  * *Vulnerability*: **SYN Flood Attacks** (exhausting the server's embryonic connection backlog table).
  * *Defense*: TCP SYN Cookies, firewall connection rate limits, and aggressive half-open timeouts.
* **User Datagram Protocol (UDP)**:
  * Connectionless, lightweight protocol with zero connection handshake or flow control.
  * *Vulnerability*: Highly prone to **IP Spoofing & Amplification DDoS** (e.g., NTP, DNS, and SNMP reflection attacks).

---

### 3. Enterprise Perimeter Defense Architecture

```text
               Public Internet (Untrusted - Sec 0)
                                │
                                ▼
               ┌─────────────────────────────────┐
               │  Edge Firewall / Border Router  │
               └────────────────┬────────────────┘
                                │
                                ▼
               ┌─────────────────────────────────┐
               │    Demilitarized Zone (DMZ)     │
               │  - Reverse Proxy (Nginx)        │
               │  - Public Web Portal (Port 443) │
               └────────────────┬────────────────┘
                                │ (Stateful Inspection / IDS)
                                ▼
               ┌─────────────────────────────────┐
               │    Internal Gateway Firewall    │
               └────────────────┬────────────────┘
                                │
                                ▼
               ┌─────────────────────────────────┐
               │ Corporate LAN (Trusted - Sec 100│
               │ - Active Directory Domain Ctrl  │
               │ - Internal Production Databases │
               └─────────────────────────────────┘
```

#### Firewall Classification:
1. **Packet Filtering (Stateless)**: Inspects source/destination IP, port, and protocol against static ACLs. Does not track connection state (easily bypassed by crafted ACK packets).
2. **Stateful Inspection**: Maintains a state table of active TCP sessions (`ESTABLISHED`, `RELATED`). Automatically allows reply traffic while dropping unsolicited ingress packets.
3. **Next-Generation Firewalls (NGFW)**: Inspects up to Layer 7 (Application layer), identifying applications regardless of port (App-ID), enforcing user identity (User-ID), and decrypting SSL/TLS streams to scan for malware payloads.

---

## Practical Labs & Exercises

### Lab 1: Wireshark Packet Inspection & Stream Reconstruction
* **Objective**: Isolate cleartext credential transmission and investigate anomalous protocol behavior within a synthetic packet capture (`.pcap`).
* **Filters Applied**:
  * `tcp.flags.syn == 1 && tcp.flags.ack == 0`: Identifies aggressive SYN port-scanning behavior.
  * `http.request.method == "POST"`: Isolates login attempts; extracted cleartext authentication payloads transmitted over unencrypted HTTP port 80.
  * `dns.flags.response == 0 && frame.len > 150`: Uncovered DNS tunneling queries carrying base64-encoded file snippets.
* **Outcome**: Documented proof of data exfiltration and recommended enforcing mandatory HTTPS with HSTS and internal DNS query logging.

### Lab 2: Network Hardening & Zero Trust Segmentation
* **Objective**: Design an internal segmentation policy preventing lateral movement between accounting workstations and production database clusters.
* **Implementation**:
  * Defined microsegmentation zones using 802.1Q VLAN tags.
  * Configured deny-by-default access control lists (ACLs) dropping all inter-VLAN routing except authenticated database traffic over port 5432 originating from verified application IP addresses.

---

## Key Takeaways

1. **Visibility Precedes Defense**: You cannot defend what you cannot see; continuous packet inspection and flow log ingestion provide ground truth for security investigations.
2. **Never Trust, Always Verify**: Perimeter firewalls are insufficient alone. Modern architectures mandate Zero Trust Network Access (ZTNA) where identity, device health, and context are validated on every request.
3. **Defense at Layers 2 through 7**: Robust security requires hardening every layer of the stack—from port security and 802.1X at Layer 2 to strict TLS and WAF rules at Layer 7.
