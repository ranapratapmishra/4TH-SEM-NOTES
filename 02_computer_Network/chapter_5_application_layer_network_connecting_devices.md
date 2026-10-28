# Chapter 5: Application Layer & Network Connecting Devices

---

## 1. Application Layer Fundamentals & Services

### 1.1 Overview & Scope
The Application Layer is the top-most layer (Layer 7 in the OSI model, Layer 5 in the TCP/IP stack) that directly interfaces with the end-user (human or software). It provides an interface to access network resources and provides services to user applications.

```text
       Host A (Alice)                                    Host B (Bob)
+-----------------------------+                  +-----------------------------+
|      Application Layer      | <==[ Logical End-to-End Channel ]============> |      Application Layer      |
+--------------+--------------+                  +--------------^--------------+
               |                                                |
               v                                                |
+--------------+--------------+                  +--------------+--------------+
|       Transport Layer       |                  |       Transport Layer       |
+--------------+--------------+                  +--------------+--------------+
               |                                                ^
               v                                                |
+--------------+--------------+                  +--------------+--------------+
|        Network Layer        |                  |        Network Layer        |
+--------------+--------------+                  +--------------+--------------+
               |                                                ^
               v                                                |
+--------------+--------------+                  +--------------+--------------+
|    Physical & Data Link     | ===[ Switched WANs, Routers, Physical Path ]==>|    Physical & Data Link     |
+-----------------------------+                  +-----------------------------+
```

#### Core Services:
1. **Electronic Mail (E-mail):** Distributed message transfer (SMTP, POP3, IMAP4).
2. **File Access & Transfer:** Uploading/downloading files between heterogeneous machines (FTP, TFTP).
3. **World Wide Web (WWW):** Accessing linked documents via browsers (HTTP, HTTPS).
4. **Remote Access:** Controlling remote command-line shells over IP (Telnet, SSH).
5. **Network Management:** Real-time health monitoring of network devices (SNMP).

* **Concept of Logical Connection:** Two application layers communicate over a logical connection. Even though data travels down through intermediate switches, routers, and WAN links, both applications behave as if directly connected.

---

## 2. Electronic Mail (E-Mail) Architecture

### 2.1 Evolution of E-Mail
Originally designed for simple 7-bit text memos between early terminal users, email evolved into a multimedia messaging platform supporting:
* **Multimedia Attachments:** Audio, video, images, PDFs.
* **Multiple Recipients:** Distribution via Carbon Copy (CC) and Blind Carbon Copy (BCC).
* **Hyperlinks & Rich Formatting:** Styled text via HTML/CSS.
* **Security & Encryption:** End-to-end transport protection via SSL/TLS.
* **Organizational Tools:** Server-side folders, automatic filters, and calendar integration.

### 2.2 The Three Architectural Agents (UA, MTA, MAA)
An e-mail system relies on three separate functional entities:

```text
 Alice (Sender)                                                                         Bob (Receiver)
   |                                                                                          ^
   | (1. Compose)                                                                             | (9. Read)
   v                                                                                          |
+------+                                                                                   +------+
|  UA  |                                                                                   |  UA  |
+------+                                                                                   +------+
   |                                                                                          ^
   | (2. Step 1: SMTP)                                                                        | (8. Step 3: POP3/IMAP4)
   v                                                                                          |
+------------+       +-------------------+                         +-------------------+   +------------+
| MTA Client | ----> | MTA Server (Spool)|                         | MAA Server (Boxes)|-> | MAA Client |
+------------+       +---------+---------+                         +---------^---------+   +------------+
[Alice Device]                 | (3 & 4)                                     | (6 & 7)     [Bob Device]
                               v                                             |
                     +-------------------+                         +-------------------+
                     | MTA Client (Gmail)| === (5. Step 2: SMTP) =>| MTA Server (Yahoo)|
                     +-------------------+       (Internet)        +-------------------+
                     [Sender Mail Server]                          [Receiver Mail Server]
```

1. **User Agent (UA):** The client program used by a human to compose, read, and manage mail (e.g., Outlook, Apple Mail).
2. **Message Transfer Agent (MTA):** Relays mail from host to host. Standard protocol: **SMTP (Simple Mail Transfer Protocol)**.
3. **Message Access Agent (MAA):** Pulls stored mail from a remote server mailbox to the recipient host. Standard protocols: **POP3** and **IMAP4**.

---

## 3. E-Mail Protocols: SMTP, POP3, IMAP4 & MIME

### 3.1 SMTP (Simple Mail Transfer Protocol)
* **Transport:** TCP Port 25.
* **Push Protocol:** Delivery is initiated by the machine holding the message (Sender $\rightarrow$ Mail Server).
* **The "Used Two Times" Rule:**
  1. *Stage 1:* Sender client $\rightarrow$ Sender's Mail Server (SMTP).
  2. *Stage 2:* Sender's Mail Server $\rightarrow$ Recipient's Mail Server across Internet (SMTP).
  3. *Stage 3:* Recipient's Mail Server $\rightarrow$ Recipient Host (**CANNOT use SMTP**).

```text
Why SMTP Fails at Stage 3:
Recipient machines are not permanently powered on with fixed public IPs. If the mail server 
attempts to PUSH mail using SMTP to an offline machine, delivery fails. Therefore, incoming 
mail is held in the server's Mailbox until the recipient actively PULLS it via POP3 or IMAP4.
```

### 3.2 POP3 vs. IMAP4 Comparison

```text
                             POP3 Operational Modes
         +-----------------------------+-----------------------------+
         | 1. Delete Mode:             | 2. Keep Mode:               |
         | Mail deleted from server    | Mail remains saved on server|
         | immediately after download. | after client download.      |
         +-----------------------------+-----------------------------+
```

| Feature | POP3 (Post Office Protocol v3) | IMAP4 (Internet Mail Access Protocol v4) |
| :--- | :--- | :--- |
| **Port** | **TCP Port 110** | **TCP Port 143** |
| **Philosophy** | Download & store locally (Offline model) | Access & synchronize directly on server |
| **Folder Organization**| Only locally on client machine | Hierarchical folders created directly on server |
| **Partial Download** | Not supported (downloads full email) | **Supported** (can download headers or text only) |
| **Search on Server** | Not supported (must download first) | **Supported** (searches strings on remote server) |
| **Multi-Device Sync** | Poor (creates sync discrepancies) | **Seamless** (all phones, PCs, web view identical state) |

### 3.3 MIME (Multipurpose Internet Mail Extensions)
* Traditional SMTP is restricted to **7-bit NVT ASCII** text. It cannot handle 8-bit binary data (images, videos, executables) or non-English alphabets (Hindi, French, Japanese, Russian).
* **MIME Solution:** A supplementary layer that encodes arbitrary non-ASCII binary data into 7-bit NVT ASCII at the sender (e.g., via Base64 encoding). At the receiver, MIME decodes the ASCII stream back into its original binary structure.

### 3.4 Web-Based E-Mail (Gmail, Yahoo, Hotmail)
1. *Browser to Server (Sender):* Handled via **HTTP/HTTPS**.
2. *Server to Server:* Handled via **SMTP** across the internet backbone.
3. *Server to Browser (Recipient):* Handled via **HTTP/HTTPS** (rendering email in HTML format instead of using POP3/IMAP4).

---

## 4. File Transfer: FTP & TFTP

### 4.1 FTP (File Transfer Protocol) Architecture
FTP addresses computer heterogeneity (differing OS file systems, directory structures, case sensitivity, newline standards: `\r\n` vs. `\n`) by maintaining **two separate parallel TCP connections**:

```text
       CLIENT SITE                                                   SERVER SITE
+------------------------+                                   +------------------------+
|     User Interface     |                                   |                        |
+-----------+------------+                                   |                        |
            |                                                |                        |
            v                                                v                        |
+------------------------+      Control Connection (Port 21) +------------------------+
|    Control Process     | <===============================> |    Control Process     |
+------------------------+    (Commands & Responses ONLY)    +------------------------+
            |                  Remains OPEN full session                 |
            |                                                            |
            v                                                            v
+------------------------+       Data Connection (Port 20)   +------------------------+
|  Data Transfer Process | <===============================> |  Data Transfer Process |
+-----------+------------+     (Actual File Transfer ONLY)   +-----------+------------+
            |                  Opens & Closes PER file                   |
            v                                                            v
   [ Local File System ]                                       [ Remote File System ]
```

* **Control Connection (TCP Port 21):** Carries plain-text commands (`USER`, `PASS`, `RETR`, `STOR`) and 3-digit status codes. Stays open throughout the entire session.
* **Data Connection (TCP Port 20):** Dedicated solely to raw file transfer. Opened dynamically when a transfer command is issued and closed immediately upon file completion.
* **Security (FTPS):** Classical FTP sends credentials in cleartext. SSL/TLS is added between FTP and TCP (SSL-FTP / FTPS) to encrypt control and data streams.

### 4.2 TFTP (Trivial File Transfer Protocol)
* **Transport:** Runs over **UDP Port 69**.
* **Features:** Stripped-down protocol with no authentication (no passwords), no directory browsing, and minimal error handling.
* **Use Case:** Loading bootstrap firmware and configuration files into diskless workstations (PXE Boot) and backing up router configurations.

---

## 5. World Wide Web (WWW) & Hypertext Transfer Protocol (HTTP)

### 5.1 History & Uniform Resource Locators (URL)
* Proposed by **Tim Berners-Lee in 1989** at CERN to link scientific documents globally via distributed hypertext.
* **URL Syntax:** `protocol://host:port/path`
  * *Protocol:* Method (`http`, `https`, `ftp`).
  * *Host:* Domain name or IP address of the target server.
  * *Port:* Optional 16-bit port (default: 80 for HTTP, 443 for HTTPS).
  * *Path:* File path of the resource on the host machine.

### 5.2 Browser & Server Architecture
* **Browser:** Divided into **Controller** (interprets inputs), **Client Protocols** (HTTP, FTP), and **Interpreters** (HTML rendering engine, CSS, JavaScript engine).
* **Server Optimization:** Employs **In-Memory Caching** (storing popular pages in RAM) and **Multithreading / Multiprocessing** to serve concurrent client requests.

### 5.3 Non-Persistent vs. Persistent HTTP

```text
Non-Persistent HTTP (HTTP/1.0):
Connection closed after EVERY response.
For base HTML + N inline images: Requires N + 1 distinct TCP connections.
Total Handshake Latency = 2 RTT * (N + 1).

Persistent HTTP (HTTP/1.1):
Connection remains OPEN for multiple requests.
For base HTML + N inline images: Requires ONLY 1 TCP connection.
Saves socket buffers and eliminates repeated 3-way handshakes.
```

### 5.4 Cookies: Adding State to Stateless HTTP
* **Statelessness:** By design, an HTTP server maintains no memory of client interactions. Once a response is sent, the transaction is closed.
* **Mechanism of Cookies:**
  1. *Creation:* On first visit/login, the server issues a `Set-Cookie: user_id=XYZ` header.
  2. *Storage:* The client browser saves this cookie string locally.
  3. *Subsequent Requests:* The browser appends `Cookie: user_id=XYZ` to all outgoing requests to that site.
  4. *Application:* Enables shopping carts, session persistence, and personalization.

---

## 6. Domain Name System (DNS)

### 6.1 Purpose & Name Space
Humans prefer descriptive names (`google.com`), whereas network routers require numerical IP addresses. DNS provides distributed name-to-address resolution over **UDP Port 53**.
* **Flat Name Space:** Unstructured names; unmanageable on global scale due to collisions.
* **Hierarchical Name Space:** Structured as an **Inverted Tree** with up to 128 levels (Level 0 to Level 127).

```text
                                        Level 0: [ . ] (Root)
                                                   |
           +---------------------------------------+---------------------------------------+
           | (Generic Domains)                                                             | (Country Domains)
           v                                                                               v
  +---+---+---+---+---+---+---+                                                   +---+---+---+---+
  |com|edu|gov|mil|net|org|biz|...                                                | in| us| uk| jp|...
  +---+---+---+---+---+---+---+                                                   +---+---+---+---+
    |   |                                                                           |
    |   +-> yale -> eng                                                             +-> ac -> dtu -> cse
    +-----> sun  -> eng -> ai -> robot
```

* **Node Label:** Max 63 characters per string. Root label is null (`""`).
* **Fully Qualified Domain Name (FQDN):** Read bottom-up from leaf to root, ending in a dot: `challenger.atc.fhda.edu.`

### 6.2 DNS Server Hierarchy & Resolution Flow

```text
[ Client Host ] 
      |  (1. Query: "IP of www.yourcompany.com?")
      v
[ ISP Local DNS Server / Resolver ]
      |
      |--- (3. Query to Root) -----------------------------------> [ Root DNS Server ]
      |<-- (4. Referral: "Contact .COM TLD server at IP_TLD") ---- |
      |
      |--- (5. Query to .COM TLD) --------------------------------> [ Top Level DNS Server for .COM ]
      |<-- (6. Referral: "Contact yourcompany.com Authoritative")- |
      |
      |--- (7. Query to Authoritative Server) --------------------> [ yourcompany.com DNS Server ]
      |<-- (8. Final Answer: "IP is 198.61.190.243") ------------- |
      |
      v  (9. Final IP returned to Client Host)
[ Client Host ] === (10. Establishes HTTP Connection to 198.61.190.243) ===>
```

1. **Root Name Server:** Directs queries to the appropriate TLD server.
2. **Top-Level Domain (TLD) Server:** Manages generic (`.com`, `.org`) or country-code (`.in`) zones.
3. **Authoritative Name Server:** Holds definitive, registered mapping records for an organization's hosts.

---

## 7. Remote Management & Access Protocols: TELNET & SNMP

### 7.1 TELNET (Telecommunication Network)
* Operates over **TCP Port 23**.
* Employs **Network Virtual Terminal (NVT)** character encoding to translate client keystrokes for remote operating systems via a pseudoterminal driver.
* **Security Flaw:** Transmits all data (including user passwords) in unencrypted plaintext. Largely superseded by **SSH (Secure Shell - TCP Port 22)**.

### 7.2 SNMP (Simple Network Management Protocol)
* Operates over **UDP Port 161 (Polling)** and **UDP Port 162 (Traps)**.
* Components:
  1. *SNMP Manager:* Central monitoring console.
  2. *SNMP Agent:* Daemon process running on network nodes (routers, switches).
  3. *MIB (Management Information Base):* Database of device parameters (packet drops, CPU load).
* **Operations:**
  * **GET:** Manager polls agent for device metrics.
  * **SET:** Manager modifies configuration variables on the device.
  * **TRAP:** Agent spontaneously sends an unsolicited emergency notification to the manager when a fault occurs (e.g., link down).

---

## 8. Remote Procedure Call (RPC)

### 8.1 Concept & Purpose
Remote Procedure Call (RPC) allows a program running on a local computer to execute subroutines or functions on a remote server across the network **as if they were local function calls**. It abstracts networking complexities (sockets, byte order) for distributed computing.

```text
       CLIENT MACHINE                                                 SERVER MACHINE
+-------------------------------+                             +-------------------------------+
| Client Application Process    |                             | Server Execution Service      |
| [1. Calls add(5, 10)]         |                             | [4. Executes add() -> returns]|
+---------------+---------------+                             +---------------+---------------+
                |                                                             ^
                v Local Call                                                  | Function Invocation
+---------------+---------------+                             +---------------+---------------+
| Client Stub                   |                             | Server Stub                   |
| [2. Packs parameters into msg]|                             | [Unpacks parameters from msg] |
| (Marshalling)                 |                             | (Unmarshalling)               |
+---------------+---------------+                             +---------------+---------------+
                |                                                             ^
                v Binary Buffer                                               | Binary Buffer
+---------------+---------------+                             +---------------+---------------+
| RPC Runtime                   |                             | RPC Runtime                   |
| [3. Transmits Call Packet]    | ===[ Network (TCP/HTTP) ]==>| [Receives Call Packet]        |
+---------------+---------------+                             +---------------+---------------+
                |                                                             |
                v (Client blocks & waits)                                     v
+---------------+---------------+                             +---------------+---------------+
| RPC Runtime                   |                             | RPC Runtime                   |
| [6. Receives Result Packet]   | <==[ Network (TCP/HTTP) ]===| [5. Transmits Result Packet]  |
+---------------+---------------+                             +---------------+---------------+
                |                                                             |
                v Binary Buffer                                               v Binary Buffer
+---------------+---------------+                             +---------------+---------------+
| Client Stub                   |                             | Server Stub                   |
| [Unpacks result value]        |                             | [Packs return value]          |
| (Unmarshalling)               |                             | (Marshalling)                 |
+---------------+---------------+                             +-------------------------------+
                |
                v Local Return
+---------------+---------------+
| Client Application Process    |
| [Receives result: 15]         |
+-------------------------------+
```

### 8.2 The 6 Steps of RPC Execution:
1. **Invocation:** The client program executes a procedure call with standard parameters.
2. **Client Marshalling (Client Stub):** The client stub captures parameters, converts them into a standardized machine-independent format (handling endianness), and serializes them into a message buffer.
3. **Transmission:** The client RPC runtime transmits the request packet over the network (via TCP, UDP, or HTTP) and the client process blocks (waits).
4. **Server Unmarshalling (Server Stub):** The server RPC runtime passes the packet to the server stub, which deserializes the parameters and calls the actual local function on the server.
5. **Execution & Server Marshalling:** The server executes the function, and the server stub packs the return values into a result message.
6. **Return & Unmarshalling:** The result packet travels back to the client; the client stub unpacks the return values, unblocks the calling program, and hands over execution.

---

## 9. Network Connecting Devices

```text
                          OSI Layer Device Hierarchy
+-------------------------------------------------------------------------------+
| Layer 7: Gateway (Application / Transport Layer Protocol Converter)           |
+-------------------------------------------------------------------------------+
| Layer 3: Router (IP Address Routing, Multiple Subnets, Collision + Broadcast) |
+-------------------------------------------------------------------------------+
| Layer 2: Switch / Bridge (MAC Address Filtering, Unicast, Separate Collision) |
+-------------------------------------------------------------------------------+
| Layer 1: Hub / Repeater (Physical Signal Regeneration, Single Shared Domain)  |
+-------------------------------------------------------------------------------+
```

### 9.1 Hub (Multiport Repeater)
* **OSI Layer:** Physical Layer (Layer 1).
* **Operation:** Receives an electrical signal on one port, amplifies it, and regenerates (floods) it out to **all other ports**, regardless of intended recipient.
* **Domain Behavior:** All connected devices share a **single collision domain** and a **single broadcast domain**.
* **Status:** Obsolete; replaced by switches.

### 9.2 Bridge
* **OSI Layer:** Data Link Layer (Layer 2).
* **Operation:** Connects two separate LAN segments to create a single aggregate network. Inspects source and destination MAC addresses to forward or filter frames.
* **Domain Behavior:** Splits a collision domain into two separate collision domains, but maintains a single shared broadcast domain.

### 9.3 Switch (Multiport Bridge)
* **OSI Layer:** Data Link Layer (Layer 2). First commercial Ethernet switch introduced by Kalpana in 1990.
* **Operation:** Uses an internal **MAC Address Table (CAM Table)** to direct traffic exclusively to the target destination port rather than broadcasting.
* **Domain Behavior:** **Every single port provides a dedicated collision domain**, eliminating collisions in full-duplex mode. All ports share a single broadcast domain.

### 9.4 Router
* **OSI Layer:** Network Layer (Layer 3).
* **Operation:** Reads packet IP headers and uses dynamic routing tables to route datagrams between logically different networks/subnets.
* **Domain Behavior:** **Breaks both collision domains and broadcast domains**. Broadcast packets are not forwarded past a router interface.

### 9.5 Gateway
* **OSI Layer:** Can operate at any layer up to Layer 7 (Application Layer).
* **Operation:** Acts as a protocol translator enabling communication between two discrete networks with completely incompatible protocol suites (e.g., bridging an enterprise IP network to a legacy SNA or cellular telecommunication network).

---

## 10. Master Comparison of Network Devices

| Device | Primary OSI Layer | Addressing Used | Forwards Traffic To | Breaks Collision Domain? | Breaks Broadcast Domain? |
| :--- | :---: | :---: | :--- | :---: | :---: |
| **Hub** | Layer 1 (Physical) | None (Electrical bits) | Floods to all ports | **No** (1 Shared) | **No** (1 Shared) |
| **Bridge** | Layer 2 (Data Link) | MAC Address | Filtered / Forwarded | **Yes** (2 Domains) | **No** (1 Shared) |
| **Switch** | Layer 2 (Data Link) | MAC Address | Specific destination port | **Yes** (Per-port) | **No** (1 Shared) |
| **Router** | Layer 3 (Network) | IP Address | Next-hop IP interface | **Yes** (Per-port) | **Yes** (Per-port) |
| **Gateway**| Layer 4 to Layer 7 | Protocol translations | Incompatible networks | **Yes** | **Yes** |

---

## 11. Important BEU Examination Questions & Answers

### Q1. Explain the architecture of File Transfer Protocol (FTP). Why does FTP use two separate connections? (14 Marks)
**Model Steps:**
1. **Introduction:** Define FTP as a client-server protocol enabling file transfer across heterogeneous file systems.
2. **Two Connections:**
   * **Control Connection (TCP Port 21):** Persistent connection for commands (`USER`, `PASS`, `LIST`) and status codes.
   * **Data Connection (TCP Port 20):** Transient connection created on-demand for transferring raw file bytes; closed immediately when transfer finishes.
3. **Diagram:** Draw the block diagram showing Control Process, Data Transfer Process, User Interface, and File Systems.
4. **Justification:** Separation of concerns keeps command control responsive, allows transfer monitoring, and avoids data stream corruption.

---

### Q2. What is Remote Procedure Call (RPC)? Explain the role of Stubs and Marshalling with a neat sequence diagram. (14 Marks)
**Model Steps:**
1. **Definition:** RPC enables a local program to invoke procedures on a remote server transparently as if executed locally.
2. **Components:**
   * *Client Stub:* Proxies the call, serializes parameters into network format (Marshalling).
   * *Server Stub:* Receives packet, unpacks parameters (Unmarshalling), and invokes local routine.
   * *RPC Runtime:* Manages network transmission (handling retransmissions, timeouts).
3. **Sequence of Execution:** Detail the 6 steps (Invocation, Marshalling, Network Call, Execution, Result Marshalling, Unmarshalling return).

---

### Q3. Differentiate between a Hub, a Switch, and a Router based on collision domains, broadcast domains, and OSI layers. (7 Marks)
**Model Answer:**
Provide the comparison table highlighting:
* *Hub:* Layer 1, no addressing, single collision/broadcast domain.
* *Switch:* Layer 2, MAC filtering, per-port collision domain, single broadcast domain.
* *Router:* Layer 3, IP routing, per-port collision domain, breaks broadcast domains.

---

### Q4. Explain the DNS resolution process when a client attempts to access `www.yourcompany.com`. (7 Marks)
**Model Steps:**
1. State client query to local ISP resolver.
2. Resolver queries Root DNS server; receives referral to `.COM` TLD server.
3. Resolver queries `.COM` TLD server; receives referral to Authoritative server for `yourcompany.com`.
4. Resolver queries Authoritative server; receives the definitive IP address (`198.61.190.243`).
5. Resolver caches entry and returns IP to browser. Browser initiates HTTP to destination IP.

---

## 12. Objective Questions (MCQ Bank)

1. **Which protocol uses two separate TCP connections simultaneously for its operation?**
   * (a) HTTP
   * (b) FTP (Port 20 and Port 21)
   * (c) SMTP
   * (d) TELNET
   * *Answer:* **(b) FTP (Port 20 and Port 21)**

2. **In e-mail delivery, which protocol is strictly classified as a Pull Protocol?**
   * (a) SMTP
   * (b) POP3
   * (c) MIME
   * (d) TFTP
   * *Answer:* **(b) POP3**

3. **In the DNS hierarchy, how many total tree levels are possible?**
   * (a) 16
   * (b) 64
   * (c) 128 (Level 0 to Level 127)
   * (d) 256
   * *Answer:* **(c) 128 (Level 0 to Level 127)**

4. **In RPC, the process of packing and serializing procedure arguments into a machine-independent network message is called:**
   * (a) Fragmentation
   * (b) Marshalling
   * (c) Decapsulation
   * (d) Polling
   * *Answer:* **(b) Marshalling**

5. **Which network connecting device operates at Layer 2 and creates a separate collision domain for every connected port?**
   * (a) Hub
   * (b) Switch
   * (c) Repeater
   * (d) Modem
   * *Answer:* **(b) Switch**

6. **Which network connecting device terminates and breaks broadcast domains?**
   * (a) Hub
   * (b) Bridge
   * (c) Switch
   * (d) Router
   * *Answer:* **(d) Router**

7. **The maximum allowable character length for an individual DNS label is:**
   * (a) 32 characters
   * (b) 63 characters
   * (c) 128 characters
   * (d) 255 characters
   * *Answer:* **(b) 63 characters**

8. **What transport layer protocol and port number are used by TFTP?**
   * (a) TCP port 21
   * (b) TCP port 20
   * (c) UDP port 69
   * (d) UDP port 161
   * *Answer:* **(c) UDP port 69**

9. **In SNMP, an unsolicited emergency alert sent by an agent to the manager upon fault occurrence is called:**
   * (a) GET
   * (b) SET
   * (c) TRAP
   * (d) PING
   * *Answer:* **(c) TRAP**

10. **A device designed to translate and interconnect completely incompatible protocol suites across any of the seven OSI layers is a:**
    * (a) Switch
    * (b) Gateway
    * (c) Bridge
    * (d) Hub
    * *Answer:* **(b) Gateway**