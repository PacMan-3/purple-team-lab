# Zeek Log Reference 

Zeek writes structured, tab-separated logs by default (`conn.log`, `http.log`, `notice.log`, etc.) in `/opt/zeek/logs/current/`. This file lists the fields actually used for each scenario's detection.

## conn.log (used in scenarios 1, 2, 4)

| Field | Meaning |
|-------|---------|
| `ts` | Timestamp |
| `id.orig_h` / `id.orig_p` | Source IP / port |
| `id.resp_h` / `id.resp_p` | Destination IP / port |
| `proto` | Transport protocol |
| `duration` | Connection duration in seconds |
| `orig_bytes` / `resp_bytes` | Bytes sent each direction |
| `conn_state` | Connection state (e.g., `S0` = no reply, `SF` = normal establish/teardown) |

**Example (scan-like entry):**
```
ts=1724600001.2  id.orig_h=192.168.56.10  id.orig_p=51422  id.resp_h=192.168.56.20  id.resp_p=445  proto=tcp  duration=0.001  conn_state=S0
```
A very short duration with `conn_state=S0` repeated across many destination ports is the scan fingerprint referenced in Splunk query 1.

## http.log (used in scenario 5)

| Field | Meaning |
|-------|---------|
| `method` | HTTP method (GET, POST, etc.) |
| `host` | Host header |
| `uri` | Requested URI |
| `request_body_len` | Size of the request body in bytes |
| `status_code` | Response status code |

## notice.log (used in scenario 3)

Zeek raises entries here when a built in or custom script detects something notable. The `note` field names the triggering script/policy, and `msg` gives a  description; this is what Splunk query 3 filters on.

