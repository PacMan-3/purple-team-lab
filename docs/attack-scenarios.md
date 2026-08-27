## Scenario 1: Network Service Scanning
**MITRE ATT&CK:** T1046 (Network Service Discovery)

**Attacker action (Kali):**
```
nmap -sV -p- 192.168.56.20
```

**What happens on the wire:** A burst of SYN packets to a wide range of destination ports from a single source in a short time window — the classic scan fingerprint.

**Detection:** Zeek's `conn.log` records connection attempts per port; a Splunk search flags a single source IP touching an unusually high number of distinct destination ports within a short window (see `detections/splunk_detections.spl`, query 1).

**Why this detection works:** Legitimate hosts rarely touch dozens of ports on the same destination in seconds. The threshold-based approach keeps false positives low while catching both fast and moderately slow scans.

---

## Scenario 2: SSH Brute Force
**MITRE ATT&CK:** T1110.001 (Brute Force: Password Guessing)

**Attacker action:**
```
hydra -l msfadmin -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.20
```

**What happens:** Many failed SSH authentication attempts in rapid succession from the same source.

**Detection:** A Suricata rule alerts on a high frequency of SSH connection attempts to the same destination within a short window; a companion Splunk query counts failed auth events per source IP per minute (query 2).

**Why this detection works:** Volume and timing, not content, are the signal — SSH traffic itself is encrypted, so the detection relies on connection metadata ie frequency and timing rather than payload inspection.

---

## Scenario 3: Exploitation of vsftpd 2.3.4 Backdoor
**MITRE ATT&CK:** T1190 (Exploit Public-Facing Application)

**Attacker action:**
```
msfconsole -q -x "use exploit/unix/ftp/vsftpd_234_backdoor; set RHOSTS 192.168.56.20; run"
```

**What happens:** The vulnerable vsftpd version opens a backdoor shell on port 6200 after a specific login string is sent — a well-documented CVE-2011-2523.

**Detection:** Zeek's `notice.log` and a custom Suricata signature both flag connections to the unusual backdoor port 6200, plus the anomalous FTP login pattern that triggers it (query 3, and `suricata.rules`).

**Why this detection works:** Port 6200 has no legitimate purpose on this host, its mere appearance in `conn.log` is a strong, low-noise indicator, unlike generic scan/brute-force detections that need thresholds.

---

## Scenario 4: Command & Control Callback
**MITRE ATT&CK:** T1071 (Application Layer Protocol, C2)

**Attacker action:** After gaining a shell via scenario 3, a reverse shell/Meterpreter session is established back to the Kali attacker on a non-standard port.

**Detection:** Zeek flags the long-lived, low-data-volume outbound connection pattern typical of a C2 beacon; Suricata's ET-style signature logic (adapted for this lab) flags the Meterpreter handshake pattern (query 4).

**Why this detection works:** C2 traffic has a distinct shape , regular packets over an unusually long connection that stands out from normal short lived service traffic even without decrypting payloads.

---

## Scenario 5: Data Exfiltration over HTTP
**MITRE ATT&CK:** T1041 (Exfiltration Over C2 Channel)

**Attacker action:** Simulated exfiltration of a sensitive file from the compromised target using `curl` POST to a listener on Kali.

**Detection:** Splunk query 5 flags outbound HTTP POST requests with unusually large payload sizes from a host that doesn't normally originate outbound HTTP traffic, cross-referenced with Zeek's `http.log`.

**Why this detection works:** Baselining "normal" outbound behavior per host is more reliable than trying to inspect payload content, especially once exfiltration is even lightly obfuscated.

---

