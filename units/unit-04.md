# 04. Ethernet Switching Basics

> 💡 Unit 04 of 23 - HCIA-Datacom course. ☰ MOC | ← Unit 03 | Unit 05 →


---

### 7.1 Historical Evolution & Collision vs. Broadcast Domains
> **Book Slide References (Parsed via MarkItDown):**
> - PDF page 271: Ethernet Switching Basics chapter cover
> - PDF page 275: Ethernet Protocols
> - PDF page 276: Collision Domain
> - PDF page 277: Broadcast Domain
> - Ethernet protocol, p. 157 Collision domain, p. 158

#### 1. Fundamental Domain Concepts

| Concept | Layer | Definition | Boundary Device | Key Limitation |
| :--- | :---: | :--- | :--- | :--- |
| **Collision Domain** | Layer 1 (Physical) | A network segment where devices share physical media and simultaneous transmissions collide. | **Layer 2 Switch / Bridge** (Isolates collisions per port) | Hubs create a single shared collision domain across all connected ports. |
| **Broadcast Domain** | Layer 2 (Data Link) | The topological scope within which a broadcast frame (`FF-FF-FF-FF-FF-FF`) is propagated to all nodes. | **Layer 3 Router / VLAN** (Stops Layer 2 broadcasts) | Switches forward broadcasts to all ports; massive domains cause broadcast storms. |

#### 2. CSMA/CD Operational Mechanics
Carrier Sense Multiple Access with Collision Detection (IEEE 802.3) governs shared media transmission:
1. **Carrier Sense (Listen before transmit):** The NIC checks if the channel voltage is idle.
2. **Multiple Access:** If idle, any station may transmit.
3. **Collision Detection (Listen while transmitting):** The NIC continues monitoring channel voltage during transmission. If voltage exceeds normal thresholds, a collision is detected.
4. **Jamming & Backoff:** The colliding station immediately stops transmission, transmits a 32-bit **jam signal** to alert all nodes, and executes the **Truncated Binary Exponential Backoff** algorithm before retrying.

---

### 7.2 Ethernet Frame Architecture & Bitwise Sizing
> **Book Slide References (Parsed via MarkItDown):**
> - PDF page 280: Ethernet Frame Format and frame length
> - PDF page 281: Ethernet frame minimum and maximum length rationale
> - Ethernet II and IEEE 802.3 frame formats, p. 162



#### 1. Ethernet II Frame Fields Specification

| Field | Length | Bit Range | Description & Architectural Purpose |
| :--- | :---: | :---: | :--- |
| **Destination MAC (DMAC)** | 6 Bytes | 48 bits | Target physical address receiving the frame. |
| **Source MAC (SMAC)** | 6 Bytes | 48 bits | Originating hardware interface transmitting the frame. |
| **Type (EtherType)** | 2 Bytes | 16 bits | Demultiplexing pointer identifying upper-layer protocol (`0x0800` = IPv4, `0x0806` = ARP, `0x86DD` = IPv6). |
| **Data (Payload)** | 46-1500 Bytes | Variable | Upper-layer PDU (IP packet). If payload < 46 bytes, padding bytes (`0x00`) are appended. |
| **Frame Check Sequence (FCS)** | 4 Bytes | 32 bits | Cyclic Redundancy Check (CRC-32) calculating polynomial checksum over MAC header and data. |

#### 2. Deep-Dive: The 64-Byte Minimum Frame Length Requirement
- **The Physical Constraint:** In a shared CSMA/CD bus, the maximum round-trip propagation time across the collision domain is called the **Slot Time** ($51.2\,\mu\text{s}$ at 10 Mbps Ethernet, corresponding to 512 bits = 64 bytes).
- **The Problem:** If a frame were smaller than 64 bytes, a transmitting host could finish sending the entire frame before the collision signal from a distant node travels back. The sender would assume the transmission was successful and discard its buffer, resulting in undetected frame loss ("late collision").
- **The Solution:** The minimum frame size is strictly enforced at **64 bytes** ($6 + 6 + 2 + 46 + 4 = 64$).

---

### 7.3 MAC Address Architecture & Classification
> **Book Slide References (Parsed via MarkItDown):**
> - PDF page 282: What Is a MAC Address?
> - PDF page 284: MAC Address Presentation
> - PDF page 285: MAC Address Composition and Classification
> - PDF page 286: Broadcast MAC address details
> - MAC addresses, p. 164

********

#### MAC Address Categories

| Address Type | Definition | Eighth Bit of Byte 0 (I/G Bit) | Example Address | Behavior on Layer 2 Switch |
| :--- | :--- | :---: | :--- | :--- |
| **Unicast MAC** | Identifies a unique physical interface on a single device. | `0` | `00-1E-10-DD-DD-02` | Forwarded directly to matching port or flooded if unknown. |
| **Multicast MAC** | Identifies a logical group of devices listening for specific traffic. | `1` | `01-80-C2-00-00-01` (BPDU) | Flooded across all ports in the broadcast domain (unless IGMP snooping configured). |
| **Broadcast MAC** | All-ones address representing every station on the Layer 2 domain. | `1` (All 48 bits = 1) | `FF-FF-FF-FF-FF-FF` | Flooded out of all ports except the receiving interface. |

---

### 7.4 Layer 2 Switch Forwarding Engine & State Machine
> **Book Slide References (Parsed via MarkItDown):**
> - PDF page 293: Working Principles of Switches
> - PDF page 294: MAC Address Table
> - PDF page 295: Three Frame Processing Behaviors of a Switch
> - PDF page 296: Flooding
> - PDF page 299: MAC Address Learning on a Switch (1)
> - Switch forwarding by MAC table, p. 173 Switch frame-processing behaviors, p. 175 Source-MAC learning, p. 179



#### 1. The Two-Step Decision Pipeline
When an Ethernet frame enters switch port $X$:
1. **Learning Phase (Inspection of Source MAC):**
   - Switch checks if `Source MAC` exists in the MAC Address Table.
   - If missing: adds an entry `[Source MAC -> Port X, Dynamic, Aging Timer = 300s]`.
   - If exists: resets the 300-second aging timer for that entry.
2. **Forwarding Decision (Inspection of Destination MAC):**
   - **Forwarding (Unicast Hit):** If `Destination MAC` is in the table and points to port $Y$ ($Y \neq X$), forward out of port $Y$.
   - **Flooding (BUM Traffic):** If `Destination MAC` is:
     - Unknown Unicast (not in table),
     - Broadcast (`FF-FF-FF-FF-FF-FF`), or
     - Multicast (`01-xx-xx-...`),
     the switch copies and transmits the frame out of all ports in the same VLAN except port $X$.
   - **Discarding (Filtering):** If `Destination MAC` matches port $X$ (the ingress interface), or violates port security/VLAN rules, the frame is silently dropped.

> 💡 Architectural Invariant: Layer 2 Transparency & Data-Plane Boundaries
> - A standard Layer 2 switch operates **strictly and exclusively at the Data Link layer (Layer 2)**.
> - **Zero Inspection of Layer 3:** A Layer 2 switch never reads, inspects, or learns **IP addresses** (Source IP or Destination IP). It has no awareness of IP packets inside the payload.
> - **Zero Modification of Layer 3:** The switch **never modifies or touches the Layer 3 header**. Fields like IP TTL, Checksum, and IP addresses pass through completely unchanged (Transparent Bridging).
> - Only Layer 3 devices (**Routers** or Layer 3 Switches performing routing across VLANs) inspect IP headers, decrement TTL, and recalculate Layer 3 checksums.

#### 2. Dynamic vs. Static MAC Address Entries
Huawei switches maintain two fundamental classes of MAC address entries in the CAM table:

| Dimension | Dynamic MAC Entries | Static MAC Entries |
| :--- | :--- | :--- |
| **Creation Method** | Learned automatically from the **Source MAC** of incoming frames. | Manually configured by an administrator (`mac-address static <mac> <interface> vlan <vlan-id>`). |
| **Aging Lifecycle** | **Subject to Aging:** Automatically deleted if no traffic from that MAC is seen within the **aging timer (default 300 seconds)**. | **Permanent (Never Ages):** Not subject to the 300s aging timer. Persists across reboots if saved. |
| **Use Case** | General access switch ports facing mobile PCs, laptops, and client endpoints. | Binding critical network infrastructure (firewalls, servers, core gateways) to prevent spoofing. |

#### 3. VRP CLI Verification Commands
```bash
<Huawei> display mac-address                  # View dynamic and static MAC address entries
<Huawei> display mac-address aging-time       # Verify aging timer (Default: 300 seconds)
<Huawei> display mac-address verbose          # Detailed breakdown including VLAN, PE-VID, and slot info
```

---


| QBank Question | Topic & Concept | Answer |
| :--- | :--- | :---: |

---

