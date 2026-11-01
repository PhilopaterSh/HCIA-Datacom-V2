# 12. ACL Fundamentals and Configuration

> 💡 Unit 12 of 23 - HCIA-Datacom course. ☰ MOC | ← Unit 11 | Unit 13 →


> **Source slides (physical PDF page anchors verified against the local 985-page training material):**
> - Slide 3: ACL Overview and Slide 4: Basic Principles of ACL
> - Slide 6: ACL Classification
> - Slide 8: Wildcard Mask Matching Rules
> - Slide 11: Matching Order (config vs auto)
> - Slide 16: Basic ACL Configuration & Traffic Filter Application
> - Slide 20: Advanced ACL Configuration Commands

> **V2 source anchors:** ACL purpose and rule model, p. 567; ACL categories and number ranges, p. 574; matching and negative-match semantics, pp. 576-580; basic/advanced ACL configuration, pp. 584-588.

### 12.1 ACL Architecture & Classification

An **Access Control List (ACL)** is a sequential series of rules used to classify and filter network traffic. In Huawei VRP, ACLs serve two primary architectural functions:
1. **Traffic Filtering:** Dropping unauthorized packets directly at switch/router interfaces using `traffic-filter`.
2. **Route / Traffic Classification:** Identifying traffic for NAT, IPsec VPN, QoS queuing, and BGP/OSPF route filtering.

#### ACL Number Categories:
| Category | ACL Number Range | Match Criteria (Packet Headers) | Application Scope |
| :--- | :---: | :--- | :--- |
| **Basic ACL** | **2000-2999** | Source IPv4 address, fragment information, and effective time range. | Source-based matching. |
| **Advanced ACL** | **3000-3999** | Source/destination IPv4 addresses, IP protocol, TCP/UDP ports, fragment information, and effective time range. | Fine-grained IP packet matching. |
| **Layer 2 ACL** | **4000-4999** | Source/destination MAC, VLAN ID, and Layer 2 protocol type. | Ethernet-frame matching. |
| **User-defined ACL** | **5000-5999** | Packet-header offsets, masks, and user-defined byte strings. | Custom byte-pattern matching. |
| **User ACL (UCL)** | **6000-9999** | Source/destination addresses and user groups, protocol, ICMP type, and TCP/UDP ports. | User/group-aware matching. |
| **ARP-based ACL** | **23000-23999** | ARP source/destination IPv4 and MAC addresses. | ARP packet matching. |

---

### 12.2 Rule Numbering & Step Size Mechanics

Every rule inside an ACL has an integer Rule ID (e.g. `rule 5 permit ...`, `rule 10 deny ...`).

```text
Default Step Size = 5
Rule IDs: 5 ──(+5)──► 10 ──(+5)──► 15 ──(+5)──► 20
```

1. **Why Step Size 5?** It allows network administrators to insert intermediate rules later without renumbering the entire ACL (e.g., inserting `rule 7` between `rule 5` and `rule 10`).
2. **Auto-Allocation Rule:** If a user configures a rule without specifying an ID:
   - The system calculates: $\mathbf{\text{Next Multiple of 5}} > \mathbf{\text{Highest Current Rule ID}}$.
   - *Exam Scenario:* If the highest rule in ACL 2000 is `rule 12`, running `[Huawei-acl-basic-2000] rule deny` will automatically allocate **Rule 15**!

---

### 12.3 Rule Matching Order: `config` vs. `auto`

In Huawei VRP, an ACL supports two matching orders configured via `acl [number] match-order { config | auto }`:

1. **`config` (Default):**
   - Packets are matched in **ascending order of Rule IDs** (Rule 5 $\rightarrow$ Rule 10 $\rightarrow$ Rule 15...).
   - **First Match Wins:** As soon as a packet matches a rule (`permit` or `deny`), the switch executes the action and **stops searching** subsequent rules!
2. **`auto` (Depth-First Search):**
   - The system automatically orders rules based on **precision / specificity**:
     - Host IP (`wildcard 0.0.0.0`) is matched before Subnet IP (`wildcard 0.0.0.255`).
     - Specific port numbers are matched before any port.

---

### 12.4 Wildcard Mask Invariant (`0` = Match, `1` = Ignore)

Unlike standard subnet masks where binary 1s must be contiguous, an ACL **Wildcard Mask** allows arbitrary bit inspection:
- **Bit = `0`:** The corresponding bit in the packet IP **MUST match** the rule IP exactly.
- **Bit = `1`:** The corresponding bit is a **"don't care" (ignored)** bit.

```text
Host Specific (Exact IP):   192.168.1.1 0.0.0.0       (All 32 bits must match exactly)
Entire /24 Subnet:          192.168.1.0 0.0.0.255     (First 24 bits match; last 8 bits ignored)
Any IP Address:             0.0.0.0 255.255.255.255   (All 32 bits ignored = "any")
```

#### Advanced Exam Pattern: Even vs. Odd IP Matching via Wildcard `0.0.0.254`
In binary arithmetic, the least significant bit (Bit 0) strictly dictates whether an integer is even or odd:
- **Even numbers (0, 2, 4, 6, 8...) always end in binary `0`.**
- **Odd numbers (1, 3, 5, 7, 9...) always end in binary `1`.**

When a Wildcard Mask ends in `254` (`11111110` in binary):
- The first 7 bits are `1` (ignored).
- **The last bit (Bit 0) is `0` (STRICT MATCH REQUIRED!).**

```text
Rule: 192.168.1.0 0.0.0.254
Base IP Last Octet:     0 = 0 0 0 0 0 0 0 [0]  <── Bit 0 is 0
Wildcard Last Octet:  254 = 1 1 1 1 1 1 1 [0]  <── Bit 0 is 0 (MUST MATCH BASE IP BIT!)

Checking IP 192.168.1.4 (EVEN):
  Octet 4 in binary:    4 = 0 0 0 0 0 1 0 [0]  <── Matches Bit 0! (PERMITTED)

Checking IP 192.168.1.1 (ODD):
  Octet 4 in binary:    1 = 0 0 0 0 0 0 0 [1]  <── Bit 0 is 1, does NOT match 0! (DENIED/DROPPED)
```

> 💡  The Universal Rule
> - Base IP ending in 0 `source 192.168.1.0 0.0.0.254` matches **ALL EVEN IPs** (`.2, .4, .6, .8...`)
> - Base IP ending in 1 `source 192.168.1.1 0.0.0.254` matches **ALL ODD IPs** (`.1, .3, .5, .7...`)

---

### 12.5 Huawei VRP Configuration Syntax

```bash
# ==========================================
# 1. Basic ACL (2000-2999) - Source IP Only
# ==========================================
[Huawei] acl number 2000
[Huawei-acl-basic-2000] rule 5 deny source 192.168.1.10 0          # Deny single host
[Huawei-acl-basic-2000] rule 10 permit source 192.168.1.0 0.0.0.255 # Permit rest of subnet
[Huawei-acl-basic-2000] quit

# ==========================================
# 2. Advanced ACL (3000-3999) - Fine-Grained
# ==========================================
[Huawei] acl number 3000
[Huawei-acl-adv-3000] rule 5 deny tcp source 192.168.1.0 0.0.0.255 destination 10.1.1.1 0 destination-port eq 23 # Block Telnet
[Huawei-acl-adv-3000] rule 10 permit tcp source any destination 10.1.1.1 0 destination-port eq 80              # Allow Web
[Huawei-acl-adv-3000] rule 15 deny ip source any destination 10.1.1.1 0                                          # Block all other IP
[Huawei-acl-adv-3000] quit

# ==========================================
# 3. Applying ACL to an Interface
# ==========================================
[Huawei] interface GigabitEthernet 0/0/1
[Huawei-GigabitEthernet0/0/1] traffic-filter inbound acl 3000       # Apply inbound (entering router)
[Huawei-GigabitEthernet0/0/1] traffic-filter outbound acl 2000      # Apply outbound (leaving router)
[Huawei-GigabitEthernet0/0/1] quit
```

#### Directional Application & Strategic Placement Rules

1. **Directional Flow Mechanics:**
   - **`traffic-filter inbound`:** Inspects and filters packets as they **enter** the interface, prior to route table lookup or forwarding.
   - **`traffic-filter outbound`:** Inspects and filters packets as they **exit** the interface, after the routing lookup and output port selection have occurred.

2. **The Golden Placement Rules (Huawei Exam Invariant):**
   - **Basic ACL (2000-2999) $\rightarrow$ Place as CLOSE TO DESTINATION as possible:**
     - *Why?* Basic ACLs only inspect the **Source IP**. If applied near the source host, the rule drops *all* outbound traffic from that host, unintentionally preventing it from communicating with any other server, gateway, or legitimate subnet (collateral damage).
   - **Advanced ACL (3000-3999) $\rightarrow$ Place as CLOSE TO SOURCE as possible:**
     - *Why?* Advanced ACLs inspect Source IP, Destination IP, Protocol type, and TCP/UDP port numbers. Because they can pinpoint an exact micro-flow (e.g., deny only Telnet to Server X while allowing Web browsing to Server Y), dropping the forbidden packet directly at the ingress edge avoids wasting WAN/campus bandwidth across transit links.

3. **Default Unmatched Packet Behavior:**
   - A packet that matches no ACL rule produces a **negative match**. The service module (for example, traffic filtering, MQC, or a protocol server) determines the resulting action; the ACL match result alone does not mean permit.
   - If a policy requires a catch-all result, configure it explicitly in the relevant ACL/service policy and verify that module?s documented handling.

---


| QBank Question | Topic & Concept | Answer |
| :--- | :--- | :---: |

---

