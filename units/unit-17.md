# 17. WLAN Technology Basics (Radio Spectrum, 802.11 Standards & Wi-Fi 6/7)

> 💡 Unit 17 of 23 - HCIA-Datacom course. ☰ MOC | ← Unit 16 | Unit 18 →



---

### 17.1 Wireless Spectrum & IEEE 802.11 Standards

#### 1. Wireless Frequency Bands

WLAN operates on two primary unlicensed radio frequency bands:

| Band        | Frequency Range  | Channel Width          | Non-Overlapping Channels                  | Characteristics                                                                                        |
| :---------- | :--------------- | :--------------------- | :---------------------------------------- | :----------------------------------------------------------------------------------------------------- |
| **2.4 GHz** | 2.400-2.4835 GHz | 20 MHz (22 MHz actual) | **3 channels: 1, 6, 11**                  | Longer range, higher wall penetration; heavily congested (المزدحمه) (microwaves, Bluetooth, other APs) |
| **5 GHz**   | 5.150-5.850 GHz  | 20/40/80/160 MHz       | **25 non-overlapping (التداخل) channels** | Higher throughput, shorter range, less congestion; supports channel bonding for wide channels          |

> 💡 2.4 GHz Non-Overlapping Channel Design
> The 2.4 GHz ISM band in most regions spans 83.5 MHz (13 channels, each 5 MHz apart (بصرف النظر) but 22 MHz wide). Only **channels 1, 6, and 11** are spaced far enough apart (25 MHz center-to-center) to avoid spectral overlap. Neighboring APs in an office or campus MUST use different channels from this set (1, 6, 11, then 1, 6, 11 again) to eliminate co-channel interference.

#### 2. IEEE 802.11 Standards Comparison

| Standard | Wi-Fi Generation | Band | Max Theoretical Throughput | Key Technology |
| :--- | :--- | :--- | :--- | :--- |
| **802.11b** | Wi-Fi 1 | 2.4 GHz only | 11 Mbps | DSSS modulation |
| **802.11a** | Wi-Fi 2 | **5 GHz only** | 54 Mbps | OFDM modulation (5 GHz pioneer) |
| **802.11g** | Wi-Fi 3 | 2.4 GHz only | 54 Mbps | OFDM; backward compatible with 802.11b |
| **802.11n** | Wi-Fi 4 | **2.4 GHz + 5 GHz** | 600 Mbps | MIMO (up to 4×4), 40 MHz channels |
| **802.11ac** | Wi-Fi 5 | **5 GHz only** | 6.9 Gbps | MU-MIMO, 80/160 MHz channels, 256-QAM |
| **802.11ax** | Wi-Fi 6 | **2.4 GHz + 5 GHz** | 9.6 Gbps | OFDMA, BSS Coloring, TWT (Target Wake Time), higher density |
| **802.11be** | **Wi-Fi 7** | Multi-link operation across supported bands | See the V2 source comparison | **Multi-Link Operation (MLO)** and **Multi-RU (MRU)** are highlighted in the V2 material. |

> 💡 Exam Mnemonic, Frequency Band Grouping
> - **The 'A' Family (802.11a & 802.11ac):** Strictly **5 GHz ONLY**.
> - **The 'B/G' Family (802.11b & 802.11g):** Strictly **2.4 GHz ONLY**.
> - **Dual-Band Standards (802.11n & 802.11ax):** Operate in **both 2.4 GHz and 5 GHz**.

V2 introduces Wi-Fi 7 as a delta beyond the 802.11b/g/n/ac/ax comparison already documented above. **MLO** lets a device use multiple links; **MRU** allows a user to receive multiple resource units in a transmission interval, including combinations of different RU sizes. The course presents these as mechanisms for higher throughput, lower latency / better efficiency, and improved link utilization. Do not infer that every client/AP combination supports all links or RU combinations; actual operation depends on device capability and configuration. Wi-Fi 7 Multi-RU, p. 829 Wi-Fi 7 Multi-Link, p. 830 Wi-Fi 7 compared with Wi-Fi 6, p. 831

---




---

### 17.3 Chapter Summary Quiz

1. *(Single Choice)* Which of the following IEEE 802.11 standards officially introduces support for the **6 GHz frequency band**?
   - A. 802.11a
   - B. 802.11ac
   - C. 802.11ax
   - D. 802.11be
   - *Rationale:* Official V2.0 Slide p. 832 confirms: 802.11be (Wi-Fi 7) natively incorporates the 6 GHz band alongside 2.4 GHz and 5 GHz with up to 320 MHz channel width.

2. *(Single Choice)* In 2.4 GHz WLAN cellular cell deployment, which channels must neighboring APs use to **completely avoid co-channel interference**?
   - A. 1, 2, 3
   - B. 1, 6, 11
   - C. 3, 6, 9
   - D. 6, 9, 12
   - *Rationale:* Only channels 1, 6, and 11 in the 2.4 GHz ISM band are separated by at least 25 MHz (each 20/22 MHz wide), preventing channel overlap.

---

