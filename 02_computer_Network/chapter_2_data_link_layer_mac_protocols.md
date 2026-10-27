# Chapter 2: Data Link Layer & MAC Protocols

---

## 1. IEEE 802 Sublayer Architecture

### 1.1 Overview
In the IEEE 802 standard, the Data Link Layer (OSI Layer 2) is partitioned into two specialized sublayers:
1. **Logical Link Control (LLC) Sublayer (Upper)**
2. **Media Access Control (MAC) Sublayer (Lower)**

```text
      OSI / Internet Model                           IEEE 802 Standard Model
   +-------------------------+                     +-------------------------+
   |   Layer 3: Network      |                     |   Layer 3: Network      |
   +-------------------------+                     +-------------------------+
   |                         |                     |   Logical Link Control  | (LLC - IEEE 802.2)
   |   Layer 2: Data Link    |  ===============>   |         (Upper)         |
   |                         |                     +-------------------------+
   |                         |                     |   Media Access Control  | (MAC - 802.3, 802.4,
   |                         |                     |         (Lower)         |  802.5, 802.11)
   +-------------------------+                     +-------------------------+
   |   Layer 1: Physical     |                     |   Layer 1: Physical     |
   +-------------------------+                     +-------------------------+
   |   Transmission Medium   |                     |   Transmission Medium   |
   +-------------------------+                     +-------------------------+
```

### 1.2 Responsibilities of Sublayers
* **Logical Link Control (LLC - IEEE 802.2):**
  * Hardware-independent sublayer that provides an interface to the network layer (IPv4, IPv6, IPX).
  * Manages flow control, error control, and the software portion of framing.
  * Uses Service Access Points (DSAP and SSAP) to multiplex multiple network-layer protocols.
* **Media Access Control (MAC):**
  * Hardware-dependent sublayer directly interfacing with the physical layer.
  * Embeds physical 48-bit hardware identifiers (MAC addresses).
  * Implements medium access mechanisms (CSMA/CD, CSMA/CA, Token Passing) to govern channel sharing.
  * Appends framing delimiters, preambles, and cyclic redundancy checks.

---

## 2. Fundamental Network Latencies & Delays

When data packets traverse transmission media and intermediate switches, total delay comprises four components:

```text
                  Total Network Delay Components
   +-------------------------------------------------------------+
   |  T_total = Transmission + Propagation + Queuing + Processing |
   +-------------------------------------------------------------+
```

### 2.1 Propagation Delay ($T_p$)
The duration required for a bit to physically travel from the transmitter to the receiver across the medium:
$$T_p = \frac{\text{Distance } (D)}{\text{Propagation Speed } (S)}$$
* $D$: Distance between sender and receiver in meters.
* $S$: Propagation velocity in the medium ($\approx 2 \times 10^8\text{ m/s}$ in copper/glass, $3 \times 10^8\text{ m/s}$ in vacuum).

### 2.2 Transmission Delay ($T_t$ or $T_{fr}$)
The duration required for a transmitter to push all bits of a packet onto the physical channel:
$$T_t = \frac{\text{Packet Length } (L)}{\text{Bandwidth / Transmission Rate } (B)}$$
* $L$: Frame size in bits.
* $B$: Bandwidth or link capacity in bits per second (bps).

### 2.3 Queuing and Processing Delays
* **Queuing Delay ($\text{Delay}_{\text{que}}$):** Time spent by a packet waiting in router input/output memory queues before transmission.
* **Processing Delay ($\text{Delay}_{\text{pro}}$):** Time taken by intermediate switches/routers to parse headers, verify checksums, and execute routing table lookups.

---

## 3. Multiple Access Protocols Overview

When multiple nodes share a broadcast or multidrop channel, access protocols prevent or manage contention:

```text
                         +--------------------------+
                         | Multiple-Access Protocols|
                         +------------+-------------+
                                      |
         +----------------------------+----------------------------+
         |                            |                            |
         v                            v                            v
+------------------+         +------------------+         +------------------+
|  Random-Access   |         | Controlled-Access|         |  Channelization  |
|    Protocols     |         |    Protocols     |         |    Protocols     |
+--------+---------+         +--------+---------+         +--------+---------+
         |                            |                            |
         |-- Pure ALOHA               |-- Reservation              |-- FDMA
         |-- Slotted ALOHA            |-- Polling                  |-- TDMA
         |-- CSMA                     +-- Token Passing            +-- CDMA
         |-- CSMA/CD
         +-- CSMA/CA
```

---

## 4. Random Access: ALOHA Protocols

Developed in 1970 by Norman Abramson at the University of Hawaii for packet-switched wireless links.

### 4.1 Pure ALOHA
* **Principle:** Stations transmit frames whenever data is ready without prior channel sensing.
* **Vulnerable Time:** Any frame overlapping during $[t - T_{fr}, t + T_{fr}]$ destroys transmission:
  $$\text{Vulnerable Time (Pure ALOHA)} = 2 \times T_{fr}$$
* **Throughput Equation:**
  $$S = G \times e^{-2G}$$
  * Maximum throughput occurs at $G = 0.5$:
    $$S_{\max} = \frac{1}{2e} \approx 0.184 \quad (\mathbf{18.4\%})$$

```text
Pure ALOHA Vulnerable Time Window:
                  t - Tfr                     t                     t + Tfr
Time Axis --------|---------------------------|----------------------|-------->
Frame A:          |====== Frame A ======|     |                      | (Overlap at start)
Frame B:          |                           |====== Frame B ======|  (Target Frame)
Frame C:          |                           |         |====== Frame C ======| (Overlap at end)
                  |<------------------ 2 * Tfr --------------------->|
```

### 4.2 Slotted ALOHA
* **Principle:** Continuous time is divided into discrete slots of length $T_{fr}$. Transmissions are synchronized to begin strictly at slot boundaries.
* **Vulnerable Time:** A frame can only collide with frames dispatched at the exact same slot boundary:
  $$\text{Vulnerable Time (Slotted ALOHA)} = 1 \times T_{fr}$$
* **Throughput Equation:**
  $$S = G \times e^{-G}$$
  * Maximum throughput occurs at $G = 1$:
    $$S_{\max} = \frac{1}{e} \approx 0.368 \quad (\mathbf{\approx 37\%})$$

### 4.3 Pure ALOHA vs. Slotted ALOHA

| Parameter | Pure ALOHA | Slotted ALOHA |
| :--- | :--- | :--- |
| **Time Structure** | Continuous; transmission at any instant | Discrete time slots of length $T_{fr}$ |
| **Vulnerable Time**| $2 \times T_{fr}$ | $1 \times T_{fr}$ |
| **Max Efficiency** | $18.4\%$ ($G = 0.5$) | $36.8\% \approx 37\%$ ($G = 1$) |
| **Clock Sync** | None required | Requires global clock synchronization |
| **Collision Scope**| Partial or complete frame overlaps | Entire slot overlaps only |

---

## 5. Carrier Sense Multiple Access (CSMA)

CSMA enforces **"Listen Before Talk"** (sense the carrier before transmitting) to minimize collisions.

### 5.1 Why Collisions Occur in CSMA
Even with carrier sensing, collisions occur due to **propagation delay ($T_p$)**. If Station A transmits at time $t_1$, Station B may sense the channel before Station A's signal arrives, falsely detect an idle line, and transmit, creating a collision midway.

```text
                  Station A                                   Station B
                  (Sends at t1)                               (Sends at t2)
                      |                                           |
Time t1:              *=================>                         | (A's signal propagating)
Time t2:              |===============>          <================* (B senses idle & transmits)
Time Collision:       |==================== X ====================| (Collision Occurs)
```
* **Vulnerable Time of CSMA:** Equal to the maximum propagation delay:
  $$\text{Vulnerable Time (CSMA)} = T_p$$

### 5.2 Persistence Methods

```text
1-Persistent:        [Busy Channel] ---> Continuously Sense ---> [Idle] ---> Transmit (p = 1)
Non-Persistent:      [Busy Channel] ---> Wait Random Time   ---> Re-sense Line
p-Persistent:        [Busy Channel] ---> Sense till Idle    ---> Slot Boundary:
                                                                 - Transmit with probability p
                                                                 - Wait 1 slot with q = 1 - p
```

| Strategy | Action when Busy | Action when Idle | Collision Probability | Channel Idle Waste |
| :--- | :--- | :--- | :--- | :--- |
| **1-Persistent** | Senses continuously | Transmits immediately ($p=1$) | Highest | Lowest |
| **Non-Persistent** | Waits random time; re-senses | Transmits immediately | Lowest | High (idle during wait) |
| **p-Persistent** | Senses continuously | Transmits with prob $p$; defers with $1-p$ | Balanced | Low |

---

## 6. CSMA with Collision Detection (CSMA/CD)

Used in wired local area networks (IEEE 802.3 Ethernet).

### 6.1 Principle & Operation
Stations monitor the channel while transmitting. If the voltage/energy level surges to an abnormal state ($2 \times \text{Normal}$), a collision is detected. The station immediately aborts transmission, sends a high-frequency **Jamming Signal** (32 to 48 bits) to alert all nodes, and initiates Binary Exponential Backoff.

```text
Energy Level
    ^                      +-------------+
    |                      |  COLLISION  |
    |                      |  (Abnormal) |  Energy = 2 x Normal (~10V)
    |  +----------------+  +-------------+  +----------------+
    |  |  TRANSMISSION  |                   |  TRANSMISSION  |
    |  |    (Normal)    |                   |    (Normal)    |  Energy = Normal (~5V)
    |  +----------------+                   +----------------+
----+----------+------------------+-----------------+----------+--------> Time
     Idle (0V)                      Idle (0V)
```

### 6.2 Minimum Frame Size Derivation
For an endpoint to detect a collision, it must continue transmitting until the collision signal returns from the farthest station:
$$T_t \ge 2 \times T_p$$
$$\frac{L}{B} \ge 2 \times \frac{D}{S} \implies \mathbf{L_{\min} = 2 \times T_p \times B}$$
* For $10\text{ Mbps}$ Ethernet over a $2.5\text{ km}$ collision domain ($2T_p = 51.2\text{ }\mu\text{s}$):
  $$L_{\min} = 51.2\text{ }\mu\text{s} \times 10\text{ Mbps} = 512\text{ bits} = \mathbf{64\text{ Bytes}}$$

---

## 7. CSMA with Collision Avoidance (CSMA/CA)

Used in wireless local area networks (IEEE 802.11 Wi-Fi).

### 7.1 Why CSMA/CD Fails in Wireless
1. **High Signal Attenuation:** Wireless signals attenuate with the square of the distance ($1/d^2$).
2. **Small Collision Footprint:** Collisions add only $5\%\text{ to }10\%$ additional energy, indistinguishable from channel noise.
3. **Half-Duplex Hardware:** Transceiver power masks weak incoming collision energy on the same antenna.

### 7.2 The Three Avoidance Strategies
1. **Interframe Space (IFS):** The station defers transmission for a mandatory period (IFS) after the channel is sensed idle to permit distant transmissions to register. Shorter IFS confers higher priority.
2. **Contention Window:** The station counts down a random back-off interval of $R$ slots. If the medium becomes busy during countdown, **the timer freezes** and resumes once the medium is idle again.
3. **Positive Acknowledgments (ACK):** The receiver returns an explicit ACK frame upon error-free reception. Unacknowledged frames trigger exponential back-off and retransmission.

---

## 8. Flow Control & Sliding Window ARQ Protocols

Flow control aligns a fast sender's transmission rate with a slow receiver's processing buffer:

```text
                       +-------------------------+
                       | Sliding Window Protocol |
                       +------------+------------+
                                    |
         +--------------------------+--------------------------+
         |                          |                          |
         v                          v                          v
+------------------+       +------------------+       +------------------+
|  Stop-and-Wait   |       |    Go-Back-N     |       | Selective Repeat |
|       ARQ        |       |       ARQ        |       |       ARQ        |
+------------------+       +------------------+       +------------------+
```

### 8.1 Stop-and-Wait ARQ
* **Operation:** Transmits 1 frame and waits for an acknowledgment before sending the next.
* **Sequence Numbers:** Alternates between 0 and 1 ($1\text{-bit}$ field).
* **Total Cycle Time:**
  $$T_{\text{total}} = T_t + 2T_p = T_t + \text{RTT}$$
* **Efficiency ($\eta$):**
  $$\eta = \frac{T_t}{T_t + 2T_p} = \frac{1}{1 + 2a} \quad \text{where } a = \frac{T_p}{T_t}$$
* **Throughput:** $\text{Throughput} = \eta \times B = \frac{B}{1 + 2a}$.

```text
Stop-and-Wait Timeline:
Sender                                     Receiver
  |================ Frame 0 ==============>| (Accepts 0, expects 1)
  |<--------------- ACK 1 -----------------|
  |                                        |
  |==== Frame 1 (Lost ❌) ================>| (Receiver waits)
  | (Timeout occurs!)                      |
  |================ Frame 1 (Resent) =====>| (Accepts 1, expects 0)
  |<--------------- ACK 0 -----------------|
```

### 8.2 Go-Back-N (GBN) ARQ
* **Send Window Size ($W_s$):** $W_s = 2^m - 1$ (for $m$-bit sequence numbers).
* **Receive Window Size ($W_r$):** Strictly $W_r = 1$.
* **Timers:** A single timer tracks the oldest outstanding frame ($S_f$).
* **Out-of-Order Frames:** Silently discarded; no buffering at the receiver.
* **Acknowledgments:** Cumulative ACKs ($R_n$ acknowledges all frames up to $R_n - 1$).
* **Efficiency:**
  $$\eta_{\text{GBN}} = \frac{W_s}{1 + 2a} \quad (\text{Max } \eta = 100\% \text{ when } W_s = 1 + 2a)$$
* **Sequence Bits Requirement:** $m = \lceil \log_2(1 + 2a) \rceil$.

### 8.3 Selective Repeat (SR) ARQ
* **Window Sizing:** $W_s = W_r = 2^{m-1}$.
* **Operation:** Retransmits **only** the frame that was damaged or lost.
* **Receiver Buffering:** Accepts out-of-order frames within the receive window and buffers them until missing frames arrive.
* **Negative Acknowledgment (NAK):** Triggers immediate retransmission of a missing frame prior to timer expiration.
* **Timers:** Each transmitted frame maintains an independent countdown timer.

### 8.4 Comprehensive Comparison of ARQ Schemes

| Metric | Stop-and-Wait ARQ | Go-Back-N ARQ | Selective Repeat ARQ |
| :--- | :--- | :--- | :--- |
| **Sender Window ($W_s$)** | $1$ | $2^m - 1$ | $2^{m-1}$ |
| **Receiver Window ($W_r$)**| $1$ | $1$ | $2^{m-1}$ |
| **Efficiency ($\eta$)** | $\frac{1}{1 + 2a}$ | $\frac{W_s}{1 + 2a}$ | $\frac{W_s}{1 + 2a}$ |
| **Retransmissions** | Single frame | Lost frame + all subsequent ($N$ frames) | Only the damaged/lost frame |
| **Receiver Buffer** | 1 frame | 1 frame (discards out-of-order) | Full buffer window ($2^{m-1}$) |
| **Timers** | 1 | 1 (tracks $S_f$) | Multiple (1 per active frame) |
| **ACK Type** | Independent ($0$ or $1$) | Cumulative | Independent / Selective + NAK |

### 8.5 Piggybacking
In bidirectional communication, rather than sending independent ACK frames, the acknowledgment number is embedded directly into the header of an outgoing data frame moving in the reverse direction, preserving bandwidth.

---

## 9. Error Detection & Correction Techniques

### 9.1 Error Types
* **Single-Bit Error:** Exactly one bit in a data unit flips ($0 \rightarrow 1$ or $1 \rightarrow 0$).
* **Burst Error:** Two or more bits are altered. The burst length spans from the first corrupted bit to the last corrupted bit inclusive.

### 9.2 Hamming Distance
The number of differing bit positions between two binary words of equal size, evaluated via bitwise XOR:
$$d(x, y) = \text{Count of 1s in } (x \oplus y)$$
* **To detect $s$ errors:** $d_{\min} \ge s + 1$.
* **To correct $t$ errors:** $d_{\min} \ge 2t + 1$.

### 9.3 Parity Schemes
1. **Simple 1D Parity Check:** Adds 1 parity bit ($n = k + 1$). Guarantees $d_{\min} = 2$. Detects all odd numbers of bit errors ($1, 3, 5\dots$); fails for even numbers of errors.
2. **Two-Dimensional (2D) Parity Check:** Organizes bits into a matrix, computing parity for each row and each column. Detects up to 3 arbitrary bit errors and can correct single-bit errors.

### 9.4 Hamming Error-Correcting Code
* **Redundancy Bit Inequality:** For $k$ data bits and $r$ parity bits:
  $$2^r \ge k + r + 1 \quad \text{or} \quad k \le 2^r - r - 1$$
* **Bit Allocation:** Parity bits occupy power-of-2 positions ($1, 2, 4, 8, 16\dots$).
  * $P_1$ checks positions with LSB bit set: $1, 3, 5, 7, 9, 11\dots$
  * $P_2$ checks positions with second bit set: $2, 3, 6, 7, 10, 11\dots$
  * $P_4$ checks positions with third bit set: $4, 5, 6, 7, 12, 13, 14, 15\dots$
* **Syndrome Evaluation:** The binary combination of parity checks generates a syndrome pointing directly to the corrupted bit index ($000 = \text{No error}$).

### 9.5 Checksum Method (One's Complement)
* **Sender:** Divides data into equal $n$-bit words. Adds them using one's complement addition (wrap-around carry). Inverts the wrapped sum to yield the Checksum.
* **Receiver:** Adds all data words plus the Checksum with wrap-around addition and complements the result. A final outcome of all zeros ($0000$) verifies error-free delivery.

### 9.6 Cyclic Redundancy Check (CRC)
* Dataword of $k$ bits is augmented with $r$ zeros, where $r$ is the degree of the divisor polynomial $G(x)$.
* Binary modulo-2 division (bitwise XOR without carry) is performed.
* The resulting remainder ($r$ bits) is appended to the dataword, forming an $n$-bit codeword ($n = k + r$).
* At the receiver, dividing the codeword by $G(x)$ yields a syndrome. A zero syndrome accepts the frame; a non-zero syndrome drops it.

```text
CRC Modulo-2 Division (Data: 1001, Divisor: 1011 -> Augment with 000):
               1  0  1  0  (Quotient)
        ---------------------
1 0 1 1 ) 1  0  0  1  0  0  0
          1  0  1  1
          ----------
          0  0  1  0  0
             0  0  0  0
             -------
             0  1  0  0  0
                1  0  1  1
                ----------
                0  0  1  1  0
                   0  0  0  0
                   ----------
                      1  1  0  <--- REMAINDER (CRC)
Transmitted Codeword: 1 0 0 1 1 1 0
```

### 9.7 Polynomial Representation
Bit patterns are expressed as polynomials where bit position defines the exponent of $x$:
$$\text{Binary } 1000011 \iff x^6 + x + 1$$

---

## 10. IEEE 802.3 Ethernet Standard

### 10.1 Frame Structure

```text
|<------------------------- Physical Layer Header ------------------------->|<------------------------------ MAC Layer Frame ------------------------------>|
+--------------------------+-----------------------+------------------------+-------------------+----------------+---------------------+--------------------+
|         Preamble         |          SFD          |  Destination MAC Addr  | Source MAC Addr   |  Length / Type |   Data and Padding  |     CRC / FCS      |
|         7 Bytes          |        1 Byte         |        6 Bytes         |      6 Bytes      |    2 Bytes     |   46 to 1500 Bytes  |      4 Bytes       |
+--------------------------+-----------------------+------------------------+-------------------+----------------+---------------------+--------------------+
|<---------------- 8 Bytes ----------------------->|                                                             |<------------- Payload ------------>|
                                                    |<--------------------------------- 64 to 1518 Bytes ------------------------------------------------>|
```

* **Preamble (7 Bytes):** 56 bits of alternating `10101010` for bit synchronization.
* **Start Frame Delimiter / SFD (1 Byte):** Fixed pattern `10101011` indicating boundary of MAC addresses.
* **Destination Address (6 Bytes):** Physical address of recipient.
  * Unicast: $b_0 = 0$
  * Multicast: $b_0 = 1$
  * Broadcast: All 48 bits set to $1$ (`FF:FF:FF:FF:FF:FF`).
* **Source Address (6 Bytes):** Physical unicast address of transmitting NIC.
* **Length / Type (2 Bytes):** Values $\le 1500$ indicate payload length; values $\ge 1536$ indicate protocol type (e.g., IPv4, ARP).
* **Data and Padding (46 to 1500 Bytes):** Upper-layer payload. Padded with zero-bytes if smaller than 46 bytes.
* **CRC / FCS (4 Bytes):** 32-bit CRC calculation over MAC destination through payload.
* **Size Bounds:** Minimum MAC frame size is **64 bytes**; maximum is **1518 bytes**.

### 10.2 Manchester Encoding
Standard $10\text{ Mbps}$ Ethernet uses Manchester encoding to guarantee transitions at the midpoint of each bit interval:
* **IEEE 802.3 Convention:**
  * Bit $0$: Low-to-High transition ($\uparrow$)
  * Bit $1$: High-to-Low transition ($\downarrow$)
* **Baud Rate:** Requires $20\text{ Megabaud}$ signaling for a $10\text{ Mbps}$ bit stream.

```text
Manchester Encoding (IEEE 802.3 Standard):
Data:              0               1               0               1
             +-----+         +-----+         +-----+         +-----+
Clock:       |     |         |     |         |     |         |     |
       +-----+     +---+-----+     +---+-----+     +---+-----+     +---+
       
Signal:      +-----+         +-----+         +-----+         +-----+
(IEEE 802.3) |     |         |     |         |     |         |     |
       +-----+     +---+-----+     +---+-----+     +---+-----+     +---+
             ^               ^               ^               ^
          (L -> H)        (H -> L)        (L -> H)        (H -> L)
```

---

## 11. Legacy LAN Architectures: Token Bus, Token Ring, FDDI

### 11.1 Token Bus (IEEE 802.4)
* **Topology:** Physical linear Bus, operating logically as a Token-Passing Ring.
* **Data Rate:** $10\text{ Mbps}$ over coaxial cable.
* **Feature:** Deterministic latency suitable for automated factory systems (MAP).

### 11.2 Token Ring (IEEE 802.5)
* **Topology:** Physical Star (via Multi-Station Access Unit / MSAU), operating logically as a closed Ring.
* **Data Rate:** $4\text{ Mbps}$ and $16\text{ Mbps}$.
* **Access Control:** Token passing eliminates collisions entirely.

### 11.3 Fiber Distributed Data Interface (FDDI)
* **Topology:** Dual Counter-Rotating Ring (Primary Ring clockwise, Secondary Ring counter-clockwise).
* **Transmission Medium:** Optical Fiber (up to $200\text{ km}$).
* **Data Rate:** $100\text{ Mbps}$.
* **Fault Tolerance:** Self-healing loopback wraps signals from primary to secondary ring during a cable sever, preventing network partition.
* **Frame Size:** Supports data payloads up to **4500 bytes**.

```text
FDDI Self-Healing Ring Architecture:
Normal State:                    Cable Cut Event (Loopback Healing):
     +--- Primary (CW) --->+          +--- Primary Loopback --->+
     |                     |          |                         |
  [Node A]              [Node B]   [Node A]                  [Node B] X (Cut)
     |                     |          |                         |
     +<-- Secondary (CCW)-+          +<-- Folded Return Loop --+
```

---

## 12. Important BEU Examination Questions & Answers

### Q1. Differentiate between Go-Back-N ARQ and Selective Repeat ARQ. (7 / 14 Marks)
**Answer:**

| Parameter | Go-Back-N ARQ | Selective Repeat ARQ |
| :--- | :--- | :--- |
| **Sender Window Size ($W_s$)** | $2^m - 1$ | $2^{m-1}$ |
| **Receiver Window Size ($W_r$)**| $1$ | $2^{m-1}$ |
| **Retransmissions** | Retransmits lost frame plus all succeeding frames | Retransmits only the specific damaged/lost frame |
| **Receiver Buffering** | Discards out-of-order frames | Buffers out-of-order frames |
| **Timers** | 1 timer (tracks oldest frame $S_f$) | Individual timer for each active frame |
| **Feedback Mechanism** | Cumulative acknowledgments | Independent ACKs and explicit NAKs |
| **Bandwidth Efficiency** | Poor on noisy links with high loss rates | High efficiency on noisy links |

---

### Q2. In CSMA/CD, derive the condition for minimum frame size ($T_t \ge 2 T_p$). (7 Marks)
**Answer:**
1. Let maximum cable distance between two stations be $D$, with propagation delay $T_p$.
2. Station A starts transmitting at $t = 0$.
3. At time $t = T_p - \delta$ (just before A's signal reaches the other end), Station B senses the medium idle and begins transmitting.
4. An immediate collision occurs at Station B's end.
5. The collision noise wave requires another $T_p$ seconds to travel back across distance $D$ to Station A.
6. Station A detects the collision at time $t = T_p + T_p = 2T_p$.
7. For Station A to recognize the collision, it must still be actively transmitting when the collision wave arrives:
   $$T_t \ge 2 \times T_p$$
8. Substituting $T_t = \frac{L}{B}$ and $T_p = \frac{D}{S}$:
   $$\mathbf{L_{\min} = 2 \times T_p \times B = 2 \times \frac{D}{S} \times B}$$

---

### Q3. Explain the Checksum calculation method using One's Complement arithmetic with a numerical example. (7 Marks)
**Answer:**
* **Procedure:** Divide data into $n$-bit words. Sum them using one's complement addition, adding any end-around carry back into the LSB. Invert the wrapped sum to produce the Checksum.
* **Example:** Data to send in 4-bit words: $7, 11, 12, 0, 6$.
  * Decimal sum $= 7 + 11 + 12 + 0 + 6 = 36_{10} = 100100_2$.
  * End-around carry addition: Add high-order bits `10` to low-order bits `0100`:
    $$0100_2 + 10_2 = 0110_2 \quad (6_{10})$$
  * Checksum $= \text{Bitwise Inversion of } 0110_2 = \mathbf{1001_2} \quad (9_{10})$.
  * Transmitted stream: $(7, 11, 12, 0, 6, 9)$.
* **Receiver Verification:**
  * Sum all received words: $7 + 11 + 12 + 0 + 6 + 9 = 45_{10} = 101101_2$.
  * Add carry `10` to `1101`: $1101_2 + 10_2 = 1111_2$.
  * Inverting $1111_2$ yields $\mathbf{0000_2} \implies$ **Zero syndrome confirms error-free transmission**.

---

### Q4. Describe the IEEE 802.3 Ethernet frame format and explain why the payload has minimum (46 B) and maximum (1500 B) restrictions. (14 Marks)
**Answer:**
* **Frame Fields:** Preamble (7 B), SFD (1 B), Destination MAC (6 B), Source MAC (6 B), Length/Type (2 B), Payload (46–1500 B), CRC (4 B).
* **Minimum Payload (46 Bytes):**
  * CSMA/CD requires a minimum frame length of 64 bytes to guarantee collision detection ($T_t \ge 2T_p$).
  * Total fixed overhead $= 6 + 6 + 2 + 4 = 18\text{ bytes}$.
  * Minimum payload $= 64 - 18 = \mathbf{46\text{ bytes}}$. Data below 46 bytes is padded with zeros.
* **Maximum Payload (1500 Bytes):**
  * *Buffer Memory Cost:* Early 1980s RAM was expensive; smaller frames prevented buffer overflow on transceivers.
  * *Medium Hogging:* Prevents a single host from monopolizing the shared bus for prolonged durations, ensuring equitable medium access.

---

## 13. Multiple-Choice Questions (MCQ Bank)

1. **In Go-Back-N ARQ with a 4-bit sequence number field, the maximum sender window size is:**
   * (a) 16
   * (b) 15
   * (c) 8
   * (d) 7
   * *Answer:* **(b) 15** [Formula: $2^4 - 1 = 15$]

2. **The maximum theoretical throughput of Pure ALOHA is approximately:**
   * (a) $36.8\%$
   * (b) $50\%$
   * (c) $18.4\%$
   * (d) $100\%$
   * *Answer:* **(c) $18.4\%$**

3. **In CSMA/CD, a transmitting station detects collisions by monitoring:**
   * (a) Acknowledgment timeouts
   * (b) Abnormal voltage/energy surges on the channel
   * (c) Frequency modulation errors
   * (d) Bit stuffing flags
   * *Answer:* **(b) Abnormal voltage/energy surges on the channel**

4. **Which technique is used by CSMA/CA during the contention window if the channel becomes busy?**
   * (a) Timer freezes until the channel is idle again
   * (b) Timer resets to maximum
   * (c) Frame is dropped immediately
   * (d) Jamming signal is sent
   * *Answer:* **(a) Timer freezes until the channel is idle again**

5. **In Stop-and-Wait ARQ, if parameter $a = \frac{T_p}{T_t} = 0.5$, the efficiency ($\eta$) is:**
   * (a) $100\%$
   * (b) $50\%$
   * (c) $33.3\%$
   * (d) $25\%$
   * *Answer:* **(b) $50\%$** [Calculation: $\eta = \frac{1}{1 + 2(0.5)} = \frac{1}{2} = 50\%$]

6. **In standard IEEE 802.3 Ethernet, a binary '1' using Manchester encoding is represented by:**
   * (a) Low-to-High transition
   * (b) High-to-Low transition
   * (c) Constant high voltage
   * (d) Constant zero voltage
   * *Answer:* **(b) High-to-Low transition**

7. **How many parity bits ($r$) are required in a Hamming code for a 7-bit dataword ($k = 7$)?**
   * (a) 3
   * (b) 4
   * (c) 5
   * (d) 8
   * *Answer:* **(b) 4** [Inequality: $2^4 = 16 \ge 7 + 4 + 1 = 12$]

8. **What is the maximum data payload size in a standard Ethernet frame?**
   * (a) $512\text{ bytes}$
   * (b) $1500\text{ bytes}$
   * (c) $1518\text{ bytes}$
   * (d) $4500\text{ bytes}$
   * *Answer:* **(b) $1500\text{ bytes}$**

9. **FDDI achieves fault tolerance through:**
   * (a) CSMA/CD collision abortion
   * (b) Dual counter-rotating self-healing rings
   * (c) Spanning tree algorithm
   * (d) Stop-and-wait retransmissions
   * *Answer:* **(b) Dual counter-rotating self-healing rings**

10. **The Hamming distance between codewords $X = 01011$ and $Y = 11100$ is:**
    * (a) 1
    * (b) 2
    * (c) 3
    * (d) 4
    * *Answer:* **(c) 3** [Evaluation: $01011 \oplus 11100 = 10111$, count of 1s $= 3$]