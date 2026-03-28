# Project Notes

## Detection Scenarios

The following detection scenarios were implemented and validated:

- repeated SSH connection attempts
- SMB probe detection
- RDP probe detection
- HTTP probing for the `/admin` URI
- built-in TCP port scan detection

---

## Custom Detection Logic

- SSH rule monitors TCP SYN packets to port `22` and alerts when the same source sends **5 attempts within 30 seconds**
- SMB rule alerts on TCP SYN packets to port `445`
- RDP rule alerts on TCP SYN packets to port `3389`
- HTTP rule alerts when a request contains the `/admin` URI
- Snort built-in `port_scan` inspector detects TCP port scanning activity

---

## Logging and Evidence

- Snort was executed in live monitoring mode
- alerts were collected in JSON format
- evidence files were saved for validation and documentation

---

## Test Method

- run Snort on the sensor machine
- use a second machine to generate controlled traffic
- test SSH, SMB, RDP, and HTTP-related detection scenarios
- test port scanning with Nmap
- confirm alert generation in terminal output and JSON logs

---

## Result

Snort successfully detected custom rule matches and built-in port scan activity in a controlled lab environment.

The project demonstrated:

- rule creation
- live traffic inspection
- alert logging
- evidence validation
- technical documentation
