# Chapter 4: Transport Layer, Security & Data Compression

---

## 1. Transport-Layer Services & Architecture

### 1.1 Overview & Responsibilities
The Transport Layer (Layer 4 of the OSI Reference Model) serves as the core liaison between the application software processes and the underlying network communication infrastructure.

```text
       Host A (Client)                                  Host B (Server)
+-----------------------------+                  +-----------------------------+
| Application Process (P1)    |                  | Application Process (P1)    |
| (e.g., Web Browser)         |                  | (e.g., Web Server)          |
+--------------+--------------+                  +--------------^--------------+
               |                                                |
               v                                                |
+--------------+--------------+                  +--------------+--------------+
|       Transport Layer       | ===[ End-to-End Channel ]======>|       Transport Layer       |
|    (Port-to-Port Delivery)  |                  |    (Port-to-Port Delivery)  |
+--------------+--------------+                  +--------------+--------------+
               |                                                ^
               v                                                |
+--------------+--------------+                  +--------------+--------------+
|        Network Layer        | ===[ Host-to-Host Delivery ]===>|        Network Layer        |
|    (Logical IP Addressing)  |                  |    (Logical IP Addressing)  |
+-----------------------------+                  +-----------------------------+
```

#### Core Differences: Host-to-Host vs. Process-to-Process Delivery
* **Network Layer Domain (Host-to-Host):**
  * The network layer (IP) is only responsible for routing a datagram across multiple intermediate links from the source host computer to the destination host computer.
  * Once the datagram arrives at the destination computer, the Network layer's job is complete. It cannot determine which specific running application should receive the incoming data.
* **Transport Layer Domain (Process-to-Process / End-to-End):**
  * Modern computers execute dozens of concurrent processes (e.g., Chrome, Zoom, WhatsApp, Spotify).
  * The transport layer protocol is responsible for delivering the message to the **exact application program (process)** running on the host computer.
* **Hinglish Explanation:**
  * Network layer ka kaam sirf packet ko ek computer se doosre computer ke darwaze tak chhodna hota hai (Host-to-Host). Lekin us computer ke andar ek sath kayi applications chal rahi hoti hain. Transport layer packet ko sahi chal rahe software (Process) tak pahunchata hai. Isliye ise **Process-to-Process ya End-to-End communication** kehte hain.

---

### 1.2 Transport Layer Service Models

```text
                       Transport Service Models
                                   |
         +-------------------------+-------------------------+
         |                                                   |
         v                                                   v
+-----------------------------+           +-----------------------------+
|    Connectionless Service   |           | Connection-Oriented Service |
+-----------------------------+           +-----------------------------+
| - No handshake setup        |           | - 3-Way Handshake setup     |
| - Independent datagrams     |           | - Maintained state channel  |
| - Low overhead, fast (UDP)  |           | - Ordered delivery & ACK    |
|                             |           | - Graceful teardown (TCP)   |
+-----------------------------+           +-----------------------------+
```

#### 1. Connectionless vs. Connection-Oriented
* **Connectionless Transport:**
  * The sender transmits individual segments without performing any prior handshake with the destination machine.
  * Each segment is handled as an independent entity containing destination address information.
  * *Example:* **UDP (User Datagram Protocol)**.
* **Connection-Oriented Transport:**
  * Establishes a formal logical connection between endpoints before sending data bytes (Connection Establishment).
  * Manages state information throughout transmission and gracefully closes the connection once data transfer ends (Connection Termination).
  * *Example:* **TCP (Transmission Control Protocol)**.

#### 2. Reliable vs. Unreliable Service
* **Reliable Transport:**
  * Implements end-to-end flow control, packet sequencing, error detection, and retransmissions for lost or corrupted segments.
  * *Trade-off:* Introduces processing overhead and transmission latency.
* **Unreliable Transport:**
  * Provides best-effort transmission without acknowledging packets or initiating retransmissions.
  * *Trade-off:* Provides minimal protocol delay and high throughput, which is essential for real-time applications (VoIP, video conferencing).

---

## 2. Port Addressing & Socket Architecture

### 2.1 16-Bit Port Numbers
To uniquely distinguish between multiple application processes running on the same host, the transport layer utilizes **Port Numbers**.
* **Size:** 16-bit unsigned integer.
* **Range:** $0$ to $2^{16} - 1$ ($\mathbf{0\text{ to }65,535}$).

```text
 0                     1023 1024                49,151 49,152             65,535
+--------------------------+--------------------------+-------------------------+
|     Well-Known Ports     |     Registered Ports     |   Dynamic / Ephemeral   |
|       (0 to 1023)        |     (1024 to 49,151)     |    (49,152 to 65,535)   |
+--------------------------+--------------------------+-------------------------+
```

#### IANA Classification: The Three Port Ranges
1. **Well-Known Ports ($0\text{ to }1023$):**
   * Strictly controlled and assigned by the **Internet Assigned Numbers Authority (IANA)**.
   * Reserved for standard, universal server processes. Clients must know these ports in advance to request services.
2. **Registered Ports ($1024\text{ to }49,151$):**
   * Not strictly controlled by IANA, but can be registered by companies and software developers to avoid port conflict duplication between applications.
3. **Dynamic / Private / Ephemeral Ports ($49,152\text{ to }65,535$):**
   * Neither registered nor controlled by IANA.
   * **Ephemeral means "short-lived":** When a client program initiates an outbound connection, the local operating system temporarily assigns an unused port from this range. When the process finishes, the port is freed.

---

### 2.2 Standard Well-Known Ports Reference Table

| Port Number | Protocol | Application / Service Description |
| :---: | :---: | :--- |
| **`20`** | TCP | **FTP Data** (File Transfer Protocol - Data transfer channel) |
| **`21`** | TCP | **FTP Control** (Commands, directory navigation, credentials) |
| **`22`** | TCP | **SSH** (Secure Shell - Encrypted remote login) |
| **`23`** | TCP | **TELNET** (Unencrypted remote terminal session) |
| **`25`** | TCP | **SMTP** (Simple Mail Transfer Protocol - Mail forwarding) |
| **`53`** | UDP / TCP | **DNS** (Domain Name System - Name-to-IP resolution) |
| **`67 / 68`** | UDP | **DHCP / BOOTP** (Automatic IP configuration: 67=Server, 68=Client) |
| **`69`** | UDP | **TFTP** (Trivial File Transfer Protocol) |
| **`80`** | TCP | **HTTP** (HyperText Transfer Protocol - Web browsing) |
| **`110`** | TCP | **POP3** (Post Office Protocol v3 - Mail retrieval) |
| **`161`** | UDP | **SNMP** (Simple Network Management Protocol) |
| **`443`** | TCP | **HTTPS** (HTTP over TLS/SSL - Secure encrypted browsing) |
| **`16384--32767`** | UDP | **RTP** (Real-Time Transport Protocol - VoIP audio/video streaming) |

---

### 2.3 Socket Address Architecture
An IP address identifies the destination machine on the network, while a port number identifies the application process inside that machine. The combination of both forms a **Socket Address**.

$$\mathbf{\text{Socket Address} = \text{IP Address } (32\text{ bits}) + \text{Port Number } (16\text{ bits}) = 48\text{ bits}}$$

```text
   IP Address (32 bits)       Port Number (16 bits)
      [ 200.23.56.8 ]     +           [ 69 ]
             |                          |
             v                          v
+-------------------------------------------------------+
|   Socket Address:      200.23.56.8 : 69               |
+-------------------------------------------------------+
```

* To establish a full-duplex transport connection, a **Socket Pair** is required:
  $$\text{Connection} = (\text{Client IP} : \text{Client Port}, \ \text{Server IP} : \text{Server Port})$$

---

### 2.4 Encapsulation & Decapsulation

```text
         SENDER (Client)                                          RECEIVER (Server)
+-------------------------------+                       +-------------------------------+
|       Application Layer       |                       |       Application Layer       |
|    [ Process creates Msg ]    |                       |    [ Process receives Msg ]   |
+---------------+---------------+                       +---------------+---------------+
                |                                                       ^
                v Message (Data)                                        | Message
+---------------+---------------+                       +---------------+---------------+
|        Transport Layer        |                       |        Transport Layer        |
|  Attaches Header (Ports, etc.)|                       | Drops Header & validates data |
|  [ Header | Payload Data ]    |                       |                               |
+---------------+---------------+                       +---------------+---------------+
                |                                                       ^
                v Transmitted across Logical Channel                    |
    ====================================================================+
```

* **Encapsulation:** The sender's transport layer receives data from the application layer, attaches a transport header (port addresses, sequence numbers, checksum), and produces a **Segment** (in TCP) or **User Datagram** (in UDP).
* **Decapsulation:** The receiver's transport layer strips the transport header, checks for transmission errors, and passes the original data payload to the destination application process bound to that port.

---

## 3. Transmission Control Protocol (TCP) Architecture

### 3.1 The Byte-Stream & Virtual Connection Model
TCP provides a connection-oriented, reliable, full-duplex **byte-stream service**.
* TCP does not view data as individual packets. It views data as a continuous sequence of incoming and outgoing bytes.
* TCP creates the operational abstraction of an imaginary, bidirectional **"tube or pipeline"** running across the Internet between the sending and receiving processes.

```text
Sending Process ===> [ TCP Sending Buffer ] ===> { Stream of Bytes in Pipe } ===> [ TCP Receiving Buffer ] ===> Receiving Process
```

#### Core Control Mechanisms:
1. **Byte-Oriented Flow Control:** Prevents the sender from overwhelming the receiver's memory buffer using a dynamic Sliding Window.
2. **Byte-Oriented Error Control:** Tracks delivery per byte using sequence numbers, cumulative acknowledgments, and retransmission timers.
3. **Congestion Control:** Modulates transmission rate based on intermediate network congestion, preventing network collapse.

---

### 3.2 TCP Segment Header Architecture

```text
 0                   15 16                                  31
+----------------------+------------------------------------+
|  Source Port Address |     Destination Port Address       |  Row 1 (4 Bytes)
|       (16 bits)      |             (16 bits)              |
+----------------------+------------------------------------+
|                     Sequence Number                       |  Row 2 (4 Bytes)
|                         (32 bits)                         |
+-----------------------------------------------------------+
|                  Acknowledgment Number                    |  Row 3 (4 Bytes)
|                         (32 bits)                         |
+-------+--------+-----+--+--+--+--+--+---------------------+
| HLEN  |Reserved| U | A| P| R| S| F|     Window Size       |  Row 4 (4 Bytes)
|(4 b)  | (6 b)  | R | C| S| S| Y| I|       (16 bits)       |
|       |        | G | K| H| T| N| N|                       |
+-------+--------+---+--+--+--+--+--+-----------------------+
|        Checksum (16 bits)    |    Urgent Pointer (16 bits)|  Row 5 (4 Bytes)
+------------------------------+----------------------------+
|               Options and Padding (0 to 40 bytes)         |  Row 6 (Variable)
+-----------------------------------------------------------+
|<-------------------------- 32 Bits ---------------------->|
```

* **Header Size:** $20\text{ bytes}$ (without options) up to $60\text{ bytes}$ (with maximum options).

---

### 3.3 Field-by-Field Specifications

#### 1. Source Port & Destination Port ($16\text{ bits each}$):
* Defines the sending process port and receiving process port.

#### 2. Sequence Number ($32\text{ bits}$):
* Identifies the byte number assigned to the **very first data byte** carried in this segment.
* Sequence numbers range from $0$ to $2^{32} - 1$.
* **Initial Sequence Number (ISN):** During connection establishment, each side randomly chooses an ISN to prevent old, delayed duplicate packets from overlapping with a new session.

#### 3. Acknowledgment Number ($32\text{ bits}$):
* TCP utilizes **Cumulative Acknowledgments**.
* Contains the **next sequence byte number** that the receiver expects to receive.
$$\mathbf{\text{ACK Number} = (\text{Last In-Order Byte Received}) + 1}$$
* Valid only if the **ACK control flag** is set to 1.

#### 4. Header Length (HLEN / Data Offset - $4\text{ bits}$) & The Scaling Factor of 4:
* Indicates the size of the TCP header in units of 4-byte words:
$$\mathbf{\text{Header Length in Bytes} = \text{HLEN Value} \times 4}$$
* *Minimum HLEN:* $\text{HLEN} = 5$ ($5 \times 4 = \mathbf{20\text{ bytes}}$).
* *Maximum HLEN:* $\text{HLEN} = 15$ ($15 \times 4 = \mathbf{60\text{ bytes}}$).

#### 5. Reserved ($6\text{ bits}$):
* Reserved for future implementation; must be set to all zeros.

#### 6. Control Flags ($6\text{ bits}$):
* **URG (Urgent):** Indicates that the Urgent Pointer field is valid.
* **ACK (Acknowledgment):** Confirms that the Acknowledgment Number field contains valid data.
* **PSH (Push):** Directs the transmitting TCP to flush its buffer immediately and instructs the receiving TCP to push data straight to the application layer without buffering delay.
* **RST (Reset):** Abruptly aborts and tears down an abnormal connection (e.g., when a packet arrives at an unopened port).
* **SYN (Synchronize):** Synchronizes sequence numbers during the connection establishment phase.
* **FIN (Finish):** Initiates graceful connection termination; indicates that the sender has finished transmitting data.

#### 7. Window Size ($16\text{ bits}$):
* The receiver's advertised window size (`rwnd`) in bytes. Dictates how many bytes the sender can transmit before waiting for an ACK.
* Base capacity: $2^{16} - 1 = \mathbf{65,535\text{ bytes}}$ ($\approx 64\text{ KB}$).
* **Window Scaling Option:** In high-speed gigabit networks, a 14-bit scale factor can be negotiated in the Options field, expanding the effective window up to $1\text{ Gigabyte}$ ($2^{30}\text{ bytes}$).

#### 8. Checksum ($16\text{ bits}$):
* Mandatory in TCP. Calculated over the TCP Header, the Data Payload, and an IP **Pseudo-Header** (Source IP, Destination IP, Protocol = 6, and Segment Length) to prevent misrouting errors.

#### 9. Urgent Pointer ($16\text{ bits}$):
* Valid only when the URG flag is set.
* Defines the offset that, when added to the Sequence Number, yields the byte number of the last urgent byte:
$$\mathbf{\text{Last Urgent Byte} = \text{Sequence Number} + \text{Urgent Pointer}}$$

---

## 4. TCP Connection Management

### 4.1 Connection Establishment: Three-Way Handshaking
Before any data exchange can begin, TCP endpoints perform a three-way handshake to synchronize their Initial Sequence Numbers (ISNs) and buffer parameters.

```text
    Client Host                                                    Server Host
(Active Open)                                                  (Passive Open)
         |                                                              |
         | -------- Step 1: SYN (seq: 8000) --------------------------> |
         |          [SYN = 1, ACK = 0]                                  |
         |                                                              |
         | <------- Step 2: SYN + ACK (seq: 15000, ack: 8001) -------- |
         |          [SYN = 1, ACK = 1, rwnd: 5000]                      |
         |                                                              |
         | -------- Step 3: ACK (seq: 8001, ack: 15001) -------------> |
         |          [ACK = 1, rwnd: 10000]                              |
         v                                                              v
 [Connection ESTABLISHED]                                       [Connection ESTABLISHED]
```

#### Step-by-Step Breakdown:
1. **Step 1: The SYN Segment (Client $\rightarrow$ Server):**
   * Client initiates an **Active Open**.
   * Flags: `SYN = 1`, `ACK = 0`.
   * Sets initial sequence number: `seq: 8000`.
   * *Rule:* Carries no application data, but **consumes exactly 1 sequence number**.
2. **Step 2: The SYN + ACK Segment (Server $\rightarrow$ Client):**
   * Server executes a **Passive Open** and accepts the connection request.
   * Flags: `SYN = 1`, `ACK = 1`.
   * Acknowledges the client's SYN: `ack: 8001` ($8000 + 1$).
   * Announces its own initial sequence number: `seq: 15000`.
   * Advertises its receive buffer capacity: `rwnd: 5000`.
   * *Rule:* Consumes 1 sequence number.
3. **Step 3: The ACK Segment (Client $\rightarrow$ Server):**
   * Client confirms the server's parameters.
   * Flags: `ACK = 1` (`SYN` is now 0).
   * Sequence number: `seq: 8001`.
   * Acknowledges the server's SYN: `ack: 15001` ($15000 + 1$).
   * Advertises client window: `rwnd: 10000`.
   * Connection is established in both directions (full-duplex).

---

## 5. User Datagram Protocol (UDP) Architecture

### 5.1 Protocol Overview
UDP is an unreliable, connectionless transport protocol providing lightweight, minimal-overhead process-to-process delivery.

```text
 0                   15 16                                  31
+----------------------+------------------------------------+
|  Source Port Number  |     Destination Port Number        |  Row 1 (4 Bytes)
|       (16 bits)      |             (16 bits)              |
+----------------------+------------------------------------+
|     Total Length     |              Checksum              |  Row 2 (4 Bytes)
|       (16 bits)      |             (16 bits)              |
+----------------------+------------------------------------+
|<-------------------------- 32 Bits ---------------------->|
```

* **Fixed Header Size:** Exactly **$8\text{ Bytes}$** (No Options field).

#### Field Descriptions:
1. **Source Port ($16\text{ bits}$):** Identifies the sending application process.
2. **Destination Port ($16\text{ bits}$):** Identifies the target receiving application process.
3. **Total Length ($16\text{ bits}$):** Total size of the user datagram ($\text{Header } [8\text{ B}] + \text{Payload Data}$) in bytes. Minimum value is $8$.
4. **Checksum ($16\text{ bits}$):** 16-bit one's complement checksum covering the header, payload, and an IP pseudo-header. In IPv4, calculating the UDP checksum is **optional** (set to 0 if unused).

---

### 5.2 Functional Comparison: TCP vs. UDP

| Operational Metric | TCP | UDP |
| :--- | :--- | :--- |
| **Connection Model** | Connection-oriented (Handshake required) | Connectionless (No handshake) |
| **Reliability** | Reliable (Retransmissions, ordering, ACKs) | Unreliable (Best-effort delivery) |
| **Header Size** | $20\text{ to }60\text{ Bytes}$ (Variable) | Fixed $8\text{ Bytes}$ |
| **Data Stream Model**| Byte-stream abstraction | Independent message datagrams |
| **Flow Control** | Yes (Sliding Window) | None |
| **Congestion Control**| Yes (Window back-off mechanisms) | None |
| **Multicast Support**| No (Point-to-point unicast only) | Yes (Supports Multicast & Broadcast) |
| **Typical Protocols**| HTTP, HTTPS, FTP, SMTP, SSH | DNS, DHCP, TFTP, SNMP, VoIP, RIP |

---

### 5.3 TCP vs. RTP (Real-Time Transport Protocol)

| Parameter | TCP | RTP |
| :--- | :--- | :--- |
| **Protocol Type** | Connection-oriented, Lossless | Stateless, Real-time (Operates on top of UDP) |
| **Latency & Overhead**| Slower due to flow/congestion control | Fast with minimal transmission latency |
| **Packet Loss Handling**| Retransmits lost segments immediately | Discards late or corrupted packets to prevent jitter |
| **Primary Deployment**| Web browsing, banking, file transfer | Video calling (Zoom, Teams), VoIP, IPTV |

---

## 6. Data Compression (Lossless vs. Lossy)

Data compression reduces data volume to optimize storage efficiency and minimize transmission latency across network links.

```text
[ Original Data ] ===> [ ENCODER ] ===> [ Compressed Bits ] ===> [ DECODER ] ===> [ Reconstructed Data ]
```

```text
                           Data Compression
                                  |
         +------------------------+------------------------+
         |                                                 |
         v                                                 v
+-----------------------------+           +-----------------------------+
|    Lossless Compression     |           |      Lossy Compression      |
+-----------------------------+           +-----------------------------+
| - Redundant bits eliminated |           | - Imperceptible data dropped|
| - Exact original recovered  |           | - Irreversible process      |
| - Moderate compression ratio|           | - High compression ratio    |
| - Text, Source Code, ZIP    |           | - Audio (MP3), Video (MP4)  |
+-----------------------------+           +-----------------------------+
```

### Comparative Summary:
1. **Lossless Compression (Data Compaction):**
   * Removes mathematical redundancies without discarding actual information.
   * Fully reversible: the reconstructed output is bit-for-bit identical to the input.
   * *Applications:* Executable binaries, text documents, medical databases, ZIP archives, PNG images.
2. **Lossy Compression:**
   * Permanently discards details that the human sensory system (vision/hearing) cannot easily perceive.
   * Irreversible: original data cannot be reconstructed perfectly.
   * Yields significantly higher compression ratios.
   * *Applications:* JPEG photography, MP3 audio, MP4/H.264 video streaming.

---

## 7. Cryptography & Network Security Fundamentals

Cryptography secures communication across untrusted, public communication media against active and passive adversaries.

```text
  [ SENDER ]                                                               [ RECEIVER ]
      |                                                                         ^
[ Plain Text (P) ]                                                      [ Plain Text (P) ]
      |                                                                         |
      v                                                                         |
+----------------------+                                             +----------------------+
| Encryption Algorithm |                                             | Decryption Algorithm |
|     C = E_k(P)       |                                             |     P = D_k(C)       |
+----------+-----------+                                             +----------^-----------+
           |                                                                    |
           v                                                                    |
  [ Cipher Text (C) ] ===== (Insecure Physical / Wireless Medium) =====> [ Cipher Text (C) ]
```

### 7.1 The Four Pillars of Information Security:
1. **Confidentiality:** Guarantees that only the authorized recipient can read the message content.
2. **Integrity:** Ensures the message has not been altered, modified, or tampered with during transit.
3. **Authentication:** Verifies the true identity of the transmitting sender.
4. **Non-Repudiation:** Prevents the sender from falsely denying having sent the message.

---

### 7.2 Classical Ciphers: The Caesar Cipher
The Caesar Cipher is a historical mono-alphabetic substitution cipher where each plaintext character is shifted by a constant integer key ($k = 3$).

$$\mathbf{C = (P + k) \pmod{26}}$$
$$\mathbf{P = (C - k) \pmod{26}}$$

```text
Plain:   A  B  C  D  E  F  G  H  I  J  K  L  M  N  O  P  Q  R  S  T  U  V  W  X  Y  Z
Cipher:  D  E  F  G  H  I  J  K  L  M  N  O  P  Q  R  S  T  U  V  W  X  Y  Z  A  B  C
```
* *Vulnerability:* The key space has only $25$ possibilities, making it trivially vulnerable to brute-force frequency analysis attacks.

---

### 7.3 Symmetric-Key vs. Asymmetric-Key Cryptography

```text
Symmetric-Key Model:
[ Plaintext ] ---> [ Encrypt ] ===(Ciphertext)===> [ Decrypt ] ---> [ Plaintext ]
                         ^                               ^
                         +------- Shared Secret Key -----+

Asymmetric-Key Model:
[ Plaintext ] ---> [ Encrypt ] ===(Ciphertext)===> [ Decrypt ] ---> [ Plaintext ]
                         ^                               ^
                Recipient Public Key            Recipient Private Key
```

#### Detailed Comparison Table:

| Security Parameter | Symmetric-Key Cryptography | Asymmetric-Key Cryptography |
| :--- | :--- | :--- |
| **Key Architecture** | Single Shared Secret Key | Two mathematically related keys (Public & Private) |
| **Key Distribution** | Difficult; secret key must be exchanged safely | Trivial; public key is shared openly |
| **Encryption Key** | Shared Secret Key ($K$) | Recipient's Public Key ($K_{\text{pub}}$) |
| **Decryption Key** | Shared Secret Key ($K$) | Recipient's Private Key ($K_{\text{priv}}$) |
| **Computational Speed**| Extremely fast (High throughput) | Slow (Computationally intensive math) |
| **Key Management** | $\frac{N(N - 1)}{2} \approx \mathbf{O(N^2)}$ keys for $N$ users | $\mathbf{2N}$ keys for $N$ users |
| **Primary Use Cases** | Bulk data encryption (hard drives, TLS payload) | Key exchange, authentication, digital signatures |
| **Standard Algorithms**| **DES, Triple-DES (3DES), AES, Blowfish** | **RSA, Diffie-Hellman, ECC** |

---

### 7.4 Data Encryption Standard (DES) Architecture
* **Algorithm Class:** Symmetric Block Cipher based on a Feistel network.
* **Block Size:** $64\text{ bits}$ of plaintext produces $64\text{ bits}$ of ciphertext.
* **Key Length:** $64\text{ bits}$ input, but $8\text{ bits}$ are used for parity checking, yielding an **effective key length of $56\text{ bits}$**.
* **Structure:** $16$ identical rounds of substitution (via non-linear S-Boxes) and permutation (via P-Boxes).
* **Modern Status:** A 56-bit key can be broken by modern brute-force hardware in hours. It has been replaced by **Triple-DES (3DES)** and **AES (Advanced Encryption Standard: 128/192/256-bit keys)**.

---

## 8. Important BEU Examination Questions & Answers

### Q1. Explain the TCP Three-Way Handshaking mechanism for connection establishment. Why is it used? (14 Marks)
**Model Answer Structure:**
1. **Definition & Purpose:**
   * TCP is a connection-oriented protocol that requires mutual synchronization of Initial Sequence Numbers (ISNs) and buffer parameters before transferring data.
   * Prevents old duplicate segments from causing spurious connections.
2. **Three-Way Handshake Steps:**
   * **Step 1 (SYN):** The client sends a segment with `SYN = 1`, `ACK = 0`, and an initial sequence number `seq: x`. This segment consumes 1 sequence number.
   * **Step 2 (SYN + ACK):** The server responds with `SYN = 1`, `ACK = 1`, acknowledging the client's SYN with `ack: x + 1`, and sends its own initial sequence number `seq: y` and advertised window size `rwnd`. This segment consumes 1 sequence number.
   * **Step 3 (ACK):** The client sends `ACK = 1`, `seq: x + 1`, and `ack: y + 1`, confirming the server's parameters.
3. **Diagram:** Draw the sequence diagram showing client-server interaction with explicit segment values (`seq: 8000`, `ack: 8001`, etc.).

---

### Q2. Draw the TCP Header format and explain the function of each field. (14 Marks)
**Model Steps:**
1. **Header Layout:** Draw the full 32-bit wide TCP header diagram showing all 6 rows clearly.
2. **Port Numbers:** Explain 16-bit Source and Destination port addresses.
3. **Sequencing:** Detail the 32-bit Sequence Number and 32-bit Cumulative Acknowledgment Number.
4. **Header Length & Reserved:** Explain the 4-bit HLEN with its scaling factor of 4 ($20\text{--}60\text{ bytes}$) and the 6 reserved bits.
5. **Control Flags:** Explain URG, ACK, PSH, RST, SYN, and FIN.
6. **Window Size & Checksum:** Detail the 16-bit sliding window flow control and the 16-bit checksum calculated over the TCP segment and the IP pseudo-header.

---

### Q3. Differentiate between TCP and UDP. When is UDP preferred over TCP? (7 Marks)
**Model Answer Structure:**
1. **Comparison Table:** Provide a structured comparison covering connection setup, reliability, header size ($20\text{--}60\text{ B}$ vs. $8\text{ B}$), flow control, and transmission speed.
2. **Why UDP is Preferred:**
   * Eliminates connection setup and teardown latency.
   * Operates without retransmission delays, preventing jitter in continuous media streams.
   * Extremely small header overhead ($8\text{ bytes}$), saving network bandwidth.
   * Supports broadcast and multicast communication channels.

---

### Q4. Compare Symmetric-Key and Asymmetric-Key Cryptography. Explain how public-key cryptography solves the key distribution problem. (7 Marks)
**Model Steps:**
1. **Comparison Table:** Contrast single shared secret key vs. public-private key pairs, execution speed, mathematical complexity, and key scaling formulas ($\frac{N(N-1)}{2}$ vs. $2N$).
2. **Key Distribution Solution:**
   * In symmetric systems, sharing a secret key across an unencrypted channel exposes it to interception.
   * In asymmetric systems, the public key is published openly. The private key never leaves the owner's possession, making interception harmless.

---

## 9. Objective Questions (MCQ Bank)

1. **What is the size of a standard port number in the TCP/IP suite?**
   * (a) 8 bits
   * (b) 16 bits
   * (c) 32 bits
   * (d) 48 bits
   * *Answer:* **(b) 16 bits**

2. **The range of Well-Known ports managed by IANA is:**
   * (a) $0\text{ to }255$
   * (b) $0\text{ to }1023$
   * (c) $1024\text{ to }49,151$
   * (d) $49,152\text{ to }65,535$
   * *Answer:* **(b) $0\text{ to }1023$**

3. **Which port number is assigned to the HyperText Transfer Protocol (HTTP)?**
   * (a) 21
   * (b) 25
   * (c) 53
   * (d) 80
   * *Answer:* **(d) 80**

4. **A Socket Address is formed by combining:**
   * (a) MAC Address and IP Address
   * (b) IP Address and Port Number
   * (c) Port Number and Sequence Number
   * (d) Subnet Mask and Host ID
   * *Answer:* **(b) IP Address and Port Number**

5. **What is the minimum header size of a TCP segment?**
   * (a) $8\text{ bytes}$
   * (b) $16\text{ bytes}$
   * (c) $20\text{ bytes}$
   * (d) $60\text{ bytes}$
   * *Answer:* **(c) $20\text{ bytes}$**

6. **In the TCP header, an HLEN value of binary `1000` indicates a header length of:**
   * (a) $8\text{ bytes}$
   * (b) $20\text{ bytes}$
   * (c) $32\text{ bytes}$
   * (d) $40\text{ bytes}$
   * *Answer:* **(c) $32\text{ bytes}$** [Calculation: $8 \times 4 = 32\text{ bytes}$]

7. **During the TCP three-way handshake, the segment sent by the server has which flags set?**
   * (a) SYN only
   * (b) ACK only
   * (c) SYN and ACK
   * (d) RST and ACK
   * *Answer:* **(c) SYN and ACK**

8. **What is the fixed header size of a User Datagram Protocol (UDP) packet?**
   * (a) $4\text{ bytes}$
   * (b) $8\text{ bytes}$
   * (c) $20\text{ bytes}$
   * (d) $32\text{ bytes}$
   * *Answer:* **(b) $8\text{ bytes}$**

9. **The effective key length of the standard Data Encryption Standard (DES) algorithm is:**
   * (a) $48\text{ bits}$
   * (b) $56\text{ bits}$
   * (c) $64\text{ bits}$
   * (d) $128\text{ bits}$
   * *Answer:* **(b) $56\text{ bits}$**

10. **In asymmetric-key cryptography, a message encrypted with a recipient's public key can be decrypted ONLY by the:**
    * (a) Sender's Public Key
    * (b) Sender's Private Key
    * (c) Recipient's Private Key
    * (d) Central Certificate Authority Key
    * *Answer:* **(c) Recipient's Private Key**