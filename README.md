## 🍯 Cowrie Honeypot — SSH Attack Detection & Network Forensics

> A hands-on defensive security lab deploying a Cowrie SSH honeypot to capture, log, and analyse a simulated SSH-based attack. Network traffic was captured live using Wireshark, attack behaviour was recorded in Cowrie logs, and a full incident report was produced — demonstrating real-world threat detection and forensic evidence collection.

---

## 📌 Project Overview

| | |
|---|---|
| **Objective** | Deploy Cowrie honeypot, simulate SSH attack, capture network traffic, analyse attacker behaviour |
| **Honeypot** | Cowrie SSH Honeypot — listening on TCP port 2222 |
| **Attacker Machine** | Kali Linux — `10.0.2.7` |
| **Honeypot Machine** | Kali Linux (cowrie-honeypot) — `10.0.2.15` |
| **Network Capture** | Wireshark — `day1 pcap capture.pcap` |
| **Date** | March 9, 2026 |

---

## 🖥️ Lab Environment

| VM | Role | IP Address |
|---|---|---|
| **Kali Linux** | Attacker | `10.0.2.7` |
| **Kali Linux (Cowrie)** | Honeypot target | `10.0.2.15` |

Network: VirtualBox Host-Only Adapter — isolated lab environment.

---

After successfully installing kali linux and cowrie honeypot on 2 different VMS, I fired it up.  
Cowrie was deployed on a seperate Server Virtual Machine and configured to listen on TCP port 2222, emulating a real SSH service. I activate and startup Cowrie:

__🔍 Phase 1  Start Cowrie__

└─$ su cowrie

└─$ cd cowrie

source cowrie-envbinactivate

└─$ cowrie start
<img width="907" height="175" alt="image" src="https://github.com/user-attachments/assets/ac950c3c-4ac4-454d-86fa-b4b83c171a1b" />


__Verify running status__

└─$ cowrie status  

cowrie is running (PID 15420)  
<img width="310" height="38" alt="image" src="https://github.com/user-attachments/assets/d350f83b-8ff6-4d2e-bfeb-6ab3b5e2f45f" />


__Confirm port listening__

└─$ ss -tuln  grep 2222  
<img width="505" height="36" alt="image" src="https://github.com/user-attachments/assets/ab724e00-e404-4a37-b2b5-51ea50aff603" />


tcp   LISTEN 0      50           0.0.0.02222      0.0.0.0  

__Key Cowrie configurations__

    SSH listen endpoint tcp2222interface=0.0.0.0
    SSH version presented to clients OpenSSH_7.9p1, OpenSSL 1.1.1a
    Public key auth supported ssh-rsa, ecdsa-sha2-nistp256, ssh-ed25519

Confirmed Cowrie Honeypot Running

---

__🔍 Phase 2 — Reconnaissance (Nmap Scan)__

From the Kali attacker machine(10.0.2.7), active scans were performed against the honeypot on 10.0.2.15 to confirm the SSH service was exposed.

    nmap -sn 10.0.2.124
    nmap -v 10.0.2.15
    nmap -sV 10.0.2.15
    nmap -sS -p 2222 10.0.2.15

The purpose of the aboce is just to generate some traffic that will be captured by Wireshark for the sake of this lab trial.

Results

PORT     STATE  SERVICE

2222tcp open   EtherNetIP-1  
MAC Address 0800278329C7 (Oracle VirtualBox virtual NIC)  
<img width="502" height="48" alt="image" src="https://github.com/user-attachments/assets/125c99c3-3b57-44fb-8433-d98aef4d3056" />


Nmap confirmed port 2222 was open — the attacker now had a target.

---

__🔍Phase 3 — Live Traffic Capture (Wireshark)__

Wireshark was launched on the Kali machine to capture live traffic on eth0 before and during the attack.  
ICMP echo requestreply packets were visible — confirming active network communication between 10.0.2.7 and 10.0.2.15.  

The full capture was saved as 'day1 pcap capture.pcap' on the Kali desktop for forensic evidence.  

<img width="873" height="274" alt="image" src="https://github.com/user-attachments/assets/786d9436-3d61-4a25-91d7-38ba3d338d6c" />


Wireshark Live Capture PCAP Saved on Desktop PCAP in Wireshark

---

__🔍Phase 4 — SSH Attack Simulation__

The attacker connected to the honeypot using SSH on port 2222 with the root username, entering a password when prompted(Note This trail also allows you to logon without a password).  
Cowrie accepted the credentials and presented a simulated Debian GNULinux shell environment.

ssh -p 2222 root@10.0.2.15   
<img width="358" height="42" alt="image" src="https://github.com/user-attachments/assets/81b070b6-40fc-4143-a2a5-9a26985b5f2c" />


What happened:

    SSH key fingerprint presented SHA256j2MuUP3q5OwjnqEiDWsEOzuffC13yW4ulkkUOcAwM8
    Warning connection not using post-quantum key exchange
    Attacker accepted the fingerprint and entered password
    Cowrie logged in the attacker to a fake shell root@svr04~#
    Session lasted 210.4 seconds before disconnecting

SSH Attack Attempt

---

__🔍Phase 5 — Honeyport log of events__

After the attack, Cowrie logs were reviewed to extract full evidence of the attacker's session.

tail -f varlogcowriecowrie.log

Key Log Events  
Screenshots of the running honeypot and activity capture where you can visibly see the commands used for the attack were also saved as well. You could see where the active scanning attack (IP) was coming from as well as the attack date and time.  

This as we know is useful tactics for us to watch the attackers and figure out how they move in scenarios like this.



<img width="1180" height="365" alt="image" src="https://github.com/user-attachments/assets/89a5138d-5a86-40f5-ae0c-690e17e8a335" />  

<img width="1196" height="255" alt="image" src="https://github.com/user-attachments/assets/4a884d97-0924-4366-98db-50b0a03f9905" />  

<img width="771" height="220" alt="image" src="https://github.com/user-attachments/assets/a05a13ae-3a64-42d9-b0ad-183ec52791f0" />  


Cowrie Log Evidence

---

__🔍🔐 Indicators of Compromise (IoCs)__


Indicator | Value |
|-------------- | ---------  |
Attacker IP: | 10.0.2.7
Target Port:  | 	2222tcp
Protocol:  | SSH
Username Used:  | 	root
Password Submitted: | 	'none' (recorded by Cowrie)
Auth Method:	 | Password
Session Duration: | 210.4 seconds
Key Fingerprint: | 	SHA256j2MuUP3q5OwjnqEiDWsEOzuffC13yW4ulkkUOcAwM8

---

__🔍📁 Evidence Collected__

Evidence  | 	Description
|-------------- | ---------  |
cowrie_attack.pcap  | 	Full Wireshark packet capture of attack traffic
cowrie.log  | 	Cowrie session log — full authentication and shell activity
Nmap scan output  | 	Confirms port 2222 exposed pre-attack
Screenshots 	 | Visual evidence of each attack phase

---

__🔍️ Mitigation & Recommendations__
Finding  |	Recommendation  |
|-------------- | ---------  |
Direct root SSH access exposed |	Disable PermitRootLogin on production systems
Password authentication accepted |	Enforce key-based SSH authentication only
SSH exposed on non-standard port |	Apply firewall rules to restrict management port access by IP
No active intrusion detection |	Deploy IDSIPS alongside honeypots for real-time alerting
Long session undetected |  Monitor authentication logs continuously — alert on root logins

---

__🧰 Tools Used__  

Tool  | 	Purpose  
|-------------- | ---------  |  
Cowrie:  | 	SSH honeypot — captures attacker behaviour and credentials
Wireshark:  | 	Live packet capture and PCAP analysis
Nmap:  | 	Port scanning and service reconnaissance
Kali Linux(.7):  | 	Attacker machine
Kali Purple(.15):  | 	Honeypot host
VirtualBox : | 	Lab virtualisation and network isolation  

---

__📁 Repository Structure__

__📦 cowrie-honeypot-ssh-detection__

├── 📄 README.md

├── 📄 index.html

├── 📄 Incident_Report.docx

└── 📁 screenshots  
    Nmap scan results1.png  
    Honeypot log of events1.png  
    Honeypot log of events2.png1  
    Honeypot log of events3.png  
    SSH_Attack_Attempt.png  
    Cowrie_Log_Attack_Evidence.png  
    day1 Incident report on Honeypot Attack.docx  
    ssh attack.png  
    Reconnaissance.png  
    Cowrie start.png  
    Cowrie status.png  
    Cowrie_attack_pcap_file_saved_on_kali_desktop.png  
    Cowrie_attack_pcap_from_wireshark.png  

---

__⚠️ Disclaimer__

    All activities were conducted in an isolated VirtualBox virtual lab. The Cowrie honeypot and attacker machine were personally controlled VMs. No real systems or third-party infrastructure were targeted. Strictly for educational purposes.

---

__👤 Skills Demonstrated__

    Cowrie SSH honeypot deployment and configuration
    Live network traffic capture with Wireshark
    Attack simulation — reconnaissance, SSH authentication, session analysis
    Forensic log analysis and IoC extraction
    PCAP evidence collection and documentation
    Incident report writing

---

__Final Thoughts__

Setting up Cowrie was such a great first project. It forced me to think like an attacker while building skills in Linux, networking, and scripting.  
If you’re new to cybersecurity and wondering what to work on I highly recommend this.  
Thanks for taking your time to go through this lab experience and if you’re setting up your own honeypot, I’d love to hear how it goes!

