# Cowrie-Honeypot-SSH-Attack-Detection
Implementing a hands-on defensive security lab using a Cowrie SSH honeypot.

Cowrie Honeypot — SSH Attack Detection & Network Forensics

📌 Project Overview

<img width="880" height="275" alt="image" src="https://github.com/user-attachments/assets/76fd11fc-f8c6-4cc4-807b-72eed47f2d06" />

🖥️ Lab Environment

<img width="485" height="114" alt="image" src="https://github.com/user-attachments/assets/904ddcb2-8358-488f-858a-a42900752f7a" />


Network: VirtualBox Host-Only Adapter — isolated lab environment.


⚙️ Phase 1 — Cowrie Honeypot Setup
# Start Cowrie
cowrie start
# Verify running status
cowrie status
# → cowrie is running (PID: 1099)
# Confirm port listening
ss -tuln | grep 2222
# → tcp LISTEN 0.0.0.0:2222

Key Cowrie configuration:

    º SSH listen endpoint: tcp:2222:interface=0.0.0.0
    º SSH version presented to clients: OpenSSH_7.9p1, OpenSSL 1.1.1a
    º Public key auth supported: ssh-rsa, ecdsa-sha2-nistp256, ssh-ed25519

Cowrie Honeypot Running


🔍 Phase 2 — Reconnaissance (Nmap Scan)

From the Kali attacker machine, a SYN scan was performed against the honeypot to confirm the SSH service was exposed.
nmap -sS -p 2222 192.168.56.103
Results:
PORT     STATE  SERVICE
2222/tcp open   EtherNetIP-1
MAC Address: 08:00:27:D6:2A:D6 (Oracle VirtualBox)

Nmap confirmed port 2222 was open — the attacker now had a target.


📡 Phase 3 — Live Traffic Capture (Wireshark)

Wireshark was launched on the Kali machine to capture live traffic on eth0 before and during the attack. ICMP echo request/reply packets were visible — confirming active network communication between 192.168.56.102 and 192.168.56.103.

The full capture was saved as cowrie_attack.pcap on the Kali desktop for forensic evidence.

⚡ Phase 4 — SSH Attack Simulation

The attacker connected to the honeypot using SSH on port 2222 with the root username, entering a password when prompted. Cowrie accepted the credentials and presented a simulated Debian GNU/Linux shell environment.
ssh root@192.168.56.103 -p 2222

What happened:

    º SSH key fingerprint presented: SHA256:j2Mu/UP3q5OwjnqEiDWsEOzuffC13yW4ulkkUOcAwM8
    º Warning: connection not using post-quantum key exchange
    º Attacker accepted the fingerprint and entered password
    º Cowrie logged in the attacker to a fake shell: root@svr04:~#
    º Session lasted 210.4 seconds before disconnecting

📋 Phase 5 — Cowrie Log Analysis

After the attack, Cowrie logs were reviewed to extract full evidence of the attacker's session.
tail -n 30 ~/cowrie/var/log/cowrie/cowrie.log

Key Log Events

<img width="661" height="444" alt="image" src="https://github.com/user-attachments/assets/280f0095-49e8-4d14-a7ff-f5adb6e1740d" />


🔐 Indicators of Compromise (IoCs)

<img width="587" height="336" alt="image" src="https://github.com/user-attachments/assets/97ce018c-663e-48dd-8dee-791f68922ab8" />


📁 Evidence Collected

<img width="609" height="189" alt="image" src="https://github.com/user-attachments/assets/c8ef82e2-04fb-4135-ad1a-9ba3a2db89aa" />


🛡️ Mitigation & Recommendations

<img width="766" height="223" alt="image" src="https://github.com/user-attachments/assets/a248b2a0-3554-4701-a237-21b929a5ceaf" />


🧰 Tools Used

<img width="594" height="263" alt="image" src="https://github.com/user-attachments/assets/b9dc6cd8-283f-47e1-b828-7f2d1f79f377" />


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

º Cowrie SSH honeypot deployment and configuration
º Live network traffic capture with Wireshark
º Attack simulation — reconnaissance, SSH authentication, session analysis
º Forensic log analysis and IoC extraction
º PCAP evidence collection and documentation
º Incident report writing



