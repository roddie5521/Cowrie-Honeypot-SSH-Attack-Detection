# Cowrie-Honeypot-SSH-Attack-Detection
Implementing a hands-on defensive security lab using a Cowrie SSH honeypot.

Cowrie Honeypot — SSH Attack Detection & Network Forensics

    A hands-on defensive security lab deploying a Cowrie SSH honeypot to capture, log, and analyse a simulated SSH-based attack. Network traffic was captured live using Wireshark, attack behaviour was recorded in Cowrie logs, and a full incident report was produced — demonstrating real-world threat detection and forensic evidence collection.

📌 Project Overview
	
Objective 	Deploy Cowrie honeypot, simulate SSH attack, capture network traffic, analyse attacker behaviour
Honeypot 	Cowrie SSH Honeypot — listening on TCP port 2222
Attacker Machine 	Kali Linux — 192.168.56.102
Honeypot Machine 	Ubuntu Server (cowrie-honeypot) — 192.168.56.103
Network Capture 	Wireshark — cowrie_attack.pcap
Date 	January 29, 2026
Course 	Bincom Academy Cybersecurity Class — February 2026
🖥️ Lab Environment
VM 	Role 	IP Address
Kali Linux 	Attacker 	192.168.56.102
Ubuntu Server (Cowrie) 	Honeypot target 	192.168.56.103

Network: VirtualBox Host-Only Adapter — isolated lab environment.
⚙️ Phase 1 — Cowrie Honeypot Setup

Cowrie was deployed on the Ubuntu Server VM and configured to listen on TCP port 2222, emulating a real SSH service.

# Start Cowrie
cowrie start

# Verify running status
cowrie status
# → cowrie is running (PID: 1099)

# Confirm port listening
ss -tuln | grep 2222
# → tcp LISTEN 0.0.0.0:2222

Key Cowrie configuration:

    SSH listen endpoint: tcp:2222:interface=0.0.0.0
    SSH version presented to clients: OpenSSH_7.9p1, OpenSSL 1.1.1a
    Public key auth supported: ssh-rsa, ecdsa-sha2-nistp256, ssh-ed25519

Cowrie Honeypot Running
🔍 Phase 2 — Reconnaissance (Nmap Scan)

From the Kali attacker machine, a SYN scan was performed against the honeypot to confirm the SSH service was exposed.

nmap -sS -p 2222 192.168.56.103

Results:

PORT     STATE  SERVICE
2222/tcp open   EtherNetIP-1
MAC Address: 08:00:27:D6:2A:D6 (Oracle VirtualBox)

Nmap confirmed port 2222 was open — the attacker now had a target.

Nmap Scan
📡 Phase 3 — Live Traffic Capture (Wireshark)

Wireshark was launched on the Kali machine to capture live traffic on eth0 before and during the attack. ICMP echo request/reply packets were visible — confirming active network communication between 192.168.56.102 and 192.168.56.103.

The full capture was saved as cowrie_attack.pcap on the Kali desktop for forensic evidence.

Wireshark Live Capture PCAP Saved on Desktop PCAP in Wireshark
⚡ Phase 4 — SSH Attack Simulation

The attacker connected to the honeypot using SSH on port 2222 with the root username, entering a password when prompted. Cowrie accepted the credentials and presented a simulated Debian GNU/Linux shell environment.

ssh root@192.168.56.103 -p 2222

What happened:

    SSH key fingerprint presented: SHA256:j2Mu/UP3q5OwjnqEiDWsEOzuffC13yW4ulkkUOcAwM8
    Warning: connection not using post-quantum key exchange
    Attacker accepted the fingerprint and entered password
    Cowrie logged in the attacker to a fake shell: root@svr04:~#
    Session lasted 210.4 seconds before disconnecting

SSH Attack Attempt
📋 Phase 5 — Cowrie Log Analysis

After the attack, Cowrie logs were reviewed to extract full evidence of the attacker's session.

tail -n 30 ~/cowrie/var/log/cowrie/cowrie.log

Key Log Events
Timestamp 	Event
2026-01-29T11:30:52 	SSH client hash fingerprint recorded
2026-01-29T11:30:52 	Key exchange algorithm negotiated
2026-01-29T11:31:06 	SSH userauth service started
2026-01-29T11:31:06 	root tried auth none (failed)
2026-01-29T11:31:22 	root tried auth password
2026-01-29T11:31:22 	Login attempt [root/tshtkls] — succeeded (Cowrie accepted)
2026-01-29T11:31:22 	Emulated server initialised as linux-x64-lsb
2026-01-29T11:31:22 	root authenticated with password
2026-01-29T11:31:22 	Shell session opened
2026-01-29T11:34:22 	TTY log closed after 180.1 seconds
2026-01-29T11:34:22 	root logged out — connection lost after 210.4 seconds

Cowrie Log Evidence
🔐 Indicators of Compromise (IoCs)
Indicator 	Value
Attacker IP 	192.168.56.102
Target Port 	2222/tcp
Protocol 	SSH
Username Used 	root
Password Submitted 	tshtkls (recorded by Cowrie)
Auth Method 	Password
Session Duration 	210.4 seconds
Key Fingerprint 	SHA256:j2Mu/UP3q5OwjnqEiDWsEOzuffC13yW4ulkkUOcAwM8
📁 Evidence Collected
Evidence 	Description
cowrie_attack.pcap 	Full Wireshark packet capture of attack traffic
cowrie.log 	Cowrie session log — full authentication and shell activity
Nmap scan output 	Confirms port 2222 exposed pre-attack
Screenshots 	Visual evidence of each attack phase
🛡️ Mitigation & Recommendations
Finding 	Recommendation
Direct root SSH access exposed 	Disable PermitRootLogin on production systems
Password authentication accepted 	Enforce key-based SSH authentication only
SSH exposed on non-standard port 	Apply firewall rules to restrict management port access by IP
No active intrusion detection 	Deploy IDS/IPS alongside honeypots for real-time alerting
Long session undetected 	Monitor authentication logs continuously — alert on root logins
🧰 Tools Used
Tool 	Purpose
Cowrie 	SSH honeypot — captures attacker behaviour and credentials
Wireshark 	Live packet capture and PCAP analysis
Nmap 	Port scanning and service reconnaissance
Kali Linux 	Attacker machine
Ubuntu Server 	Honeypot host
VirtualBox 	Lab virtualisation and network isolation
📁 Repository Structure

📦 cowrie-honeypot-ssh-detection/
├── 📄 README.md
├── 📄 index.html
├── 📄 Incident_Report_Network_Attack_Detection.docx
├── 📦 06_cowrie_attack_pcap.pcap
└── 📁 screenshots/
    ├── 01_Kali_Wireshark_Live_Capture.png
    ├── 02_Cowrie_Honeypot_Running.png
    ├── 03_Nmap_Scan_Port_2222.png
    ├── 04_SSH_Attack_Attempt.png
    ├── 05_Cowrie_Log_Attack_Evidence.png
    ├── cowrie_attack_pcap_file_saved_on_kali_desktop.png
    └── cowrie_attack_pcap_from_wireshark.png

⚠️ Disclaimer

    All activities were conducted in an isolated VirtualBox virtual lab. The Cowrie honeypot and attacker machine were personally controlled VMs. No real systems or third-party infrastructure were targeted. Strictly for educational purposes.

👤 Skills Demonstrated

    Cowrie SSH honeypot deployment and configuration
    Live network traffic capture with Wireshark
    Attack simulation — reconnaissance, SSH authentication, session analysis
    Forensic log analysis and IoC extraction
    PCAP evidence collection and documentation
    Incident report writing
