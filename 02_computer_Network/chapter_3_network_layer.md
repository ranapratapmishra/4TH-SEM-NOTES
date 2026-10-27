# Chapter 3: Network Layer

---

## 1. Network Layer Fundamentals & Architecture

### 1.1 Overview & Responsibilities
The Network Layer (Layer 3 of the OSI model) provides end-to-end communication, delivering packets from the original source host to the final destination host across multiple interconnected networks (internetworking).

```text
       Host A (Source)                                  Host B (Destination)
    +-------------------+                             +-------------------+
    | Application Layer |                             | Application Layer |
    +-------------------+                             +-------------------+
    |  Transport Layer  |                             |  Transport Layer  |
    +-------------------+                             +-------------------+
    |   Network Layer   | ===[ Host-to-Host Path ]===>|   Network Layer   |
    +-------------------+                             +-------------------+
    |  Data Link Layer  |                             |  Data Link Layer  |
    +-------------------+                             +-------------------+
    |  Physical Layer   |                             |  Physical Layer   |
    +-------------------+                             +-------------------+
```

#### Core Duties of the Network Layer:
1. **Source-to-Destination Delivery (Host-to-Host):**
   * Delivers datagrams from the initiating host to the target receiver regardless of intervening intermediate topologies.
   * *Hinglish:* Data Link Layer sirf ek hop (node-to-node) tak packet le jata hai, jabki Network Layer pure internet se guzar kar original source se final destination tak packet deliver karta hai.
2. **Logical Addressing:**
   * Allocates globally unique 32-bit (IPv4) or 128-bit (IPv6) addresses that remain independent of physical MAC hardware.
3. **Routing & Forwarding:**
   * Selects the most optimal path across network links using dynamic or static routing algorithms and populates the router forwarding tables.
4. **Packetizing (Encapsulation/Decapsulation):**
   * Encapsulates transport layer protocol data units (TCP segments, UDP datagrams) inside network datagrams by adding a Layer 3 header at the sender and extracting it at the receiver.
5. **Fragmentation & Reassembly:**
   * Divides datagrams exceeding physical Maximum Transmission Units (MTU) into smaller fragments at intermediate nodes and reassembles them at the destination host.
6. **Error & Flow Reporting (via ICMP):**
   * Does not guarantee reliability directly, but uses ICMP to notify the sender regarding delivery issues (TTL expired, destination unreachable).

---

## 2. IPv4 Datagram Header Architecture

IPv4 is a connectionless, best-effort, unreliable packet-switched datagram protocol. Delivery is not guaranteed; reliability must be handled by Layer 4 (e.g., TCP).

```text
 0                   4                   8                   16                                  31
+-------------------+-------------------+-------------------+-----------------------------------+
|    VER (4 bits)   |   HLEN (4 bits)   | Service Type (8 b)|        Total Length (16 bits)     | Row 1
+-------------------+-------------------+-------------------+-----------------------------------+
|           Identification (16 bits)                        | Flags (3 b)|  Frag. Offset (13 b) | Row 2
+---------------------------------------+-------------------+------------+----------------------+
|    Time-to-Live   |  Protocol (8 bits)|               Header Checksum (16 bits)               | Row 3
|      (8 bits)     |                   |                                                       |
+-------------------+-------------------+-------------------------------------------------------+
|                             Source IP Address (32 bits / 4 bytes)                             | Row 4
+-----------------------------------------------------------------------------------------------+
|                           Destination IP Address (32 bits / 4 bytes)                          | Row 5
+-----------------------------------------------------------------------------------------------+
|                              Options + Padding (0 to 40 bytes)                                | Row 6
+-----------------------------------------------------------------------------------------------+
```

### 2.1 Field Specifications

#### Field 1: Version (VER - 4 Bits)
* Specifies the protocol version.
* For IPv4: Binary `0100` ($4_{10}$).
* If an incoming datagram contains another version (such as `0110` for IPv6), a router running only IPv4 immediately discards the frame.

#### Field 2: Header Length (HLEN - 4 Bits) & The Scaling Factor of 4
* Defines total length of the IP header in units of 4-byte (32-bit) words.
* Header size range: $20\text{ bytes}$ (minimum without options) to $60\text{ bytes}$ (maximum with options).
$$\mathbf{\text{Header Length in Bytes} = \text{HLEN Value} \times 4}$$
* *Minimum legal value:* $\text{HLEN} = 5$ ($\text{Binary: } 0101 \implies 5 \times 4 = 20\text{ bytes}$).
* *Maximum legal value:* $\text{HLEN} = 15$ ($\text{Binary: } 1111 \implies 15 \times 4 = 60\text{ bytes}$).

#### Field 3: Service Type / Differentiated Services (8 Bits)
Used for Quality of Service (QoS):
* **Bits 0–2 (Precedence - 3 bits):** Priority levels ranging from 0 (`000`: Normal) to 7 (`111`: Network Control). Under congestion, packets with lower precedence are discarded first.
* **Bits 3–6 (Type of Service / TOS - 4 bits):**
  * `1000` (D-bit): Minimize Delay (Interactive applications like Telnet, SSH, VoIP).
  * `0100` (T-bit): Maximize Throughput (Bulk transfers like FTP).
  * `0010` (R-bit): Maximize Reliability (SNMP, network management).
  * `0001` (C-bit): Minimize Cost.
  * `0000`: Normal default service.
* **Bit 7:** Reserved (must be set to 0).

#### Field 4: Total Length (16 Bits)
* Total length of the entire datagram ($\text{Header} + \text{Payload Data}$) in bytes.
* *Maximum theoretical size:* $2^{16} - 1 = \mathbf{65,535\text{ bytes}}$.
* *Minimum size:* $20\text{ bytes}$ (header with zero payload).
$$\mathbf{\text{Payload Length} = \text{Total Length} - (\text{HLEN} \times 4)}$$

#### Field 5: Identification (16 Bits)
* A counter maintained by the source machine.
* If a datagram is fragmented, this exact same 16-bit identifier is copied into all fragments so that the destination host can group them together during reassembly.

#### Field 6: Flags (3 Bits)
* **Bit 0:** Reserved (must be 0).
* **Bit 1 (D-bit: Do Not Fragment):**
  * $D = 1$: Router must not fragment this packet. If packet size exceeds the link MTU, it is dropped and an ICMP error is issued.
  * $D = 0$: Router is permitted to fragment the packet as necessary.
* **Bit 2 (M-bit: More Fragments):**
  * $M = 1$: There are more fragments following this one.
  * $M = 0$: This is the last fragment, or the datagram was never fragmented.

#### Field 7: Fragmentation Offset (13 Bits) & The Scaling Factor of 8
* Specifies the relative position of the fragment's payload data compared to the beginning of the original unfragmented datagram.
* Expressed in units of **8-byte (64-bit) blocks**:
$$\mathbf{\text{Stored Offset} = \frac{\text{Byte Offset}}{8}}$$
$$\mathbf{\text{Actual Byte Position} = \text{Stored Offset} \times 8}$$
* *Rule:* The payload size of every fragment (except the last) must be a multiple of 8 bytes.

#### Field 8: Time-to-Live (TTL - 8 Bits)
* Defines the maximum lifetime of the datagram to prevent packets from circulating indefinitely inside routing loops.
* Initialized by the source (typically 64 or 128).
* Every router that forwards the datagram decrements TTL by 1.
* When $\text{TTL} = 0$, the router drops the packet and transmits an **ICMP Time Exceeded (Type 11)** message to the source.

#### Field 9: Protocol (8 Bits)
Defines the higher-layer protocol encapsulated inside the IP datagram to enable proper demultiplexing at the receiver:
* `01` $\rightarrow$ **ICMP**
* `02` $\rightarrow$ **IGMP**
* `06` $\rightarrow$ **TCP**
* `17` $\rightarrow$ **UDP**
* `89` $\rightarrow$ **OSPF**

#### Field 10: Header Checksum (16 Bits)
* Covers the **IPv4 Header only** (payload data is not verified here to reduce router processing latency; upper-layer protocols verify their own payload).
* Calculated using 16-bit one's complement addition.
* Recalculated by **every intermediate router** because the TTL field is decremented at every hop, altering the header contents.

#### Fields 11 & 12: Source & Destination IP Addresses (32 Bits Each)
* Global logical addresses of the sender and the intended recipient.

#### Field 13: Options + Padding (0 to 40 Bytes)
* Optional settings for testing, debugging, and security:
  * **End of Option:** 1-byte padding to align the header to a 32-bit boundary.
  * **Record Route:** Routers write their IP addresses into the header. Capped at a maximum of **9 router addresses** due to the 40-byte options limit ($40 - 4\text{ overhead} = 36\text{ bytes} \implies \frac{36}{4} = 9$).
  * **Strict Source Route:** Packet must follow an exact predefined router path listed by the sender. Any deviation causes packet drop.
  * **Loose Source Route:** Packet must visit the listed routers, but is permitted to pass through intermediate unlisted routers in between.

---

## 3. Reverse Address Resolution Protocol (RARP)

RARP is used by a diskless workstation (which knows its permanent 48-bit physical MAC address burned into its ROM/NIC) to query a server for its assigned 32-bit IP address at boot time.

```text
Step 1: RARP Request (Broadcast)
   [ Diskless Client ]
   MAC: A4:6E:A5:57:82:36
   IP: Unknown (?)
         |
         v (Broadcast frame: FF:FF:FF:FF:FF:FF)
   +---------------------------------------------+
   |             Local LAN Medium                |
   +---------------------------------------------+
         /                      |              \
        v                       v               v
   [ Host 1 ]               [ Host 2 ]     [ RARP Server ]
   (Ignored)                (Ignored)      (Matches MAC in table)

Step 2: RARP Reply (Unicast)
   [ Diskless Client ] <================== [ RARP Server ]
   "Your IP is 141.14.56.21" (Unicast to client MAC)
```

* **Request:** Encapsulated in a Data Link layer broadcast frame.
* **Reply:** Returned by the RARP Server as a unicast frame directed straight to the requesting MAC.
* *Note:* Modern networks have replaced RARP with **BOOTP** and **DHCP**.

---

## 4. Internet Control Message Protocol (ICMP)

### 4.1 Deficiencies of IP Solved by ICMP
1. **Lack of Error Reporting:** IP silently discards packets when an error occurs (such as link failure or TTL expiration). It has no built-in mechanism to alert the source.
2. **Lack of Assistance/Query Mechanism:** A host cannot natively check if an intermediate gateway or remote host is operational.

ICMP compensates for these deficiencies. It is encapsulated inside an IP datagram with `Protocol = 1`.

### 4.2 Error-Reporting Messages

```text
                               ICMP Error Reporting
  +-------------------------------------------------------------------------------+
  |                 |                   |                   |                     |
  v                 v                   v                   v                     v
Destination      Source               Time              Parameter             Redirection
Unreachable      Quench              Exceeded            Problems             (Type 5)
 (Type 3)        (Type 4)            (Type 11)           (Type 12)
```

1. **Destination Unreachable (Type 3):**
   * Generated when a router cannot route the packet or the destination host cannot deliver the payload to the designated transport port.
2. **Time Exceeded (Type 11):**
   * *Code 0:* Generated when a router decrements a packet's TTL to 0. (Used by `traceroute`).
   * *Code 1:* Generated when the destination reassembly timer expires before all datagram fragments arrive.
3. **Parameter Problem (Type 12):**
   * Generated when an error or corrupted field in the header is detected (e.g., failed checksum).
4. **Source Quench (Type 4):**
   * A flow/congestion control notification sent by a congested router requesting the sender to slow its transmission rate.
5. **Redirection (Type 5):**
   * Sent by a router to an endpoint informing it to update its local routing table to use an alternative, more optimal router available on the same local segment.

#### Strict Rules on ICMP Error Generation:
* No ICMP error message will be generated in response to a datagram carrying an ICMP error message.
* No ICMP error message will be generated for a fragmented datagram that is not the first fragment ($\text{Offset} \neq 0$).
* No ICMP error message will be generated for datagrams with multicast or broadcast destination addresses.
* No ICMP error message will be generated for datagrams having special addresses like `127.0.0.0` or `0.0.0.0`.

### 4.3 Query Messages (Request / Reply Pairs)
* **Echo Request (Type 8) & Echo Reply (Type 0):**
  * Verifies end-to-end host reachability and calculates Round-Trip Time (RTT). Used by the `ping` command.
* **Router Solicitation (Type 10) & Router Advertisement (Type 9):**
  * Enables bootstrapping hosts to discover active default gateway routers on their subnet.
* **Address-Mask Request (Type 17) & Reply (Type 18):**
  * Allows a host to determine its subnet mask from a local router.
* **Timestamp Request (Type 13) & Reply (Type 14):**
  * Calculates transit delays and synchronizes system clocks between machines.

---

## 5. Internet Group Management Protocol (IGMP) & Casting Types

### 5.1 Casting Classifications

```text
                               Casting in Networks
                                        |
         +------------------------------+------------------------------+
         |                              |                              |
         v                              v                              v
    +---------+                    +-----------+                  +-----------+
    | Unicast |                    | Broadcast |                  | Multicast |
    +---------+                    +-----+-----+                  +-----------+
    (One-to-One)                         |                        (One-to-Many)
                         +---------------+---------------+
                         |                               |
                         v                               v
              +--------------------+           +-------------------+
              |  Direct Broadcast  |           | Limited Broadcast |
              +--------------------+           +-------------------+
              (NetID + All 1s)                 (255.255.255.255)
```

* **Unicast:** One-to-one delivery from a single sender to a specific recipient.
* **Multicast:** One-to-many delivery from a single sender to a specific group of subscribed hosts (Class D addresses: `224.0.0.0` to `239.255.255.255`). Managed locally by **IGMP**.
* **Limited Broadcast (`255.255.255.255`):** Delivers to all hosts on the local network. Routers never forward this packet to external networks.
* **Direct Broadcast:** Delivers to all hosts on a remote destination network. Formatted as: $\text{Target NetID} + \text{All 1s in HostID}$.

---

## 6. IPv4 Classful Addressing

An IPv4 address consists of 32 bits ($4\text{ bytes}$), yielding a theoretical address space of $2^{32} = \mathbf{4,294,967,296}$ addresses.

```text
Class A:  0 + 7 NetID bits (8 b)        |              24 HostID bits
Class B:  10 + 14 NetID bits (16 b)     |        16 HostID bits
Class C:  110 + 21 NetID bits (24 b)    |  8 HostID bits
Class D:  1110 + 28 Multicast bits      (No NetID/HostID)
Class E:  1111 + 28 Reserved bits       (No NetID/HostID)
```

### 6.1 Master Class Architecture Table

| Class | Leading Bits | 1st Octet Range | NetID Bits | HostID Bits | Total Networks | Usable Hosts / Network | Default Mask |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **A** | `0` | $0\text{--}127$ (Usable: $1\text{--}126$) | 8 | 24 | $2^7 - 2 = \mathbf{126}$ | $2^{24} - 2 = \mathbf{16,777,214}$ | `255.0.0.0` |
| **B** | `10` | $128\text{--}191$ | 16 | 16 | $2^{14} = \mathbf{16,384}$ | $2^{16} - 2 = \mathbf{65,534}$ | `255.255.0.0` |
| **C** | `110` | $192\text{--}223$ | 24 | 8 | $2^{21} = \mathbf{2,097,152}$ | $2^8 - 2 = \mathbf{254}$ | `255.255.255.0` |
| **D** | `1110`| $224\text{--}239$ | N/A | N/A | N/A (Multicasting) | N/A ($2^{28}$ group addresses) | N/A |
| **E** | `1111`| $240\text{--}255$ | N/A | N/A | N/A (Experimental) | N/A ($2^{28}$ reserved addresses) | N/A |

### 6.2 Key Address Deduction Rules
1. **Why Networks are reduced by 2 in Class A:**
   * `0.0.0.0` is reserved for default routing and local identification.
   * `127.0.0.1` is reserved for loopback self-testing.
   * Usable networks $= 2^7 - 2 = 126$.
2. **Why Usable Hosts are reduced by 2 in all networks:**
   * All 0s in HostID specifies the **Network ID**.
   * All 1s in HostID specifies the **Directed Broadcast Address**.
   * $\text{Usable Hosts} = 2^h - 2$.

---

## 7. Subnetting Principles & Numericals (FLSM & VLSM)

Subnetting divides a large network into smaller logical subnetworks to enhance security, reduce broadcast traffic, and improve administration.
* **Golden Rule:** *Subnetting borrows host bits, which increases the number of 1s in the subnet mask.*

### 7.1 Numerical 1: Dividing Class C Network `200.1.2.0` into 2 Subnets (FLSM)
* Original block: `200.1.2.0/24` (256 IP addresses).
* We need 2 subnets $\implies$ borrow 1 bit ($2^1 = 2$).
* New mask: $24 + 1 = \mathbf{/26}$ (`255.255.255.128`).
* Remaining host bits: $8 - 1 = 7 \implies 2^7 = 128\text{ addresses/subnet}$ ($126\text{ usable}$).

| Subnet | Subnet ID | Direct Broadcast | Total Range | Usable Host Range |
| :---: | :---: | :---: | :---: | :---: |
| **1st Subnet** | `200.1.2.0` | `200.1.2.127` | `200.1.2.0 - 200.1.2.127` | `200.1.2.1 - 200.1.2.126` |
| **2nd Subnet** | `200.1.2.128` | `200.1.2.255` | `200.1.2.128 - 200.1.2.255` | `200.1.2.129 - 200.1.2.254` |

### 7.2 Numerical 2: Dividing Class C Network `200.1.2.0` into 4 Subnets (FLSM)
* Borrow 2 bits ($2^2 = 4$).
* New mask: $24 + 2 = \mathbf{/26}$ (`255.255.255.192`).
* Remaining host bits: $8 - 2 = 6 \implies 2^6 = 64\text{ addresses/subnet}$ ($62\text{ usable}$).

| Subnet | Subnet ID | Direct Broadcast | Total Range | Usable Host Range |
| :---: | :---: | :---: | :---: | :---: |
| **1st Subnet** | `200.1.2.0` | `200.1.2.63` | `200.1.2.0 - 200.1.2.63` | `200.1.2.1 - 200.1.2.62` |
| **2nd Subnet** | `200.1.2.64` | `200.1.2.127` | `200.1.2.64 - 200.1.2.127` | `200.1.2.65 - 200.1.2.126` |
| **3rd Subnet** | `200.1.2.128` | `200.1.2.191` | `200.1.2.128 - 200.1.2.191` | `200.1.2.129 - 200.1.2.190` |
| **4th Subnet** | `200.1.2.192` | `200.1.2.255` | `200.1.2.192 - 200.1.2.255` | `200.1.2.193 - 200.1.2.254` |

---

## 8. Classless Addressing (CIDR) & Information Extraction

CIDR replaces class privileges by using a variable-length prefix denoted as `/n`.

### 8.1 Information Extraction Formulas
Given an address $\text{IP}/n$:
1. **Total Block Size ($N$):**
   $$N = 2^{32 - n}$$
2. **First Address (Block / Network ID):**
   * Keep the $n$ leftmost prefix bits unchanged; set the rightmost $(32 - n)$ suffix bits to all 0s.
   * *Bitwise logic:* $\text{First Address} = \text{Address } \mathbf{AND} \text{ Mask}$.
3. **Last Address (Direct Broadcast Address):**
   * Keep the $n$ leftmost prefix bits unchanged; set the rightmost $(32 - n)$ suffix bits to all 1s.
   * *Bitwise logic:* $\text{Last Address} = \text{Address } \mathbf{OR} \ [\mathbf{NOT}(\text{Mask})]$.

### 8.2 Numerical: Analyze `167.199.170.82/27`
* Prefix $n = 27$. Suffix bits $= 32 - 27 = 5$.
* **Total Addresses:** $N = 2^5 = \mathbf{32\text{ addresses}}$.
* Fourth octet $82 = 01010010_2$.
* Split into 3 prefix bits (`010`) and 5 suffix bits (`10010`):
  * **First Address:** $010\mathbf{00000}_2 = 64 \implies \mathbf{167.199.170.64/27}$
  * **Last Address:** $010\mathbf{11111}_2 = 95 \implies \mathbf{167.199.170.95/27}$

### 8.3 Three Mandatory Rules to Form a Valid CIDR Block
1. **Contiguous:** All IP addresses in the block must be in an unbroken sequence.
2. **Power of 2:** Block size must be an exact power of 2 ($N = 2^k$).
3. **Divisibility Rule:** The first IP address (converted to decimal) must be evenly divisible by the block size $N$.

---

## 9. Supernetting & Route Aggregation

Supernetting combines multiple contiguous smaller blocks into one larger block, reducing the size of routing tables.
* **Rule:** *Supernetting decreases the number of 1s in the mask.*

### 9.1 CIDR Aggregation Numerical
Aggregate the following 4 routes:
* `128.56.24.0/24`
* `128.56.25.0/24`
* `128.56.26.0/24`
* `128.56.27.0/24`

#### Solution:
1. Four networks $= 2^2$ (Power of 2).
2. Third octets: $24, 25, 26, 27$ are contiguous. $24 \pmod 4 = 0$ (Divisible).
3. Binary expansion of 3rd octets:
   * $24 = \mathbf{000110}00$
   * $25 = \mathbf{000110}01$
   * $26 = \mathbf{000110}10$
   * $27 = \mathbf{000110}11$
4. The first 6 bits match across all addresses.
5. New prefix length $= 16 + 6 = \mathbf{22\text{ bits}}$.
$$\mathbf{\text{Aggregated Supernet}} = \mathbf{128.56.24.0/22} \quad (\text{Mask: } 255.255.252.0)$$

---

## 10. Unicast Routing Protocols

```text
                           Routing Protocols
                                   |
         +-------------------------+-------------------------+
         |                                                   |
         v (Intra-Domain / IGP)                              v (Inter-Domain / EGP)
    Intradomain                                         Interdomain
         |                                                   |
   +-----+-----+                                             v
   |           |                                        Path Vector
   v           v                                             |
Distance     Link State                                      v
 Vector         |                                           BGP
   |            v
   v           OSPF
  RIP
```

### 10.1 Autonomous System (AS)
An Autonomous System is a collection of networks and routers managed by a single administrative entity sharing a common routing policy.
* **Intra-Domain Routing (IGP):** Routing inside an AS (e.g., RIP, OSPF).
* **Inter-Domain Routing (EGP):** Routing between different ASs (e.g., BGP).

---

## 11. Distance Vector Routing (DVR) & Instability

* **Philosophy:** *Each router tells its immediate neighbors what it knows about the whole internet.*
* **Algorithm Basis:** Bellman-Ford Shortest Path Algorithm.
* **Operation:** Routers share their vectors periodically (e.g., RIP sends updates every 30 seconds) or via triggered updates upon link change.

```text
DVR Table Sharing:
[ Router A ] <====== (Shares Cost Vector) ======> [ Router B ]
(Only direct neighbors exchange information)
```

### 11.1 The "Count-to-Infinity" Problem (Two-Node Instability)
When a link fails, routers may continuously bounce incremental updates back and forth due to routing loops:
1. If link $A-X$ fails, A sets cost to $X = \infty$.
2. Before A updates B, B broadcasts its old table: *"I can reach X with cost 2"*.
3. A assumes B has a valid alternative route and updates its cost to $2 + 1 = 3$.
4. B receives A's update and increases its cost to $3 + 1 = 4$.
5. Costs increment mutually until reaching the protocol infinity ($\infty = 16$ in RIP).

### 11.2 Solution: Split Horizon
* **Rule:** If Router B learned its route to destination X through Router A, **Router B must not advertise the route to X back to Router A**.
* Eliminates the false loop by removing the misleading path entry from the update sent to A.

---

## 12. Link State Routing (LSR) & Dijkstra's Algorithm

* **Philosophy:** *Each router tells the whole internet what it knows about its immediate neighbors.*
* **Algorithm Basis:** Dijkstra's Algorithm.

### 12.1 The Four Operational Steps:
1. **LSP Creation:** Every node discovers its neighbors and measures link cost to create a **Link State Packet (LSP)**.
2. **Flooding:** Each node floods its LSP to every router in the network domain.
3. **LSDB Compilation:** Every router builds an identical **Link State Database (LSDB)** matrix containing the full network topology graph.
4. **Shortest Path Tree Execution:** Each router runs Dijkstra's algorithm independently, placing itself at the root to generate a shortest path tree and derive its routing table.

```text
Dijkstra Execution Walkthrough:
1. Divide nodes into Permanent (locked) and Tentative sets.
2. Select the tentative node with minimum cumulative cost; mark it Permanent.
3. Update (relax) neighbor costs through the newly permanent node.
4. Repeat until all network nodes are marked Permanent.
```

---

## 13. Comprehensive Comparison: DVR vs. LSR

| Metric | Distance Vector Routing (DVR) | Link State Routing (LSR) |
| :--- | :--- | :--- |
| **Information Shared** | Entire routing table | State of local links only (LSP) |
| **Recipient** | Immediate neighbors only | All routers in the domain (Flooding) |
| **Topology Knowledge** | Local knowledge (neighbor hearsay) | Global knowledge (full topology map) |
| **Underlying Algorithm**| Bellman-Ford | Dijkstra's Algorithm |
| **Convergence Speed** | Slow convergence | Rapid convergence |
| **Loop Vulnerability** | Suffers from Count-to-Infinity | Free from Count-to-Infinity loops |
| **Bandwidth & CPU** | Low CPU, periodic table overhead | High CPU (Dijkstra), flood burst |
| **Standard Protocol** | **RIP (Routing Information Protocol)** | **OSPF (Open Shortest Path First)** |

---

## 14. Important BEU Examination Questions & Answers

### Q1. Explain the IPv4 header format with a neat diagram. Discuss the functions of HLEN, Total Length, and TTL fields. (14 Marks)
**Model Answer Structure:**
1. **Diagram:** Draw the 32-bit wide IPv4 header grid clearly showing all fields across rows 1 to 6.
2. **HLEN:** 4-bit field defining header size in 4-byte words ($\text{Header Bytes} = \text{HLEN} \times 4$). Base header $= 20\text{ bytes}$ ($\text{HLEN} = 5$); maximum $= 60\text{ bytes}$ ($\text{HLEN} = 15$).
3. **Total Length:** 16-bit field defining the entire datagram size ($\text{Header} + \text{Data}$). Maximum value is $65,535\text{ bytes}$. Payload size is extracted as: $\text{Total Length} - (\text{HLEN} \times 4)$.
4. **Time-to-Live (TTL):** 8-bit hop counter decremented by 1 at every router. When $\text{TTL} = 0$, the packet is discarded and an ICMP Time Exceeded message is issued, preventing infinite routing loops.

---

### Q2. What is CIDR? For an assigned address `167.199.170.82/27`, calculate: (a) Total number of addresses, (b) First address, and (c) Last address. (7 / 14 Marks)
**Model Steps:**
1. State CIDR definition: Eliminates classful boundaries by using a `/n` prefix length.
2. Identify parameters: $n = 27$ bits $\implies$ Host bits $= 32 - 27 = 5$ bits.
3. Total addresses: $N = 2^5 = \mathbf{32\text{ addresses}}$.
4. Fourth octet binary expansion: $82 = 01010010_2$.
   * First address (Network ID): Set 5 suffix bits to 0s: $01000000_2 = 64 \implies \mathbf{167.199.170.64/27}$.
   * Last address (DBA): Set 5 suffix bits to 1s: $01011111_2 = 95 \implies \mathbf{167.199.170.95/27}$.

---

### Q3. Explain the "Count-to-Infinity" problem in Distance Vector Routing. How does Split Horizon resolve it? (7 Marks)
**Model Answer:**
1. **Cause:** DVR routers report minimum distances based on neighbor advertisements without knowing if their own path traverses that neighbor.
2. **Failure Sequence:** When a link fails, two adjacent routers update each other incrementally ($2, 3, 4\dots$) in a mutually dependent loop until the metric reaches infinity.
3. **Split Horizon Solution:** Enforces the rule that a router never advertises a route back out of the interface through which it learned that route, breaking the update loop immediately upon link failure.

---

### Q4. Differentiate between Distance Vector Routing and Link State Routing. (7 Marks)
**Model Answer:**
Provide the 8-point comparison table covering Philosophy, Algorithm (Bellman-Ford vs. Dijkstra), Convergence speed, Knowledge scope (Local vs. Global), Loop vulnerability (Count-to-Infinity), and Standard protocols (RIP vs. OSPF).

---

## 15. Objective Questions (MCQ Bank)

1. **What is the scaling factor for the IPv4 HLEN field?**
   * (a) 2
   * (b) 4
   * (c) 8
   * (d) 16
   * *Answer:* **(b) 4**

2. **The Fragmentation Offset in an IPv4 header is measured in units of:**
   * (a) 1 byte
   * (b) 4 bytes
   * (c) 8 bytes
   * (d) 16 bytes
   * *Answer:* **(c) 8 bytes**

3. **In an IPv4 datagram, which protocol number represents TCP?**
   * (a) 1 (ICMP)
   * (b) 2 (IGMP)
   * (c) 6
   * (d) 17 (UDP)
   * *Answer:* **(c) 6**

4. **The Limited Broadcast Address for any IPv4 network is:**
   * (a) `0.0.0.0`
   * (b) `127.0.0.1`
   * (c) `255.255.255.255`
   * (d) `224.0.0.1`
   * *Answer:* **(c) `255.255.255.255`**

5. **How many usable networks exist in IPv4 Class A?**
   * (a) 128
   * (b) 126 ($2^7 - 2$)
   * (c) 16,384
   * (d) 2,097,152
   * *Answer:* **(b) 126**

6. **Subnetting always causes:**
   * (a) Decrease in the number of 1s in the mask
   * (b) Increase in the number of 1s in the mask
   * (c) Decrease in the total number of subnets
   * (d) Removal of the router
   * *Answer:* **(b) Increase in the number of 1s in the mask**

7. **The `ping` utility relies on which pair of ICMP messages?**
   * (a) Type 3 and Type 11
   * (b) Type 8 (Echo Request) and Type 0 (Echo Reply)
   * (c) Type 13 and Type 14
   * (d) Type 17 and Type 18
   * *Answer:* **(b) Type 8 (Echo Request) and Type 0 (Echo Reply)**

8. **Which routing protocol uses Dijkstra's Algorithm to compute the shortest path tree?**
   * (a) RIP
   * (b) OSPF (Link State Routing)
   * (c) BGP
   * (d) RARP
   * *Answer:* **(b) OSPF (Link State Routing)**

9. **In CIDR, combining four `/24` networks into a single supernet results in a prefix length of:**
   * (a) `/22` ($24 - \log_2 4 = 22$)
   * (b) `/23`
   * (c) `/26`
   * (d) `/28`
   * *Answer:* **(a) `/22`**

10. **The Split Horizon mechanism is used to solve which problem in Distance Vector Routing?**
    * (a) Packet fragmentation
    * (b) Count-to-Infinity instability
    * (c) Checksum errors
    * (d) Buffer congestion
    * *Answer:* **(b) Count-to-Infinity instability**