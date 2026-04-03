# Network Traffic Analysis with Wireshark

This repository documents a set of packet-analysis investigations across HTTP, TCP, UDP, NAT, Ethernet, and ARP traffic. The focus is not on lab completion checkboxes—it is on reading packets like evidence, explaining what fields mean, and connecting captures to real troubleshooting and security questions.

## What This Repository Demonstrates

- Packet capture and focused filtering in Wireshark
- Interpreting HTTP request/response behavior at packet level
- Reading TCP and UDP behavior through stream and retransmission evidence
- Explaining source/destination addressing across IP and Ethernet layers
- Reasoning about NAT effects when traffic is observed at different network points
- Understanding local-LAN behavior through Ethernet and ARP frames
- Turning packet observations into practical analyst conclusions

## How to Read This Project

Each lab section is presented as a mini investigation:

1. **Scenario** – what traffic was being examined
2. **Method / Filters Used** – how traffic was isolated
3. **Key Findings** – strongest packet-level observations
4. **Why It Matters** – operational or security relevance
5. **Evidence** – supporting screenshots

---

## Key Networking Concepts Explained

### Packet capture
- **Plain English:** A packet capture is a time-ordered record of what devices are sending and receiving.
- **Why it matters:** It shows what actually happened on the wire, not what an app *thinks* happened.
- **Analogy:** Like CCTV footage for network traffic.

### Protocol filtering
- **Plain English:** Filters reduce noise so you can focus on one behavior (for example `http`, `arp`, or one IP).
- **Why it matters:** Good filtering turns large traces into fast root-cause analysis.
- **Analogy:** Like searching a long chat history by keyword instead of reading everything.

### Encapsulation
- **Plain English:** Data is wrapped in layers (application, transport, IP, Ethernet), each adding routing/delivery info.
- **Why it matters:** Troubleshooting depends on knowing which layer is responsible for a failure.
- **Analogy:** A smaller envelope inside a larger labeled envelope inside a shipping box.

### TCP vs UDP
- **Plain English:** TCP is connection-oriented and reliability-focused; UDP is lightweight and best-effort.
- **Why it matters:** TCP gives ordering/retransmission; UDP reduces overhead and latency.
- **Analogy:** TCP is certified mail with confirmations; UDP is a postcard.

### TCP handshake
- **Plain English:** TCP starts with SYN → SYN/ACK → ACK before data transfer.
- **Why it matters:** Handshake failures quickly point to reachability, firewall, or service issues.
- **Analogy:** Introducing yourself before starting a serious conversation.

### RTT (Round-Trip Time)
- **Plain English:** RTT is how long it takes for a packet and its response to complete one round trip.
- **Why it matters:** High RTT can hurt user experience even when packets are not dropped.
- **Analogy:** Asking a question and timing how long it takes to hear back.

### Source port vs destination port
- **Plain English:** Source port identifies the client-side socket; destination port identifies the target service.
- **Why it matters:** Port direction helps separate client behavior from server behavior in mixed traffic.
- **Analogy:** Caller extension vs the department extension dialed.

### NAT (Network Address Translation)
- **Plain English:** NAT rewrites addresses (and sometimes ports) between private and public sides.
- **Why it matters:** Captures from two vantage points can look inconsistent unless NAT is accounted for.
- **Analogy:** A front desk forwarding replies back to the correct internal person.

### Ethernet frames
- **Plain English:** Ethernet is local-link delivery with source/destination MAC and EtherType fields.
- **Why it matters:** It explains local-hop forwarding and gateway behavior.
- **Analogy:** Local delivery label used only within one neighborhood.

### MAC addresses
- **Plain English:** MAC addresses identify interfaces on the local segment.
- **Why it matters:** They reveal who receives the frame *on this LAN hop*.
- **Analogy:** Apartment numbers inside one building.

### EtherType
- **Plain English:** EtherType identifies what payload the Ethernet frame carries (IPv4, ARP, etc.).
- **Why it matters:** It is the first clue for protocol demultiplexing at Layer 2.
- **Analogy:** A package sticker saying “contains documents” vs “contains electronics.”

### ARP
- **Plain English:** ARP maps IPv4 addresses to MAC addresses on local networks.
- **Why it matters:** Without ARP resolution, IP packets cannot be delivered on Ethernet.
- **Analogy:** Asking, “Who has this IP? Tell me your MAC.”

### Broadcast vs unicast
- **Plain English:** Broadcast is sent to everyone on the segment; unicast is sent to one specific MAC.
- **Why it matters:** ARP requests are broadcast; most data traffic is unicast.
- **Analogy:** Public announcement vs direct phone call.

### Checksums
- **Plain English:** Checksums detect corruption in headers/payloads (IP/TCP/UDP dependent).
- **Why it matters:** Unexpected checksum behavior can indicate corruption, offload effects, or header rewrites (for example NAT).
- **Analogy:** A quick math fingerprint to verify content integrity.

### Why packet analysis matters for troubleshooting and security
Packet analysis validates real behavior: service responses, delay, retransmission patterns, local gateway forwarding, and translation side effects. It is often the fastest way to separate app issues from network issues.

---

# Lab 1 – HTTP Flow and Addressing Baseline

## Why This Matters
A strong baseline on HTTP flow, timing, and addressing is useful before deeper transport/security analysis.

## Scenario
The investigation focused on HTTP traffic to a web target and measured response behavior and endpoint addressing.

## Method / Filters Used
- `http`
- Time reference around HTTP request/response packets
- Packet detail inspection for source and destination IP fields

## Key Findings
- HTTP request/response flow is visible and traceable at packet level.
- Measured request-to-response timing captured as **160.394953882 seconds**.
- Endpoint addressing in the captured flow shows:
  - Source IP: **10.0.2.15**
  - Destination IP: **185.125.190.18**

## Why These Findings Matter
Timing and endpoint context create a practical baseline: if later captures show worse response time or changed destination patterns, this baseline helps quickly identify regressions, routing changes, or upstream service issues.

## Evidence
![Lab 1 HTTP filtered traffic](screenshots/lab1_q3_http_filter.png)
![Lab 1 HTTP timing](screenshots/lab1_q4_http_timing.png)
![Lab 1 source/destination IP](screenshots/lab1_q5_source_destination_ip.png)

---

# Lab 2 – HTTP Semantics, Caching, and Authentication Behavior

## Why This Matters
HTTP packet semantics directly support web troubleshooting and security analysis: version mismatch, caching confusion, and auth challenge loops all appear in headers/status lines.

## Scenario
The investigation examined normal HTTP retrieval, cache-aware responses, and Basic Authentication exchange behavior.

## Method / Filters Used
- `http`
- Frame-level inspection of request and status lines
- Header review (`Accept-Language`, `Last-Modified`, `Authorization`, `WWW-Authenticate`)

## Key Findings
- Client and server both operate with **HTTP/1.1** in the captured flow.
- One server response is **304 Not Modified**, confirming cache validation behavior instead of full object retransmission.
- Language preference is visible in headers: `Accept-Language: en-US,en;q=0.5`.
- Captured server metadata includes: `Last-Modified: Wed, 15 Oct 2025 23:34:32 GMT`.
- Authentication flow shows a classic challenge/response pattern:
  - First response: **401 Unauthorized** with `WWW-Authenticate: Basic ...`
  - Follow-up request: adds `Authorization: Basic ...`
- Client/server addressing in this dataset includes:
  - Client/source: **10.0.2.15**
  - Server/destination: **128.119.245.12**

## Why These Findings Matter
- A `304` can explain “page loaded instantly” vs “new content not visible” user reports.
- A `401` + later `Authorization` header validates auth workflow at packet level.
- Header context (`Last-Modified`, language preference) helps verify app behavior and client policy.

## Evidence
![Lab 2 request/response frames](screenshots/lab2_q1_frame_665.png)
![Lab 2 304 status](screenshots/lab2_q1_frame_759_status.png)
![Lab 2 accept-language header](screenshots/lab2_q2_accept_language.png)
![Lab 2 last-modified header](screenshots/lab2_q5_last_modified.png)
![Lab 2 basic auth header](screenshots/lab2_q8_basic_auth_header.png)

---

# Lab 3 – Transport Analysis: TCP Reliability Signals, UDP Context, and VPN Visibility

## Why This Matters
Transport-layer patterns are where performance and reliability problems become visible. Packet evidence here can separate congestion/retransmission issues from application defects.

## Scenario
The lab artifacts include ping context, TCP stream analysis, retransmission-focused evidence, and VPN-related capture points.

## Method / Filters Used
- `ip.addr == 8.8.8.8` (ping-focused context)
- `tcp.stream == <n>` (stream-level TCP review)
- Retransmission-focused TCP inspection (for example fast retransmit indicators)
- VPN traffic verification view

## Key Findings
- TCP stream-level and retransmission evidence is present in the dataset screenshots.
- VPN-related traffic confirmation evidence is included.
- The available materials in this repository do **not** preserve a complete text extraction of all numeric findings for this lab, so values are interpreted conservatively.

## Why These Findings Matter
Analyst note: **UDP and TCP trade-offs are operational, not theoretical**—TCP overhead buys reliability and ordering, while UDP keeps overhead low at the cost of built-in recovery.

Analyst note: **A capture with retransmission indicators is often the fastest clue** that packet loss, path instability, or congestion is affecting user experience.

## Evidence
![Lab 3 ping-focused capture](screenshots/lab3_q1_ping_capture.png)
![Lab 3 TCP stream evidence](screenshots/lab3_q2_tcp_stream.png)
![Lab 3 fast retransmission evidence](screenshots/lab3_q3_fast_retransmission.png)
![Lab 3 VPN evidence](screenshots/lab3_q4_vpn_confirmation.png)

---

# Lab 4 – NAT and Multi-Vantage Packet Interpretation

## Why This Matters
NAT is a frequent source of confusion when teams compare captures from different points in the network. Understanding translation prevents false conclusions.

## Scenario
The investigation compares traffic behavior around NAT and reviews TCP/UDP/ICMP views relevant to translation-aware analysis.

## Method / Filters Used
- NAT-side and counterpart capture comparison
- TCP and UDP field review (ports/header context)
- ICMP inspection where applicable

## Key Findings
- NAT-focused screenshots show traffic from different vantage points.
- TCP port and UDP header comparison evidence exists in the dataset.
- ICMP behavior is captured as part of translation-aware analysis context.
- Full per-packet numeric extraction for every NAT prompt is not clearly recoverable from remaining repo artifacts; findings are reported without inventing unseen values.

## Why These Findings Matter
Analyst note: **Two captures of the same flow can look “different” and both be correct** when NAT rewrites source information between observation points.

Analyst note: **Checksum interpretation must account for header rewrites and host offload behavior**, or analysts may mislabel normal translation/offload effects as corruption.

## Evidence
![Lab 4 NAT-side capture](screenshots/lab4_q1_nat_home_capture.png)
![Lab 4 TCP port comparison](screenshots/lab4_q2_tcp_ports.png)
![Lab 4 UDP header comparison](screenshots/lab4_q3_udp_header_compare.png)
![Lab 4 ICMP context](screenshots/lab4_q4_icmp_nat_view.png)

---

# Lab 5 – Ethernet and ARP: Local-LAN Truth

## Why This Matters
When communication fails on a local network, Ethernet and ARP behavior often tells the real story before higher-layer logs do.

## Scenario
The lab artifacts focus on Ethernet frame fields, MAC addressing interpretation, EtherType identification, and ARP request/response behavior.

## Method / Filters Used
- `eth`
- `arp`
- Frame detail review for source/destination MAC, EtherType, and ARP fields

## Key Findings
- Ethernet frame structure and MAC-level addressing are clearly represented in capture evidence.
- EtherType-based protocol identification is included in the dataset.
- ARP request/response flow evidence is present.
- Some ARP/Ethernet item-level text answers were not fully recoverable after source artifact cleanup, so conclusions are limited to validated screenshot evidence.

## Why These Findings Matter
Analyst note: **An Ethernet destination MAC on internet-bound traffic is usually the local gateway, not the remote web server**, because Ethernet addressing is local-segment scope.

Analyst note: **ARP broadcast-to-unicast transition is a key sanity check** for local reachability: broadcast asks who owns the IP, unicast follows once MAC mapping is known.

## Evidence
![Lab 5 Ethernet frame details](screenshots/lab5_q1_ethernet_frame.png)
![Lab 5 MAC addressing](screenshots/lab5_q2_mac_addresses.png)
![Lab 5 EtherType evidence](screenshots/lab5_q3_ether_type.png)
![Lab 5 ARP packet details](screenshots/lab5_q4_arp_details.png)
![Lab 5 ARP resolution behavior](screenshots/lab5_q5_arp_resolution.png)

---

## Common Wireshark Filters Used

- `http` — isolate HTTP request/response traffic and headers.
- `arp` — inspect address resolution and local-LAN mapping behavior.
- `eth` — examine frame-level fields (MAC addresses, EtherType).
- `ip.addr == <ip>` — isolate traffic to/from one host.
- `tcp.stream == <n>` — reconstruct one TCP conversation.
- `udp.stream == <n>` — isolate one UDP flow where stream indexing is available.
- `tcp.flags.syn == 1` — focus on connection setup attempts.
- `icmp` — review echo/reachability behavior and control-plane responses.

## Practical Takeaways

- Packet evidence can validate application behavior faster than logs alone.
- Multi-layer analysis (Ethernet → IP → TCP/UDP → HTTP) is essential for accurate root cause analysis.
- NAT-aware interpretation prevents incorrect conclusions when comparing captures from different network points.
- ARP and MAC-level checks are critical early steps in local connectivity troubleshooting.

## Real-World Relevance

This project maps directly to real network and security workflows:

- **Network troubleshooting:** identify where communication breaks (L2/L3/L4/L7).
- **SOC/analyst triage:** validate suspicious or failed communication at packet level.
- **Application validation:** confirm status codes, auth exchanges, and header behavior.
- **Performance analysis:** reason about delay and retransmission symptoms.
- **Operational confidence:** understand how traffic *actually moves*, not just how diagrams describe it.
