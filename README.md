# DPI Engine - Deep Packet Inspection System

A C++ based Network Packet Analyzer and Deep Packet Inspection (DPI) system designed to analyze captured network traffic, identify applications and domains, track connections, apply filtering rules, and generate processed PCAP files.

---

## 📌 Overview

This project implements a Deep Packet Inspection engine that processes network packets from PCAP files.

The system performs:

- Packet parsing
- TCP/UDP identification
- IP and port extraction
- Five-tuple flow identification
- Connection tracking
- TLS SNI extraction
- Application classification
- Traffic filtering
- Multithreaded packet processing
- Load balancing
- Forward/drop decisions
- Output PCAP generation

### Basic Flow

```text
PCAP File
    ↓
PCAP Reader
    ↓
Packet Parser
    ↓
Load Balancer
    ↓
Fast Path Workers
    ↓
DPI / SNI Extraction
    ↓
Application Classification
    ↓
Rule Manager
    ↓
Forward / Drop
    ↓
Output PCAP
```

---

## 🎯 What Does This Project Do?

The DPI engine takes a captured network traffic file as input and analyzes the packets inside it.

It extracts information such as:

- Source IP
- Destination IP
- Source port
- Destination port
- Protocol
- Packet size
- TCP/UDP information
- TLS SNI/domain information

The extracted information is then used to classify traffic and apply configured rules.

The processed packets are finally written into an output PCAP file.

---

## ✨ Key Features

### 1. Deep Packet Inspection

The engine analyzes packet headers and available payload information to understand network traffic.

### 2. Packet Parsing

The system parses:

- Ethernet headers
- IPv4 headers
- TCP headers
- UDP headers
- Packet payloads

### 3. TLS SNI Extraction

The system can inspect TLS Client Hello packets and extract the Server Name Indication (SNI).

Example:

```text
TLS Client Hello
       ↓
SNI: www.youtube.com
       ↓
Application: YouTube
```

### 4. Application Classification

The extracted domain/SNI information can be mapped to applications such as:

- YouTube
- Facebook
- Instagram
- Twitter/X
- Spotify
- Telegram
- Discord
- Amazon
- Google
- GitHub
- TikTok
- Zoom

### 5. Connection Tracking

Packets are associated with network flows using a five-tuple:

```text
Source IP
Destination IP
Source Port
Destination Port
Protocol
```

This allows packets belonging to the same connection to be tracked.

### 6. Traffic Filtering

The rule manager can apply filtering rules based on:

- IP address
- Application
- Domain

Packets can then be:

```text
FORWARD
   or
DROP
```

### 7. Multithreaded Processing

The project uses multiple processing threads to process packets concurrently.

```text
                 Load Balancer
                /             \
               ↓               ↓
        Fast Path Worker  Fast Path Worker
               \               /
                ↓             ↓
                 Packet Processing
```

### 8. Load Balancing

Packets are distributed among processing workers using load-balancing logic.

Flow affinity is maintained so packets belonging to the same flow can be processed consistently.

### 9. PCAP Output

After processing, forwarded packets are written to an output PCAP file.

---

# 🏗️ System Architecture

```text
                    PCAP FILE
                        │
                        ▼
                ┌───────────────┐
                │  PCAP Reader  │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │ Packet Parser │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │ Load Balancer │
                └───────┬───────┘
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
      ┌──────────────┐      ┌──────────────┐
      │ Fast Path    │      │ Fast Path    │
      │ Worker       │      │ Worker       │
      └──────┬───────┘      └──────┬───────┘
             │                     │
             └──────────┬──────────┘
                        ▼
                ┌───────────────┐
                │ DPI / SNI     │
                │ Classification│
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │ Rule Manager  │
                └───────┬───────┘
                        │
                 ┌──────┴──────┐
                 ▼             ▼
              FORWARD         DROP
                 │
                 ▼
             Output PCAP
```

---

# 🔄 How the System Works

## 1. PCAP Reader

The PCAP reader opens the captured network traffic file and reads packets sequentially.

```text
test_dpi.pcap
      ↓
PCAP Reader
```

## 2. Packet Parser

Each packet is parsed to extract important network information.

The parser identifies:

```text
Ethernet
   ↓
IPv4
   ↓
TCP / UDP
   ↓
Payload
```

## 3. Five-Tuple Extraction

For each network flow, the system uses:

```text
Source IP
Destination IP
Source Port
Destination Port
Protocol
```

## 4. Connection Tracking

The connection tracker keeps information about network flows.

This allows multiple packets belonging to the same connection to be associated with the same flow.

## 5. Load Balancing

The load balancer distributes packets between processing workers.

```text
             Packets
                │
                ▼
         Load Balancer
          /          \
         ↓            ↓
       Worker 1     Worker 2
```

## 6. Fast Path Processing

Fast Path workers perform the main packet-processing operations.

These include:

- Packet classification
- Flow lookup
- Connection tracking
- Application detection
- Rule checking

---

# 🔐 TLS SNI Extraction

## What is SNI?

SNI stands for:

**Server Name Indication**

It is a hostname provided during the TLS Client Hello.

For example:

```text
Client
  │
  │ TLS Client Hello
  │
  ├── SNI: www.youtube.com
  │
  ▼
Server
```

The DPI engine extracts the SNI when it is available and uses it for application/domain classification.

## SNI Processing

```text
Packet
  ↓
TCP Port 443
  ↓
TLS Client Hello
  ↓
Extract SNI
  ↓
www.youtube.com
  ↓
Application Classification
  ↓
YouTube
```

## Example SNI Detection

```text
www.youtube.com      → YouTube
github.com           → GitHub
open.spotify.com     → Spotify
www.instagram.com    → Instagram
www.amazon.com       → Amazon
discord.com          → Discord
web.telegram.org     → Telegram
```

---

# 🌐 Application Detection

The project contains application/domain mapping logic.

Examples include:

```text
YouTube
Facebook
Instagram
Twitter/X
Spotify
Telegram
Discord
Amazon
Google
GitHub
TikTok
Zoom
Cloudflare
Apple
```

Application classification is based on the implemented detection and mapping rules.

---

# 🚫 Traffic Filtering

The Rule Manager is responsible for applying filtering rules.

Rules can be based on:

### IP Address

```text
Block specific IP
```

### Application

```text
Block YouTube
Block Instagram
```

### Domain

```text
Block example.com
```

The result of rule processing is:

```text
Packet
  ↓
Rule Check
  ↓
 ┌───────────────┐
 │               │
 ▼               ▼
FORWARD         DROP
```

---

# 🧵 Multithreaded Architecture

The DPI engine uses multiple threads for packet processing.

The main components are:

```text
PCAP Reader
     ↓
Load Balancers
     ↓
Fast Path Workers
     ↓
Output Processing
```

Thread-safe queues are used to safely transfer packets between processing stages.

---

# ⚡ Load Balancing

The system contains load balancers that distribute packets to Fast Path workers.

```text
                  Packets
                     │
                     ▼
              ┌─────────────┐
              │Load Balancer│
              └──────┬──────┘
                     │
             ┌───────┴───────┐
             ▼               ▼
          Fast Path       Fast Path
          Worker 1        Worker 2
```

Flow-based hashing helps keep packets from the same flow associated with the same processing path.

---

# 📂 Project Structure

```text
Packet_analyzer/
│
├── include/
│   ├── dpi_engine.h
│   ├── packet_parser.h
│   ├── pcap_reader.h
│   ├── sni_extractor.h
│   ├── connection_tracker.h
│   ├── load_balancer.h
│   ├── fast_path.h
│   ├── rule_manager.h
│   └── types.h
│
├── src/
│   ├── dpi_mt.cpp
│   ├── dpi_engine.cpp
│   ├── packet_parser.cpp
│   ├── pcap_reader.cpp
│   ├── sni_extractor.cpp
│   ├── connection_tracker.cpp
│   ├── load_balancer.cpp
│   ├── fast_path.cpp
│   ├── rule_manager.cpp
│   └── types.cpp
│
├── CMakeLists.txt
├── README.md
├── WINDOWS_SETUP.md
├── generate_test_pcap.py
└── .gitignore
```

---

# 🔧 Technologies Used

```text
C++
C++17
Computer Networks
TCP/IP
UDP
TLS
PCAP
Multithreading
CMake
Git
GitHub
```

---

# 💻 Requirements

- Windows
- MSYS2 UCRT64
- GCC 16+
- C++17
- Python
- Git
- CMake

---

# ⚙️ Installation

## Clone the Repository

```bash
git clone https://github.com/mridul2213/Packet_analyzer_inspection.git
cd Packet_analyzer_inspection
```

---

# 🔨 Build

Use the MSYS2 UCRT64 terminal.

```bash
g++ -std=c++17 -pthread -O2 -I include -o dpi_engine.exe src/dpi_mt.cpp src/dpi_engine.cpp src/pcap_reader.cpp src/packet_parser.cpp src/sni_extractor.cpp src/types.cpp src/load_balancer.cpp src/fast_path.cpp src/connection_tracker.cpp src/rule_manager.cpp
```

If the build is successful, the executable will be:

```text
dpi_engine.exe
```

---

# ▶️ Run

```powershell
.\dpi_engine.exe test_dpi.pcap dpi_output.pcap
```

Format:

```text
dpi_engine.exe <input.pcap> <output.pcap>
```

---

# 🧪 Testing

The project can be tested using a sample PCAP file.

```powershell
.\dpi_engine.exe test_dpi.pcap dpi_output.pcap
```

The engine processes the packets and displays:

- Total packets
- Total bytes
- TCP packets
- UDP packets
- Forwarded packets
- Dropped packets
- Application classification
- Detected domains/SNI
- Thread statistics

---

# 📊 Example Output

```text
DPI ENGINE v2.0
Multi-threaded

Total packets: 77
Total bytes: 5738

TCP: 73
UDP: 4

Forwarded: 77
Dropped: 0
```

Application detection may include:

```text
HTTPS
DNS
YouTube
Facebook
Instagram
Twitter/X
Spotify
Telegram
Discord
Amazon
Google
GitHub
TikTok
Zoom
```

Detected domains may include:

```text
www.youtube.com
www.facebook.com
github.com
www.instagram.com
open.spotify.com
web.telegram.org
discord.com
www.amazon.com
www.google.com
```

---

# 📤 Output

After processing, the engine generates:

```text
dpi_output.pcap
```

The output file can be opened using Wireshark for further packet analysis.

---

# 🧠 Core Networking Concepts

This project demonstrates practical use of:

- Packet
- IP Address
- MAC Address
- Port
- TCP
- UDP
- Five-Tuple
- Flow
- PCAP
- DPI
- TLS
- SNI

### Five-Tuple

```text
Source IP
Destination IP
Source Port
Destination Port
Protocol
```

---

# 🚧 Current Limitations

- The current system processes PCAP files rather than directly capturing live network traffic.
- SNI-based identification depends on SNI being available in the inspected TLS traffic.
- Application classification is based on the implemented signatures/mapping rules.
- Modern encrypted-traffic techniques can limit visibility into application/domain information.

---

# 🚀 Future Improvements

Possible improvements include:

- Live packet capture
- Npcap integration
- More application signatures
- More protocol detection
- Improved application classification
- Performance benchmarking
- Advanced filtering rules
- Better handling of encrypted traffic
- Additional network protocols
- Real-time traffic monitoring

---

# 📚 Learning Outcomes

This project provides practical experience with:

- Computer Networks
- C++ programming
- Network packet processing
- TCP/IP
- UDP
- TLS
- SNI
- Deep Packet Inspection
- Multithreading
- Thread-safe queues
- Connection tracking
- Traffic classification
- Network filtering
- PCAP analysis

---

# 📄 License

This project is intended for educational and development purposes.
