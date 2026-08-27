# Purple Team Home Lab: Attack Simulation & Detection Engineering

A self-hosted purple team lab where I simulate real-world attack techniques mapped to MITRE ATT&CK against a small vulnerable environment, then build and validate detections for them using Splunk, Zeek, and Suricata, bridging both offensive and defensive security to demonstrate the full loop.


## Architecture

```
                     Host-Only Network (192.168.56.0/24)
   ┌───────────────┐        ┌───────────────┐        ┌───────────────┐
   │     Kali      │  --->  │ Metasploitable│  --->  │ Ubuntu/Splunk │
   │  (Attacker)   │        │ 2 (Target)    │        │ +Zeek/Suricata│
   │ 192.168.56.10 │        │ 192.168.56.20 │        │ 192.168.56.30 │
   └───────────────┘        └───────────────┘        └───────────────┘
```

See [`docs/architecture.md`](docs/architecture.md) for the full network design and VM specs.

## Attack scenarios covered

| # | Technique | MITRE ATT&CK ID | Detection source |
|---|-----------|------------------|-------------------|
| 1 | Network service scanning (Nmap) | T1046 | Zeek `conn.log` + Splunk |
| 2 | Brute force — SSH | T1110.001 | Suricata + Splunk |
| 3 | Exploitation of vsftpd 2.3.4 backdoor | T1190 | Zeek `notice.log` + Suricata |
| 4 | Command & control callback (Metasploit) | T1071 | Zeek + Suricata signature |
| 5 | Data exfiltration over HTTP | T1041 | Splunk + Zeek `http.log` |

Full scenario write-ups, including attacker commands and the reasoning behind each detection, are in [`docs/attack-scenarios.md`](docs/attack-scenarios.md).

## Detections

- [`detections/splunk_detections.spl`](detections/splunk_detections.spl) — SPL queries for each scenario, with false-positive notes
- [`detections/suricata.rules`](detections/suricata.rules) — custom Suricata signatures
- [`detections/zeek_notes.md`](detections/zeek_notes.md) — relevant Zeek log fields and example entries per scenario

## Incident reporting

Each scenario is documented using a lightweight incident report template, aligned loosely with NIST SP 800-61 phases (Detection & Analysis, Containment, Post-Incident Activity).

## Lab setup

Built on VirtualBox with Host-Only networking to keep all traffic isolated from the host network. 

## Disclaimer

All testing was performed exclusively against intentionally vulnerable machines inside an isolated, host-only virtual network with no internet routing. Nothing here targets third-party systems.
