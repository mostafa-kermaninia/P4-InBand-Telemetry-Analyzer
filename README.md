# P4 Real-Time Network Telemetry Engine

![P4](https://img.shields.io/badge/Language-P4_16-blue)
![Python](https://img.shields.io/badge/Control_Plane-Python-yellow)
![Platform](https://img.shields.io/badge/Platform-Mininet-green)

## Overview
**P4-Realtime-Telemetry-Engine** is an advanced implementation of **In-Band Network Telemetry (INT)** designed to provide granular visibility into network performance. Unlike traditional monitoring systems (like SNMP) that poll devices, this engine injects telemetry metadata directly into data packets as they traverse the switching fabric.

This project demonstrates the power of **Programmable Data Planes** by utilizing P4 registers to track stateful metrics such as **Queue Depth** and **Hop-by-Hop Latency** in real-time.



[Image of Architecture Diagram]
 
*(You can upload a screenshot of your topology or output here in the docs folder)*

## Key Features

###  1. In-Band Telemetry (INT)
- **Header Injection:** Custom logic to parse, modify, and deparse packets to insert switch IDs and port information.
- **Hop-by-Hop Visibility:** Traces the exact path of packets through the network topology.

###  2. Stateful Analytics (P4 Registers)
- **Queue Depth Monitoring:** Uses `standard_metadata.enq_qdepth` to measure congestion at each hop.
- **Aggregated Statistics:** Maintains a running history of packet counts and queue states directly on the switch hardware (simulated) without control plane intervention.
- **Micro-burst Detection:** Capable of detecting transient congestion events that average-based tools miss.

###  3. Python Control Plane
- **Custom Traffic Generation:** Scapy-based scripts to generate TCP/UDP loads.
- **Telemetry Collector:** Decapsulates INT headers at the destination host and calculates the **Average Path Queue Depth**:
  $$Avg\_Queue = \frac{\sum Queue\_Depths}{Hop\_Count}$$

## Project Structure

The repository is divided into two modules:

| Module | Description |
| :--- | :--- |
| **`basic-int`** | Fundamental implementation of INT, inserting Switch ID and Output Port into packets. |
| **`aggregated-stats`** | Advanced implementation using **P4 Registers** to track cumulative queue depths and packet counts for statistical analysis. |

## Getting Started

### Prerequisites
To run this simulation, you need a P4-enabled environment:
- **P4 VM** (Recommended) or:
- **Mininet**
- **BMv2** (Behavioral Model Switch)
- **P4 Compiler (p4c)**

### Installation
1.  Clone the repository:
    ```bash
    git clone [https://github.com/Mostafa-Kermaninia/P4-Realtime-Telemetry-Engine.git](https://github.com/Mostafa-Kermaninia/P4-Realtime-Telemetry-Engine.git)
    cd P4-Realtime-Telemetry-Engine
    ```

### Usage Guide

#### 1. Running the Simulation
Navigate to the desired module (e.g., `src/aggregated-stats`) and start the Mininet topology:

```bash
cd src/aggregated-stats/topology
sudo p4run
```
2. Starting the Collector (Receiver)
On the destination host (e.g., h2), run the telemetry receiver. This script listens for packets and parses the INT headers.
```Bash
mininet> h2 python3 ../control-plane/receive.py
```
3. Generating Traffic
On the source host (e.g., h1), inject traffic into the network. You can specify the destination IP, message, and packet count.
```Bash
# Syntax: python3 send.py <Target_IP> <Message> <Count>
mininet> h1 python3 ../control-plane/send.py 10.0.2.2 "Telemetry Test" 10
```
4. Analyzing Output
The receiver will output real-time metrics extracted from the packet headers:
```
--- INT Packet Received ---
[Switch 1] Port: 2 | Queue Depth: 12
[Switch 2] Port: 1 | Queue Depth: 45 (Congested)
Path Average Queue Depth: 28.5
```
### Protocol Implementation Details
#### UDP/TCP Fairness Analysis
The project includes scenarios to compare UDP and TCP traffic behavior under congestion. The P4 data plane allows us to observe how UDP traffic (uncontrolled) can starve TCP flows by filling up switch queues, visualized through the reported Queue Depth metrics.

### License
Distributed under the MIT License.


---
