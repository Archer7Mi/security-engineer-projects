# Lab: Wireshark Packet Capture Analysis

**Environment:** Isolated VirtualBox lab  
**Objective:** Capture and analyse traffic to identify communication patterns and anomalies  
**Capture file:** `lab-capture-01.pcapng` (not committed  contains local IPs)

---

## 1. Capture Setup

```bash
# On Kali, capture on the host-only interface
wireshark &
# Interface: eth1 (192.168.56.x subnet)
# Filter: host 192.168.56.101
```

---

## 2. Protocol Breakdown

After a 5-minute capture during normal lab activity:

| Protocol | Packets | % of total |
|----------|---------|-----------|
| TCP | 1,842 | 61% |
| ARP | 312 | 10% |
| DNS | 287 | 10% |
| HTTP | 201 | 7% |
| ICMP | 89 | 3% |
| Other | 289 | 9% |

---

## 3. Notable Findings

### Finding 1: HTTP traffic in cleartext
Multiple HTTP GET requests captured to the Apache server with no TLS.

```
GET / HTTP/1.1
Host: 192.168.56.101
User-Agent: Mozilla/5.0 ...
```

**Risk:** Credentials or session tokens in HTTP are visible in plaintext. All web services should use HTTPS.

### Finding 2: ARP scan pattern
A burst of 47 ARP requests in under 2 seconds, all from `192.168.56.100`:

```
Who has 192.168.56.1? Tell 192.168.56.100
Who has 192.168.56.2? Tell 192.168.56.100
Who has 192.168.56.3? Tell 192.168.56.100
...
```

This is the pattern of a host discovery scan (consistent with `nmap -sn`). In a real environment this would be worth investigating if the source was unexpected.

---

## 4. Display Filters Used

```
# Filter to one host
ip.addr == 192.168.56.101

# Show only HTTP
http

# Show failed TCP connections (RST)
tcp.flags.reset == 1

# ARP only
arp

# DNS queries
dns
```
