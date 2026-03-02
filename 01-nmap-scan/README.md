# Lab: Nmap Host Discovery and Port Scanning

**Environment:** Isolated VirtualBox lab (Kali Linux attacker, Ubuntu target)  
**Objective:** Identify live hosts and open services on the lab subnet

---

## 1. Host Discovery

Find which hosts are alive on the /24 subnet before doing port scans.

```bash
nmap -sn 192.168.56.0/24
```

### Output (abbreviated)
```
Starting Nmap 7.94 ( https://nmap.org )
Nmap scan report for 192.168.56.1 (gateway)
Host is up (0.00040s latency).
Nmap scan report for 192.168.56.101 (ubuntu-target)
Host is up (0.00081s latency).
2 hosts up out of 256 scanned
```

2 hosts alive: the gateway and the Ubuntu target at `192.168.56.101`.

---

## 2. Service and Version Scan

```bash
nmap -sV -sC -p- 192.168.56.101 -oN scan_results.txt
```

Flags used:
- `-sV`  detect service versions
- `-sC`  run default scripts
- `-p-`  scan all 65535 ports
- `-oN`  save output to file

### Results
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu (protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.52
| http-title: Apache2 Ubuntu Default Page
3306/tcp open  mysql   MySQL 8.0.32
```

---

## 3. Observations

| Port | Service | Finding |
|------|---------|---------|
| 22 | SSH | Open  should restrict to key-based auth |
| 80 | HTTP | Apache default page visible  no content deployed yet |
| 3306 | MySQL | Exposed on network  should be bound to localhost only |

MySQL (3306) being accessible on the network is a misconfiguration. In a real environment this would be flagged for immediate remediation  databases should not be directly accessible on the network without a firewall rule restricting it to the application server only.

---

## 4. Remediation Notes

```bash
# Bind MySQL to localhost only  edit /etc/mysql/mysql.conf.d/mysqld.cnf
bind-address = 127.0.0.1

# Restrict SSH password auth
PasswordAuthentication no

# Disable Apache default page
sudo a2dissite 000-default.conf
```
