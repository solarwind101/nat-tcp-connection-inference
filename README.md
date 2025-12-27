# TCP Connection Inference Behind NAT

This project performs TCP connection inference and hijacking attacks behind a NAT.  
There are two steps to the attack:
1. Inferring client-side ephemeral ports in use in the NAT (port preservation)
2. Inferring NATed clients communicating with a target server

---

## Prerequisites

### System Requirements
- Linux (tested Ubuntu 24.02)
- Root access (required for raw packet injection)

### Dependencies

#### Scapy (Required)
sudo apt update
sudo apt install scapy

---

## 1. Inferring Client Ports in Use

### Configuration

Edit port-infer-main.py and fill in the configuration section:

# ---------- CONFIG ----------
IFACE = "wlo1"                 # Network interface
START_PORT = 32768             # Ephemeral port range start
END_PORT = 65535               # Ephemeral port range end
attacker_ip = "192.168.0.10"   # Attacker's private IP inside NAT
server_ip = "4.4.4.4"          # Victim server IP
SERVER_PORT = 22               # Target server port
nat_ip = "6.6.6.6"             # NAT public IP
# ----------------------------

Run the script in live mode:
sudo python3 port-infer-main.py --live

**Note:**  
The --live flag is mandatory. Without it, the script performs a cold run and does not infer active connections port.

---

## 2. Preparing the Port File

Once the client ports are inferred, store them in a file named port (one port per line).

Example port file:
44201
50000
60000

---

## 3. Inferring NATed Clients

### Configuration

Edit NATed-client-infer.py and update the configuration:

# ---------- Configuration ----------
ATTACKER_IP = "192.168.0.10"   # Attacker's private IP in the LAN
SERVER_IP = "4.4.4.4"          # Target server IP
SERVER_PORT = 22               # Target server port
IFACE = "wlo1"                 # Network interface

# NAT timing parameters (router-dependent)
WAIT_AFTER_RST = 11.0          # Time for NAT to clear mapping after RST
WAIT_AFTER_SYN = 2.0           # Additional wait after SYN
INTER_PROBE_DELAY = 1.0        # Delay between testing different clients
# ----------------------------------

**Note:**  
WAIT_AFTER_RST is NAT firmware dependent and must be determined either by:
- Inspecting router source code
- Empirical brute-force testing

---

## 4. Execute

### Using a Ports File
sudo ./NATed-client-infer.py --subnet-mask 24 --ports-file port

### Providing Ports via CLI
sudo ./NATed-client-infer.py \
  --subnet-mask 255.255.255.224 \
  --default-ports 50000 60000 44201

---
