# Chapter 1: Introduction to Computer Networks & Physical Architecture

---

## 1. Fundamentals of Computer Networks

### 1.1 Definition
A **Computer Network** is a telecommunications network that allows autonomous digital devices (nodes) to exchange data with each other using wired or wireless transmission media to share hardware and software resources[cite: 11].

* **Autonomous Nodes:** Each connected computer is independent and capable of local processing without being forcibly controlled by another node[cite: 11].
* **Resource Sharing:** Hardware peripherals (printers, hard drives) and software resources (applications, databases) are shared across systems to minimize redundancy[cite: 11, 12].

### 1.2 Network Architecture Overview
```text
                      +-------------------+
                      |   The Internet    |
                      +---------+---------+
                                |
                                v
                      +-------------------+
                      |       Modem       |
                      +---------+---------+
                                |
            +-------------------+-------------------+
            |                                       |
            v                                       v
     +--------------+                       +---------------+
     | Wired Switch |                       |Wireless Router|
     +-------+------+                       +-------+-------+
             |                                      |
   +---------+---------+                 +----------+----------+
   |         |         |                 |          |          |
   v         v         v                 v          v          v
[Client]  [Server]  [Printer]        [Laptop]    [Tablet]   [Mobile]
 (Wired)   (Wired)   (Shared)       (Wireless)  (Wireless) (Wireless)
```

### 1.3 Key Goals of Computer Networks
1. **Facilitating Communication:** Enables swift and efficient transfer of data via emails, video conferencing, and instant messaging[cite: 12].
2. **Resource Sharing:** Peripheral hardware (such as office printers) and expensive licensed software can be shared across an entire enterprise[cite: 11, 12].
3. **Centralized Data Storage & Access:** Central servers allow ubiquitous access to files while simplifying scheduled data backups and disaster recovery[cite: 12].
4. **Cost Efficiency:** Reduces capital expenditure (CAPEX) by eliminating unnecessary hardware and software duplication[cite: 12].
5. **Reliability & Redundancy:** Provides alternate routing paths and backup systems to maintain continuous uptime during node or link failures[cite: 12].

### 1.4 Applications of Computer Networks & Internet
* **Business & E-Commerce:** Electronic payments (UPI, Net Banking), supply chains, and remote work collaboration[cite: 1].
* **Education:** E-learning platforms, virtual classrooms, digital examinations, and global knowledge bases[cite: 1].
* **Healthcare (Telemedicine):** Remote patient monitoring and instant transmission of Electronic Health Records (EHR)[cite: 1].
* **Government Services (E-Governance):** Citizen portals, digital identity authentication, and secure public database access[cite: 1].
* **Entertainment:** On-demand audio/video streaming and low-latency online multiplayer gaming[cite: 1].
* **Travel & Navigation:** Online reservation engines (IRCTC, airlines) and real-time GPS fleet tracking[cite: 1].

---

## 2. Data Communication

### 2.1 Definition
**Data Communication** is the exchange of data between two devices via a transmission medium governed by an agreed-upon set of rules called protocols[cite: 2, 3].

### 2.2 The Five Essential Components
1. **Message:** The information or data payload to be transmitted (text, numbers, images, audio, video)[cite: 3].
2. **Sender (Transmitter):** The device creating and initiating the transmission[cite: 3].
3. **Receiver:** The destination device intended to receive the message[cite: 3].
4. **Transmission Medium:** The physical conduit along which the signal travels (guided or unguided)[cite: 3].
5. **Protocol:** A formalized set of rules governing communication, consisting of:
   * **Syntax:** Structure and format of data bits[cite: 3].
   * **Semantics:** Meaning and operational interpretation of the bit patterns[cite: 3].
   * **Timing:** Transmission rate and synchronization control[cite: 3].

```text
                     +---------------------------------------+
                     |       Protocol (Rule 1, Rule 2)       |
                     +---------------------------------------+
                                         |
     +------------+               +--------------+               +------------+
     |   Sender   | ------------> |   Message    | ------------> |  Receiver  |
     +------------+               +--------------+               +------------+
           \                             |                             /
            \                            v                            /
             +-----------------> Transmission Medium <---------------+
```

### 2.3 Transmission Modes
Defines the direction of signal flow between two interconnected systems[cite: 4]:

```text
                  +-------------------+
                  | Transmission Mode |
                  +---------+---------+
         +------------------+------------------+
         |                  |                  |
         v                  v                  v
+-----------------+ +---------------+ +---------------+
|  Simplex Mode   | |  Half-Duplex  | |  Full-Duplex  |
+-----------------+ +---------------+ +---------------+
```

| Parameter | Simplex Mode | Half-Duplex Mode | Full-Duplex Mode |
| :--- | :--- | :--- | :--- |
| **Direction** | Strictly unidirectional (1 direction)[cite: 5] | Bidirectional, alternating turns[cite: 6] | Bidirectional simultaneously[cite: 7] |
| **Transmission Capability** | One device transmits; other receives[cite: 5] | Both can transmit, but not together[cite: 6] | Both transmit and receive concurrently[cite: 7] |
| **Channel Capacity** | Entire bandwidth reserved for 1 direction[cite: 5] | Entire bandwidth used by active sender[cite: 6] | Divided/shared between both directions[cite: 7] |
| **Real-World Example** | Radio broadcast (Akashvani), Keyboard[cite: 5] | Walkie-Talkies[cite: 6] | Mobile telephone conversations[cite: 7] |

### 2.4 Fundamental Network Criteria
1. **Delivery & Accuracy:** Packets must reach the correct recipient completely error-free[cite: 8].
2. **Performance:** Evaluated via **Transit Time** (propagation delay) and **Response Time** (time between request and reply)[cite: 8].
3. **Reliability:** Quantified by failure frequency and recovery duration (MTBF/MTTR)[cite: 8].
4. **Security:** Safeguards against unauthorized packet interception, data alteration, and physical damage[cite: 8].

---

## 3. Physical Connections & Topologies

### 3.1 Line Discipline
* **Point-to-Point Connection:** A dedicated link reserved strictly between two endpoints[cite: 9]. 100% channel capacity is allocated to those two systems[cite: 9].
* **Multipoint (Multidrop) Connection:** More than two devices share a single common transmission medium either spatially (concurrently) or temporally (time-shared)[cite: 10].

### 3.2 Network Topologies

#### A. Mesh Topology (Fully Connected)
Every node has a dedicated point-to-point link to every other node[cite: 14].
* **Total Duplex Links:** $\frac{n(n - 1)}{2}$[cite: 14]
* **I/O Ports per Node:** $(n - 1)$
* **Advantages:** High throughput (no shared traffic), robust against individual line cuts, high privacy/security[cite: 15].
* **Disadvantages:** Expensive, thick bundles of wiring, complex installation[cite: 15].

```text
                  [ Node 1 ]
                 /    |    \
                /     |     \
      [ Node 2 ]------+-------[ Node 3 ]
        |   \  \     / \     /   |
        |    \  \   /   \   /    |
        |     \  \ /     \ /     |
        |      \  X       X      |
        |     /  / \     / \     |
      [ Node 4 ]------+-------[ Node 5 ]
               \      |      /
                \     |     /
                  [ Node 6 ]
```

#### B. Star Topology
All nodes connect via dedicated point-to-point links to a central controller (Hub or Switch)[cite: 16].
* **Total Links:** $n$ links[cite: 16, 17]
* **Advantages:** Economical cabling compared to mesh, simple node additions, link failure isolation[cite: 17].
* **Disadvantages:** **Single Point of Failure**; failure of the central hub collapses the entire network[cite: 17].

```text
               [ Station 1 ]
                     |
      [ Station 4 ]  |  [ Station 2 ]
             \       |       /
              +------+------+
              | Central Hub |
              +------+------+
             /       |       \
      [ Station 5 ]  |  [ Station 3 ]
                     |
               [ Station 6 ]
```

#### C. Bus Topology
Multipoint layout where all devices tap into a single shared backbone cable terminating at both ends with resistive terminators[cite: 18].
* **Components:** Backbone cable, Drop lines, Taps[cite: 18].
* **Advantages:** Minimal cable length, low initial deployment cost[cite: 19].
* **Disadvantages:** Difficult fault isolation; a break in the backbone halts all communication[cite: 19].

```text
[Terminator]                                                           [Terminator]
   |                                                                        |
===+===========+======================+======================+==============+===
               |                      |                      |
             [Tap]                  [Tap]                  [Tap]
               |                      |                      |
          (Drop Line)            (Drop Line)            (Drop Line)
               |                      |                      |
         +-----------+          +-----------+          +-----------+
         | Station 1 |          | Station 2 |          | Station 3 |
         +-----------+          +-----------+          +-----------+
```

#### D. Ring Topology
Each station connects to exactly two neighboring stations, creating a continuous closed loop[cite: 20].
* **Repeaters:** Each station integrates a repeater to regenerate weakened bit streams before forwarding[cite: 20].
* **Data Flow:** Unidirectional circulating signal[cite: 20].
* **Advantages:** Straightforward installation, automated fault isolation[cite: 21].
* **Disadvantages:** In a single ring, a cable break or disabled node collapses the entire network[cite: 21].

```text
                     +--------------------+
                     |  [ Station 1 ]     |
                     |    [Repeater]      |
                     +---------+----------+
                           /        \
                          /          \
           +------------+              +------------+
           | [Station 6]|              | [Station 2]|
           | [Repeater] |              | [Repeater] |
           +------+-----+              +-----+------+
                  |      (Clockwise Flow)    |
           +------+-----+              +-----+------+
           | [Station 5]|              | [Station 3]|
           | [Repeater] |              | [Repeater] |
           +------------+              +------------+
                         \            /
                          \          /
                     +---------+----------+
                     |    [Repeater]      |
                     |  [ Station 4 ]     |
                     +--------------------+
```

---

## 4. Geographic Network Classification

| Category | Full Form | Geographic Span | Ownership | Primary Example |
| :--- | :--- | :--- | :--- | :--- |
| **LAN** | Local Area Network[cite: 22] | Building, Lab, Campus ($\le$ few km)[cite: 22] | Private[cite: 22] | College computer lab[cite: 22] |
| **MAN** | Metropolitan Area Network[cite: 22] | City-wide ($\approx 10\text{--}50\text{ km}$)[cite: 22] | Private/Public Consortium[cite: 22] | Connected network across Mumbai[cite: 22] |
| **WAN** | Wide Area Network[cite: 22] | State, Country, Worldwide[cite: 22] | Multiple Carriers / Distributed | Statewide network of UP, The Internet[cite: 1, 22] |

---

## 5. OSI Reference Model & Layer Duties

The **Open Systems Interconnection (OSI)** model was formulated by the **International Organization for Standardization (ISO)** to permit heterogeneous computing systems to communicate[cite: 23]. It is a conceptual framework (directive principles), whereas **TCP/IP** represents the operational protocol suite[cite: 24].

```text
       DEVICE A (Sender)                                              DEVICE B (Receiver)
 +---------------------------+                                  +---------------------------+
 | 7. Application Layer      | <--- Peer-to-Peer (Layer 7) ---> | 7. Application Layer      |
 +-------------+-------------+                                  +-------------^-------------+
        | (Interface)                                                  | (Interface)
 +-------------v-------------+                                  +-------------+-------------+
 | 6. Presentation Layer     | <--- Peer-to-Peer (Layer 6) ---> | 6. Presentation Layer     |
 +-------------+-------------+                                  +-------------^-------------+
        |                                                              |
 +-------------v-------------+                                  +-------------+-------------+
 | 5. Session Layer          | <--- Peer-to-Peer (Layer 5) ---> | 5. Session Layer          |
 +-------------+-------------+                                  +-------------^-------------+
        |                                                              |
 +-------------v-------------+                                  +-------------+-------------+
 | 4. Transport Layer        | <--- Peer-to-Peer (Layer 4) ---> | 4. Transport Layer        |
 +-------------+-------------+                                  +-------------^-------------+
        |                                                              |
 +-------------v-------------+       Intermediate Node          +-------------+-------------+
 | 3. Network Layer          |-----> [3. Network Layer  ] ----->| 3. Network Layer          |
 +-------------+-------------+       [       |          ]       +-------------^-------------+
        |                            [2. Data Link Layer]              |
 +-------------v-------------+       [       |          ]       +-------------+-------------+
 | 2. Data Link Layer        |-----> [1. Physical Layer ] ----->| 2. Data Link Layer        |
 +-------------+-------------+              |                   +-------------^-------------+
        |                                   |                          |
 +-------------v-------------+              |                   +-------------+-------------+
 | 1. Physical Layer         |==============+==================>| 1. Physical Layer         |
 +---------------------------+                                  +---------------------------+
               \_________________ Transmission Medium ________________/
```

### 5.1 Layer 1: Physical Layer Duties
1. **Representation of Bits:** Converts binary streams into physical signals via encoding (data mapping) and signaling (waveform generation)[cite: 28].
2. **Data Rate:** Establishes transmission speed (bits per second) and bit duration[cite: 30].
3. **Line Configuration:** Point-to-Point vs Multipoint links[cite: 31].
4. **Physical Topology:** Mesh, Star, Bus, Ring[cite: 32].
5. **Transmission Mode:** Simplex, Half-Duplex, Full-Duplex[cite: 33].

#### Physical Cable Standards
* **Twisted Pair:** 10Base-T (10 Mbps), 100Base-T (100 Mbps), 1000Base-T (1 Gbps), Token Ring (4–16 Mbps)[cite: 29].
* **Coaxial Cable:** 10Base2 (Thinnet, 10 Mbps), 10Base5 (Thicknet, 10 Mbps)[cite: 29].
* **Fiber Optic:** 10Base-F (10 Mbps), 100Base-FX (100 Mbps), FDDI (100 Mbps)[cite: 29].

### 5.2 Layer 2: Data Link Layer Duties
1. **Framing:** Encapsulates network layer packets into frames with a Header (H2) and Trailer (T2)[cite: 34].
2. **Physical Addressing (MAC Address):** Attaches a 48-bit (6-byte) physical hardware address[cite: 34, 35]:
   * **First 24 Bits (3 Bytes):** OUI (Organizationally Unique Identifier assigned by IEEE)[cite: 35].
   * **Last 24 Bits (3 Bytes):** NIC specific identifier[cite: 35].
   * **Byte 1, Bit $b_0$:** $0 = \text{Unicast}$, $1 = \text{Multicast}$[cite: 35].
   * **Byte 1, Bit $b_1$:** $0 = \text{Globally Administered}$, $1 = \text{Locally Administered}$[cite: 35].
3. **Access Control:** Implemented via Multiple-Access Protocols[cite: 36]:
   * *Random-Access:* ALOHA, CSMA, CSMA/CD, CSMA/CA[cite: 36].
   * *Controlled-Access:* Reservation, Polling, Token Passing[cite: 36].
   * *Channelization:* FDMA, TDMA, CDMA[cite: 36].
4. **Flow Control:** Uses sliding window buffers to prevent a high-speed transmitter from overfilling a slow receiver's buffer[cite: 37]. Frames lost in transit trigger timeouts and retransmissions[cite: 37].

### 5.3 Layer 6: Presentation Layer Duties
1. **Translation:** Translates native machine character sets (e.g., ASCII/EBCDIC) into standard network formats and back to allow interoperability[cite: 38].
2. **Encryption & Decryption:** Uses cryptographic keys to transform readable Plain Text into coded Cipher Text ($C = E_k(P)$) and decode it back ($P = D_k(C)$) to preserve data privacy[cite: 39].
3. **Compression:** Reduces bit length of transmission strings, crucial for high-bandwidth multimedia streams[cite: 40].

### 5.4 Layer 7: Application Layer Duties & Services
1. **Network Virtual Terminal (NVT):** Software terminal emulation permitting remote host logon (e.g., TELNET)[cite: 24, 41].
2. **File Transfer, Access, and Management (FTAM):** Remote file manipulation and transport (e.g., FTP)[cite: 24, 41].
3. **Mail Services:** E-mail composition, routing, and mailbox storage (e.g., SMTP)[cite: 24, 41].
4. **Directory Services:** Distributed lookup databases for network object locations and address resolution (e.g., DNS)[cite: 24, 41].

---

## 6. Transmission Media (Layer-0)

```text
                         +---------------------+
                         | Transmission Medium |
                         +----------+----------+
                                    |
          +-------------------------+-------------------------+
          |                                                   |
          v                                                   v
+-------------------+                               +-------------------+
|   Guided / Wired  |                               | Unguided / Wireless|
+---------+---------+                               +---------+---------+
          |                                                   |
   +------+------+------+                              +------+------+------+
   |             |      |                              |             |      |
   v             v      v                              v             v      v
+-------+     +------+ +------+                     +-------+     +-------+ +--------+
|Twisted|     |Coax- | |Fiber |                     |Ground |     |  Sky  | |Space/  |
| Pair  |     | ial  | |Optic |                     | Wave  |     | Wave  | |Line-of-|
+-------+     +------+ +------+                     +-------+     +-------+ | Sight  |
                                                                            +--------+
```

### 6.1 Guided Media
1. **Twisted Pair Cable:** Two copper conductors with individual plastic insulation twisted together to balance and cancel out electromagnetic noise (crosstalk)[cite: 43]. Terminated using **RJ-45** connectors[cite: 43].
2. **Coaxial Cable:** Central copper conductor surrounded by dielectric insulation, an EMI-blocking metallic foil/braided shield, and an outer jacket[cite: 44]. Terminated using **BNC** or threaded **F-Type** connectors[cite: 44].
3. **Fiber Optic Cable:** Uses **Total Internal Reflection (TIR)** to guide optical pulses along a high-density glass/plastic core ($n_1$) surrounded by lower-density cladding ($n_2 < n_1$)[cite: 45]. Immune to EMI, lightweight, supporting bandwidth up to **1600 Gbps**[cite: 45].

### 6.2 Unguided (Wireless) Propagation
1. **Ground Wave Propagation (Surface Wave):** Low-frequency waves ($\text{kHz to low MHz}$) travel along the earth's curved contour[cite: 46]. Waves diffract around obstacles due to long wavelengths ($\lambda$) but attenuate rapidly over short distances[cite: 46].
2. **Sky Wave Propagation:** High-frequency waves ($3\text{ MHz} - 32\text{ MHz}$) radiate into the **Ionosphere** and reflect back down to earth[cite: 47]. Supports long-distance multi-hop communications up to **5000 km**[cite: 47]. The unserved gap between ground wave reach and the first reflected wave is called the **Skip Area**[cite: 46, 47].

---

## 7. Integrated Services Digital Network (ISDN)

ISDN upgrades legacy copper PSTN subscriber lines to carry end-to-end digital voice and data concurrently[cite: 48].

```text
[PC LAN] ---> [Local Hub] ---> [ISDN Bridge] === (Digital PSTN) ===> [ISDN Network]
                                     |
                                [Telephone]
```

* **Basic Rate Interface (BRI):** Configured as **$2\text{B} + 1\text{D}$** ($2$ Bearer channels of $64\text{ kbps}$ + $1$ Delta channel of $16\text{ kbps}$)[cite: 48]. Delivers aggregated user throughput of **$128\text{ kbps}$** for residential/SOHO setups[cite: 48].
* **Primary Rate Interface (PRI):** Designed for enterprises and digital PABX exchanges[cite: 48].
  * *North America / Japan (T1 standard):* **$23\text{B} + 1\text{D}$** yielding **$1.544\text{ Mbps}$**[cite: 48].
  * *Europe / Global (E1 standard):* **$30\text{B} + 2\text{D}$** yielding **$2.048\text{ Mbps}$**[cite: 48].
* **Decline:** Succeeded by high-speed broadband lines (ADSL, Cable, FTTH) offering superior throughput at lower subscriber costs[cite: 29, 45, 48].

---

## 8. High-Frequency BEU University Examination Questions

### Part A: Short / Conceptual Questions (2 to 5 Marks)
1. **Define a Computer Network. How does an autonomous node differ from a terminal in a time-sharing system?**[cite: 11]
2. **State the three core parameters that define a network protocol.** *(Answer: Syntax, Semantics, Timing)*[cite: 3]
3. **Differentiate between Bit Encoding and Signaling.**[cite: 28]
4. **Explain the structural role of the $b_0$ and $b_1$ bits in a 48-bit MAC address.**[cite: 35]
5. **Why are copper conductors in network cables twisted in pairs?**[cite: 43]
6. **Define the Skip Zone/Skip Area in wireless radio propagation.**[cite: 46, 47]
7. **What are the data rates and channel configurations of ISDN BRI and PRI?**[cite: 48]

### Part B: Long Analytical Questions (7 to 14 Marks)
1. **Explain the five components of data communication with a block diagram. Provide an evaluation of how protocol mismatches prevent communication between connected systems.**[cite: 2, 3]
2. **Compare Simplex, Half-Duplex, and Full-Duplex transmission modes using directionality, channel capacity sharing, and real-world examples.**[cite: 4, 5, 6, 7]
3. **Provide a technical comparison of Mesh, Star, Bus, and Ring topologies. Derive the formula for total duplex links and node I/O ports in an $n$-node Mesh network.**[cite: 14, 16, 18, 20]
4. **Draw and explain the 7-layer ISO/OSI Reference Model architecture. Trace the vertical data flow and explain the role of intermediate routing nodes.**[cite: 23, 24, 25, 26]
5. **Discuss the core duties of the Data Link Layer, detailing framing, physical addressing, and multiple-access techniques.**[cite: 34, 35, 36]
6. **Explain the construction, physical operating principles, advantages, and limitations of Fiber Optic Cables versus Coaxial Cables.**[cite: 44, 45]

---

## 9. Objective / Multiple-Choice Questions (MCQ Bank)

1. **A fully connected Mesh network consisting of 8 devices requires how many physical duplex links?**[cite: 14]
   * (a) 56
   * (b) 28
   * (c) 16
   * (d) 8
   * *Answer:* **(b) 28** [Calculation: $\frac{n(n - 1)}{2} = \frac{8 \times 7}{2} = 28$][cite: 14]

2. **In a 48-bit MAC address, if the least significant bit ($b_0$) of the first byte is set to 1, the address is classified as:**[cite: 35]
   * (a) Unicast
   * (b) Broadcast only
   * (c) Multicast
   * (d) Locally administered
   * *Answer:* **(c) Multicast**[cite: 35]

3. **Which OSI layer is directly responsible for data translation, encryption, and compression?**[cite: 38, 39, 40]
   * (a) Application Layer
   * (b) Presentation Layer
   * (c) Session Layer
   * (d) Transport Layer
   * *Answer:* **(b) Presentation Layer**[cite: 38, 39, 40]

4. **100Base-FX uses which physical transmission medium?**[cite: 29]
   * (a) Thick Coaxial cable
   * (b) Shielded Twisted Pair
   * (c) Fiber Optic cable
   * (d) Wireless radio
   * *Answer:* **(c) Fiber Optic cable**[cite: 29]

5. **The channel arrangement of an ISDN Basic Rate Interface (BRI) is:**[cite: 48]
   * (a) $23\text{B} + 1\text{D}$
   * (b) $30\text{B} + 2\text{D}$
   * (c) $2\text{B} + 1\text{D}$
   * (d) $1\text{B} + 2\text{D}$
   * *Answer:* **(c) $2\text{B} + 1\text{D}$**[cite: 48]

6. **Which wireless propagation mode utilizes reflection from the ionospheric layer of the atmosphere?**[cite: 47]
   * (a) Ground wave propagation
   * (b) Sky wave propagation
   * (c) Line-of-sight propagation
   * (d) Surface wave propagation
   * *Answer:* **(b) Sky wave propagation**[cite: 47]

7. **A central hub/switch represents a Single Point of Failure in which topology?**[cite: 16, 17]
   * (a) Mesh Topology
   * (b) Bus Topology
   * (c) Star Topology
   * (d) Ring Topology
   * *Answer:* **(c) Star Topology**[cite: 17]