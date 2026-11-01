# 19. WLAN Fundamentals and Configuration

> 💡 Unit 19 of 23 — HCIA-Datacom course. ☰ MOC | ← Unit 18 | Unit 20 →


 (AP Onboarding, STA Access & VRP Config)

ightarrow$ `2. Discover WAC (Option 43/DNS/Broadcast)` $
ightarrow$ `3. Establish CAPWAP Tunnel & DTLS Handshake` $
ightarrow$ `4. Download Software & Configuration (Join/Config)` $
ightarrow$ `5. Normal Operation (Run State)`.
> - **WAC Source Interface Configuration Trap:** On Huawei WACs, the CAPWAP source address must be explicitly declared before APs can join:
>   `[AC] capwap source interface Vlanif 100` (or `capwap source ip-address <ip>`). Omitting this command prevents the WAC from listening on UDP 5246/5247!
> - **VAP Profile Binding Flexibility (V2 Exam Trap):** A Virtual AP (VAP) profile can be bound to an **AP group** (for batch deployment across all APs in a zone) OR directly to an **individual AP** for site-specific customization.

> **Source slides (verified against official HCIA-Datacom V2.0 Training Material):**
> - Slide 1: WLAN Fundamentals and Configuration Cover
> - Slide 5: Five Core Phases of AP Onboarding Lifecycle
> - Slide 8: Phase 1: AP Address Allocation & DHCP Option 43 Discovery
> - Slide 13: Phase 2 & 3: CAPWAP Tunnel Setup, DTLS Handshake & AP Authentication
> - Slide 17: Phase 4 & 5: Version Check, Configuration Download & Run State
> - Slide 19: STA Access Process (Scanning, Association, Authentication & DHCP)
> - Slide 22: Huawei WAC Configuration Profile Hierarchy (VAP, SSID, Security)
> - Slide 37: Complete WAC + Fit AP CLI Configuration Example
> - Slide 47: Verification & Diagnostic Commands: display ap all, display station all

---

### 17.5 AP Onboarding and STA Access

**AP onboarding** is a prerequisite to WLAN service: the WAC cannot centrally manage a Fit AP until it is online. V2 gives the sequence as address allocation, CAPWAP tunnel establishment, AP access control, AP software upgrade, configuration-status check, CAPWAP tunnel maintenance, and WAC service-configuration delivery. These stages extend the existing AP lifecycle summary with explicit operational checkpoints. AP onboarding stages, p. 879

| AP onboarding stage | Purpose in the sequence |
| :--- | :--- |
| IP address allocation | Gives the AP network reachability toward the WAC. |
| CAPWAP tunnel establishment | Creates the AP-to-WAC control relationship. |
| AP access control | Determines whether the AP is permitted to join. |
| Software upgrade | Aligns AP software where required by the deployment. |
| Configuration-status check | Confirms the AP configuration state. |
| CAPWAP tunnel maintenance | Keeps the management connection available. |
| WAC service configuration delivery | Supplies WLAN service configuration after onboarding. |

#### 1. AP Address Allocation and WAC Discovery

V2 gives three AP address acquisition modes: static configuration, DHCP, and IPv6 SLAAC (SLAAC supplies an IPv6 address only). DHCP is the typical deployment choice in the course.

For dynamic WAC discovery, DHCP can advertise the WAC address (IPv4 Option 43 or DHCPv6 Option 52); DNS discovery uses a DHCP-supplied WAC domain name and DNS server; broadcast discovery is limited to the same broadcast domain. The course's DNS procedure uses the `huawei-wlan-controller.` prefix before the configured domain name. These are discovery mechanisms, not interchangeable address-allocation modes. AP address modes, p. 880 Dynamic WAC discovery methods, p. 883 DNS naming and WAC selection, p. 884

#### 2. CAPWAP Establishment and AP Access Control

After discovery, the AP and WAC establish CAPWAP tunnels. The control tunnel carries management/control traffic; the data tunnel carries STA service traffic only in tunnel-forwarding mode. The WAC then applies AP access control (including IP-version checks, blacklist checks, and the configured none/MAC/SN authentication mode), may upgrade AP software, and supplies initial configuration. V2 distinguishes control-tunnel Echo detection on UDP 5246 from data-tunnel Keepalive detection on UDP 5247. The stages and ports describe the V2 course sequence; verify software-specific behavior before using them as an operational runbook. CAPWAP tunnel roles, p. 886 AP access checks, p. 888 Configuration status, p. 891 CAPWAP tunnel maintenance, p. 892

**CAPWAP Link Maintenance & Disconnection Behavior (Exam Invariant):**
The AP and AC periodically exchange heartbeat packets to detect tunnel liveliness:
- **Control Tunnel Heartbeat:** Maintained via **Echo Request / Echo Response** packets over **UDP port 5246**.
- **Data Tunnel Heartbeat:** Maintained via **Keepalive** packets over **UDP port 5247**.
- **Tunnel Interruption Consequence:** If consecutive Echo heartbeats fail to receive an Echo Response within the timeout threshold, the CAPWAP Control Tunnel is torn down. Because a Fit AP relies on the AC for control-plane logic, client authentication, and policy delivery:
  1. The Fit AP enters a **degraded / offline state** (status transitions to `Fault` or `Offline` on the WAC).
  2. In **Tunnel Forwarding mode**, all wireless user data forwarding halts immediately because the CAPWAP data tunnel terminates with the control tunnel.
  3. The AP cannot admit new wireless stations (STAs) and immediately initiates WAC re-discovery (Discovery Request) to recover the CAPWAP session. Fit APs can never operate as autonomous standalone devices.

The V2 STA access process separates **scanning**, **link authentication**, and **association**. In the WAC + Fit AP architecture, association is handled through the AP/WAC architecture. Successful radio association is not by itself proof of end-to-end service: the STA still needs the correct service VLAN and address/DHCP path. V2 also lists device capacity limits, VAP user limits, user CAC, and STA blacklist/whitelist controls as factors affecting access. STA access phases and controls, p. 895 Scanning, p. 896 Link authentication, p. 898 Association, p. 900 STA association in WAC + Fit AP, p. 901

**STA data-forwarding relationship:** after association, user traffic follows the WLAN's configured forwarding mode. In direct mode, user traffic exits locally from the AP; in tunnel mode, it is carried to the WAC. Keep this user-data path distinct from the CAPWAP control path used for AP management. Direct forwarding, p. 903 Tunnel forwarding, p. 904

> 💡 Practical boundary
> This adds the V2 theory sequence. The multi-step Huawei configuration examples and live AP/STA verification remain for the deferred lab phase; this is not a tested deployment recipe.

---

### 17.6 Huawei VRP WLAN Configuration (AC + Fit AP)

```bash
# =====================================================================
# Step 1: DHCP Pool — Assign IP to AP and tell it the AC IP (Option 43)
# =====================================================================
[AC] dhcp enable
[AC] ip pool AP_POOL
[AC-ip-pool-AP_POOL] network 192.168.100.0 mask 255.255.255.0
[AC-ip-pool-AP_POOL] gateway-list 192.168.100.1
[AC-ip-pool-AP_POOL] option 43 ascii huawei-ac 192.168.100.254  # AC management IP
[AC-ip-pool-AP_POOL] quit

# =====================================================================
# Step 2: Create SSID Profile (defines the wireless network name)
# =====================================================================
[AC] wlan
[AC-wlan-view] ssid-profile name CORP_SSID
[AC-wlan-ssid-prof-CORP_SSID] ssid CorpWiFi
[AC-wlan-ssid-prof-CORP_SSID] quit

# =====================================================================
# Step 3: Create Security Profile (defines authentication/encryption)
# =====================================================================
[AC-wlan-view] security-profile name CORP_SEC
[AC-wlan-sec-prof-CORP_SEC] security wpa2 psk pass-phrase Admin@123 aes
[AC-wlan-sec-prof-CORP_SEC] quit

# =====================================================================
# Step 4: Create VAP Profile (binds SSID + Security + Forwarding mode)
# =====================================================================
[AC-wlan-view] vap-profile name CORP_VAP
[AC-wlan-vap-prof-CORP_VAP] forward-mode tunnel          # or: direct-forward
[AC-wlan-vap-prof-CORP_VAP] service-vlan vlan-id 10
[AC-wlan-vap-prof-CORP_VAP] ssid-profile CORP_SSID
[AC-wlan-vap-prof-CORP_VAP] security-profile CORP_SEC
[AC-wlan-vap-prof-CORP_VAP] quit

# =====================================================================
# Step 5: Create AP Group and bind VAP Profile to Radio
# =====================================================================
[AC-wlan-view] ap-group name FLOOR1_APS
[AC-wlan-ap-group-FLOOR1_APS] vap-profile CORP_VAP wlan 1 radio 0    # 2.4 GHz
[AC-wlan-ap-group-FLOOR1_APS] vap-profile CORP_VAP wlan 1 radio 1    # 5 GHz
[AC-wlan-ap-group-FLOOR1_APS] quit

# =====================================================================
# Step 6: Confirm AP has joined (verification)
# =====================================================================
<AC> display ap all                    # Show all APs: state should be "Run"
<AC> display ap id 0 verbose           # Detailed AP info, radio, channel
<AC> display vap all                   # Verify VAP is active and broadcasting
<AC> display station all               # Show associated client stations
```

---

---


| :--- | :--- | :---: |

---

### 19.4 Chapter Summary Quiz

1. *(True/False)* In Huawei VRP WLAN service configuration, a VAP profile can be bound only to an AP group, and cannot be bound to an individual AP.
   - A. True
   - B. False
   - *Rationale:* Official V2.0 Slide p. 929 confirms: A VAP profile can be bound to an AP group (`ap-group name <group-name>`) OR bound directly to a single AP (`ap-id <id>`).

2. *(Single Choice)* An engineer is configuring basic WLAN services on a Huawei WAC and requires that the wireless SSID broadcast name be `Employee`. Which CLI command is syntactically correct?
   - A. `[WAC-wlan-ssid-prof-employee] ssid employee`
   - B. `[WAC-wlan-ssid-prof-employee] ssid Employee`
   - C. `[WAC-wlan-ssid-prof-employee] ssid name employee`
   - D. `[WAC-wlan-ssid-prof-employee] ssid name Employee`
   - *Rationale:* Official V2.0 Slide p. 929 confirms: Inside the SSID profile view, the command syntax is `ssid <ssid-name>` (case-sensitive).

3. *(Multiple Choice)* In the centralized WAC + Fit AP architecture, which of the following statements are correct regarding the **AP onboarding process**? (Select all that apply)
   - A. APs can obtain IP addresses through static configuration or DHCP.
   - B. The WAC centrally manages and controls APs through CAPWAP tunnels.
   - C. APs can discover WACs in static mode, broadcast discovery, or dynamic mode via DHCP Option 43 / DNS.
   - D. The WAC and APs exchange Echo messages to detect connectivity of both control and data tunnels.
   - *Rationale:* Official V2.0 Slide p. 930 confirms: Options A, B, and C are true. Echo Request/Response messages are exchanged to detect connectivity of the **Control tunnel** (Keepalive detects data tunnel).

---

> 💡 🔗 Convergent unit: 18. WLAN Networking Architecture (Fat - Fit AP, Leader AP & CAPWAP Tunnel) is the canonical CAPWAP/architecture treatment; this unit covers the configuration side.

> 💡 🔗 Convergent unit: 14. Network Services and Applications (SSH - STelnet, SFTP, FTP) carries the canonical DHCP server treatment; this unit covers DHCP only as the WLAN AP/STA addressing step.

