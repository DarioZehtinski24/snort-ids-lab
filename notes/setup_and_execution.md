Snort Detection Lab – Setup and Execution Summary
Project Purpose

The purpose of this project was to set up Snort on Kali Linux and build a small intrusion detection lab using multiple custom detection rules along with Snort’s built-in port scan detection.

The objective was to simulate realistic alerting scenarios and document the results in a clear and reproducible way.

Project Environment

The lab was built using two machines:

Kali Linux machine acting as the Snort sensor
Second machine used to generate test traffic

The monitored network interface on the sensor machine was:

eth0
1. Validating the Snort Configuration

After installation and configuration, Snort was tested to verify that the configuration file and rules were loaded correctly.

Command Used
sudo snort -c /etc/snort/snort.lua -T
Explanation
sudo → runs the command with administrative privileges
snort → starts the Snort engine
-c /etc/snort/snort.lua → specifies the configuration file
-T → performs a configuration test without live monitoring
Result

Snort successfully validated the configuration and confirmed that the local rules and enabled modules were loaded without errors.

2. Creating Custom Detection Rules

Custom rules were added to the local rules file.

Command Used
sudo nano /etc/snort/rules/local.rules
Rules Added
alert tcp any any -> 192.168.0.101 22 (msg:"LOCAL Repeated SSH Connection Attempts"; flags:S; detection_filter:track by_src,count 5,seconds 30; sid:1000001; rev:2;)

alert http any any -> 192.168.0.101 any (msg:"LOCAL Web Admin Probe"; http_uri; content:"/admin",nocase; sid:1000002; rev:2;)

alert tcp any any -> 192.168.0.101 445 (msg:"LOCAL SMB Probe Detected"; flags:S; sid:1000003; rev:1;)

alert tcp any any -> 192.168.0.101 3389 (msg:"LOCAL RDP Probe Detected"; flags:S; sid:1000004; rev:1;)
Rule Purpose
SSH rule → detects repeated TCP SYN attempts to port 22
HTTP rule → detects requests containing /admin
SMB rule → detects TCP SYN probes to port 445
RDP rule → detects TCP SYN probes to port 3389
3. Loading Local Rules in Snort

The main Snort configuration file was modified to load the local rules file.

Command Used
sudo nano /etc/snort/snort.lua
IPS Configuration
ips = {
    enable_builtin_rules = true,
    variables = default_variables,
    rules = [[
        include /etc/snort/rules/local.rules
    ]]
}
Explanation
enable_builtin_rules = true → enables built-in decoder and inspector alerts
variables = default_variables → keeps default rule variables
include ... → loads the custom local rules file
4. Built-in Port Scan Detection

Snort’s built-in port_scan inspector was enabled to detect TCP port scans.

Configuration
port_scan = {
    watch_ip = '192.168.0.101',
    alert_all = true,
    protos = 'tcp',
    scan_types = 'portscan'
}
Explanation
watch_ip → monitored target host
alert_all = true → logs all matching events
protos = 'tcp' → TCP only
scan_types = 'portscan' → scan-specific detection
5. Suppressing Unwanted Background Alerts

To reduce noise from unrelated built-in alerts, selected events were suppressed.

Configuration
suppress = {
    { gid = 112, sid = 1 },
    { gid = 116, sid = 444 },
    { gid = 116, sid = 434 }
}
Purpose

This was used to suppress background ARP and IPv4 option alerts so logs remained focused on:

SSH
SMB
RDP
HTTP
Port scan activity

6. Starting Live Monitoring

Snort was started in live monitoring mode on interface eth0.

Command Used
sudo snort -i eth0 -c /etc/snort/snort.lua -A alert_fast
Explanation
-i eth0 → tells Snort to inspect traffic on interface eth0
-c /etc/snort/snort.lua → loads the Snort configuration
-A alert_fast → displays alerts in a readable terminal format

This mode was used to observe alerts in real time during traffic testing.

7. JSON Alert Logging

Snort was also started in JSON logging mode to generate machine-readable evidence files.

Command Used
sudo snort -i eth0 -c /etc/snort/snort.lua -A alert_json \
-l ~/snort-ids-lab/evidence/json \
--lua "alert_json = {file = true, fields = 'timestamp proto src_ap dst_ap rule msg action'}"
Explanation
-A alert_json → enables JSON alert output
-l ... → specifies the output directory
fields = ... → keeps only the most relevant fields
Output Fields

The JSON logs included:

timestamp
proto
src_ap
dst_ap
rule
msg
action

This made the alerts easy to review and suitable for evidence collection.

8. Test Traffic Generation

A second machine was used to generate controlled traffic toward the monitored host.

Repeated SSH Attempts
for i in {1..6}; do
    nc -zv 192.168.0.101 22
    sleep 1
done
SMB Probe
nc -zv 192.168.0.101 445
RDP Probe
nc -zv 192.168.0.101 3389
HTTP Admin Probe
curl http://192.168.0.101/admin
TCP Port Scan
sudo nmap -Pn -sS 192.168.0.101
Explanation
nc was used for direct TCP connection attempts
curl was used to simulate HTTP web traffic
nmap -Pn -sS generated a TCP SYN port scan without host discovery

This traffic was intentionally crafted to trigger the configured Snort rules.

9. Observed Results

The project successfully generated alerts for all configured use cases.

Example Alerts Observed
LOCAL Repeated SSH Connection Attempts
LOCAL SMB Probe Detected
LOCAL RDP Probe Detected
LOCAL Web Admin Probe
(port_scan) TCP portscan

JSON evidence files confirmed that the alerts were triggered against the monitored host:

192.168.0.101

This validated both the custom rules and Snort’s built-in detection capabilities.

10. Project File Collection

Relevant project files were copied into a dedicated project directory for organization and documentation.

Commands Used
mkdir -p ~/snort-ids-lab/{rules,configs,notes,evidence}

cp /etc/snort/rules/local.rules ~/snort-ids-lab/rules/

cp /etc/snort/snort.lua ~/snort-ids-lab/configs/
Directory Purpose
rules/ → custom Snort rules
configs/ → Snort configuration files
notes/ → project documentation
evidence/ → alert output and validation files
Final Result

The project successfully demonstrated a working Snort-based intrusion detection lab on Kali Linux.

Custom rules for:

SSH
SMB
RDP
HTTP probing

were successfully implemented and validated.

Additionally, Snort’s built-in port scan inspector was used to detect TCP scanning activity.

Alerts were successfully recorded in both:

terminal output
JSON evidence files

This confirms that the lab can reliably detect multiple types of suspicious network behavior.

Skills Demonstrated

This project demonstrates practical skills in:

Snort installation and configuration
custom IDS rule creation
Linux command-line administration
live network traffic inspection
JSON alert logging
alert tuning and suppression
multi-host testing and validation
technical cybersecurity documentation
