# Snort Detection Lab – Setup and Execution Summary

## Project Purpose

The purpose of this project was to set up Snort on Kali Linux and build a small intrusion detection lab using multiple custom detection rules along with Snort’s built-in port scan detection.

The objective was to simulate realistic alerting scenarios and document the results in a clear and reproducible way.

---

## Project Environment

The lab was built using two machines:

- Kali Linux machine acting as the Snort sensor
- Second machine used to generate test traffic

The monitored network interface on the sensor machine was:

`eth0`

---

## 1. Validating the Snort Configuration

After installation and configuration, Snort was tested to verify that the configuration file and rules were loaded correctly.

### Command Used

```bash
sudo snort -c /etc/snort/snort.lua -T
```

### Explanation

- `sudo` runs the command with administrative privileges
- `snort` starts the Snort engine
- `-c /etc/snort/snort.lua` specifies the Snort configuration file
- `-T` performs a configuration test without starting live monitoring

### Result

Snort successfully validated the configuration and confirmed that the local rules and enabled modules were loaded without errors.

---

## 2. Creating Custom Detection Rules

Custom rules were added to the local rules file.

### Command Used

```bash
sudo nano /etc/snort/rules/local.rules
```

### Rules Added

```bash
alert tcp any any -> 192.168.0.101 22 (msg:"LOCAL Repeated SSH Connection Attempts"; flags:S; detection_filter:track by_src,count 5,seconds 30; sid:1000001; rev:2;)

alert http any any -> 192.168.0.101 any (msg:"LOCAL Web Admin Probe"; http_uri; content:"/admin",nocase; sid:1000002; rev:2;)

alert tcp any any -> 192.168.0.101 445 (msg:"LOCAL SMB Probe Detected"; flags:S; sid:1000003; rev:1;)

alert tcp any any -> 192.168.0.101 3389 (msg:"LOCAL RDP Probe Detected"; flags:S; sid:1000004; rev:1;)
```

### Explanation

- SSH rule detects repeated TCP SYN attempts to port 22
- HTTP rule detects requests containing `/admin`
- SMB rule detects TCP SYN probes to port 445
- RDP rule detects TCP SYN probes to port 3389

---

## 3. Loading Local Rules in Snort

The main Snort configuration file was modified so that the local rules file would be loaded.

### Command Used

```bash
sudo nano /etc/snort/snort.lua
```

### IPS Configuration

```lua
ips = {
    enable_builtin_rules = true,
    variables = default_variables,
    rules = [[
        include /etc/snort/rules/local.rules
    ]]
}
```

### Explanation

- `enable_builtin_rules = true` allows built-in decoder and inspector alerts
- `variables = default_variables` keeps the default rule variables
- `include /etc/snort/rules/local.rules` loads the custom rule file

---

## 4. Built-in Port Scan Detection

In addition to custom rules, Snort’s built-in `port_scan` inspector was enabled and used to detect TCP port scans.

### Configuration

```lua
port_scan = {
    watch_ip = '192.168.0.101',
    alert_all = true,
    protos = 'tcp',
    scan_types = 'portscan'
}
```

### Explanation

- `watch_ip` focuses detection on the monitored host
- `alert_all = true` allows all matching events to be logged
- `protos = 'tcp'` limits detection to TCP scans
- `scan_types = 'portscan'` focuses on port scanning activity

---

## 5. Suppressing Unwanted Background Alerts

To reduce noise from unrelated built-in alerts, selected events were suppressed.

### Configuration

```lua
suppress = {
    { gid = 112, sid = 1 },
    { gid = 116, sid = 444 },
    { gid = 116, sid = 434 }
}
```

### Explanation

This was used to suppress background ARP and IPv4 option alerts so the logs remained focused on:

- SSH
- SMB
- RDP
- HTTP
- port scan events

---

## 6. Starting Live Monitoring

Snort was started in live monitoring mode on interface `eth0`.

### Command Used

```bash
sudo snort -i eth0 -c /etc/snort/snort.lua -A alert_fast
```

### Explanation

- `-i eth0` tells Snort to inspect traffic on interface `eth0`
- `-A alert_fast` prints alerts in a readable terminal format

---

## 7. JSON Alert Logging

Snort was also started in JSON logging mode to create machine-readable evidence files.

### Command Used

```bash
sudo snort -i eth0 -c /etc/snort/snort.lua -A alert_json -l ~/snort-ids-lab/evidence/json --lua "alert_json = {file = true, fields = 'timestamp proto src_ap dst_ap rule msg action'}"
```

### Explanation

- `-A alert_json` enables JSON alert output
- `-l` sets the output directory
- `fields` keeps the JSON output focused on the most useful fields

---

## 8. Test Traffic Generation

A second machine was used to generate controlled traffic toward the monitored host.

### Repeated SSH Attempts

```bash
for i in {1..6}; do
    nc -zv 192.168.0.101 22
    sleep 1
done
```

### SMB Probe

```bash
nc -zv 192.168.0.101 445
```

### RDP Probe

```bash
nc -zv 192.168.0.101 3389
```

### HTTP Admin Probe

```bash
curl http://192.168.0.101/admin
```

### TCP Port Scan

```bash
sudo nmap -Pn -sS 192.168.0.101
```

---

## 9. Observed Results

The project successfully generated alerts for the configured use cases.

### Example Alerts

```text
LOCAL Repeated SSH Connection Attempts
LOCAL SMB Probe Detected
LOCAL RDP Probe Detected
LOCAL Web Admin Probe
(port_scan) TCP portscan
```

JSON evidence files confirmed that the alerts were triggered against the monitored host `192.168.0.101`.

---

## 10. Project File Collection

Relevant project files were copied into a dedicated project directory.

### Commands Used

```bash
mkdir -p ~/snort-ids-lab/{rules,configs,notes,evidence}

cp /etc/snort/rules/local.rules ~/snort-ids-lab/rules/

cp /etc/snort/snort.lua ~/snort-ids-lab/configs/
```

### Directory Purpose

- `rules` stores the custom rule file
- `configs` stores the Snort configuration
- `notes` stores project documentation
- `evidence` stores alert output and validation files

---

## Final Result

The project successfully demonstrated a working Snort-based detection lab on Kali Linux.

Custom rules for SSH, SMB, RDP, and HTTP probing were implemented and validated, while the built-in port scan inspector was used to detect scanning activity.

Alerts were successfully recorded both in terminal output and in JSON evidence files.

---

## Skills Demonstrated

This project demonstrates practical skills in:

- Snort installation and configuration
- custom IDS rule creation
- Linux command-line administration
- live network traffic inspection
- JSON alert logging
- basic alert tuning and suppression
- testing and validation from a second machine
- technical documentation of a cybersecurity lab project
