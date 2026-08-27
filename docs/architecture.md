# Lab Architecture

## Goal

Build a small, fully isolated environment where offensive actions can be safely generated and observed end-to-end at the network and log level, without any traffic reaching the internet or the host LAN.

## Topology

| VM | Role | OS | IP | Specs |
|----|------|----|----|-------|
| Kali | Attacker | Kali Linux (latest) | 192.168.56.10 | 2 vCPU / 4 GB RAM |
| Metasploitable 2 | Vulnerable target | Ubuntu 8.04 (intentionally outdated) | 192.168.56.20 | 1 vCPU / 512 MB RAM |
| SecOps | SIEM / sensor | Ubuntu Server 22.04 + Splunk (free tier), Zeek, Suricata | 192.168.56.30 | 2 vCPU / 4 GB RAM |

## Networking

- VirtualBox , subnet `192.168.56.0/24`
- No NAT/bridged adapter attached to Kali or Metasploitable since they cannot reach the internet 
- The SecOps VM has a second NIC (NAT) used only to download Splunk/Zeek/Suricata packages during setup, then could be optionally disabled

## Log flow

```
Kali attack traffic
      │
      ▼
Metasploitable 2 (target, generates logs/responses)
      │
      ▼  (mirrored via host-only segment)
Zeek + Suricata on SecOps VM  →  conn.log / http.log / notice.log / alert.log
      │
      ▼
Splunk (Universal Forwarder or direct file monitor)  →  indexed, searchable
```

