# Snort Detection Lab – Setup and Execution Summary

## Project purpose
The purpose of this project was to set up Snort on Kali Linux and build a small intrusion detection lab with multiple custom detection rules and built-in port scan detection. The goal was to simulate realistic alerting scenarios and document the results in a reproducible way.

## Project environment
The lab was built with two machines:
- a Kali Linux machine acting as the Snort sensor
- a second machine used to generate test traffic

The monitored network interface on the sensor machine was `eth0`.

## 1. Validating the Snort configuration
After installation and configuration, Snort was tested to verify that the configuration file and rules were loaded correctly.

Command used:

bash
sudo snort -c /etc/snort/snort.lua -T

Explanation:

sudo runs the command with administrative privileges
snort starts the Snort engine
-c /etc/snort/snort.lua specifies the Snort configuration file
-T performs a configuration test without starting live monitoring

Result:
Snort successfully validated the configuration and confirmed that the local rules and enabled modules were loaded without errors.

2. Creating custom detection rules

Custom rules were added to the local rules file.

Command used:

sudo nano /etc/snort/rules/local.rules

alert tcp any any -> 192.168.0.101 22 (msg:"LOCAL Repeated SSH Connection Attempts"; flags:S; detection_filter:track by_src,count 5,seconds 30; sid:1000001; rev:2;)

alert http any any -> 192.168.0.101 any (msg:"LOCAL Web Admin Probe"; http_uri; content:"/admin",nocase; sid:1000002; rev:2;)

alert tcp any any -> 192.168.0.101 445 (msg:"LOCAL SMB Probe Detected"; flags:S; sid:1000003; rev:1;)

alert tcp any any -> 192.168.0.101 3389 (msg:"LOCAL RDP Probe Detected"; flags:S; sid:1000004; rev:1;)

Explanation:

the SSH rule detects repeated TCP SYN attempts to port 22 from the same source
the HTTP rule detects requests containing /admin
the SMB rule detects TCP SYN probes to port 445
the RDP rule detects TCP SYN probes to port 3389

3. Loading local rules in Snort

The main Snort configuration file was modified so that the local rules file would be loaded.

Command used:
sudo nano /etc/snort/snort.lua

The ips section included the local rules file:
ips =
{
    enable_builtin_rules = true,
    variables = default_variables,
    rules = [[
include /etc/snort/rules/local.rules
    ]]
}

Explanation:

enable_builtin_rules = true allows built-in decoder and inspector alerts
variables = default_variables keeps the default rule variables
include /etc/snort/rules/local.rules loads the custom rule file

4. Built-in port scan detection

In addition to custom rules, Snort’s built-in port_scan inspector was enabled and used to detect TCP port scans.

Example configuration:

port_scan =
{
    watch_ip = '192.168.0.101',
    alert_all = true,
    protos = 'tcp',
    scan_types = 'portscan'
}


Explanation:

watch_ip focuses detection on the monitored host
alert_all = true allows all matching events to be logged
protos = 'tcp' limits detection to TCP scans
scan_types = 'portscan' focuses on port scanning activity

5. Suppressing unwanted background alerts

To reduce noise from unrelated built-in alerts, selected events were suppressed.

Example configuration:
suppress =
{
    { gid = 112, sid = 1 },
    { gid = 116, sid = 444 },
    { gid = 116, sid = 434 }
}

Explanation:

this was used to suppress background ARP and IPv4 option alerts
the purpose was to keep the logs focused on SSH, SMB, RDP, HTTP and port scan events

6. Starting live monitoring

Snort was started in live monitoring mode on interface eth0.

Command used:
sudo snort -i eth0 -c /etc/snort/snort.lua -A alert_fast

Explanation:

-i eth0 tells Snort to inspect traffic on interface eth0
-A alert_fast prints alerts in a readable terminal format

7. JSON alert logging

Snort was also started in JSON logging mode to create machine-readable evidence files.

Command used:
sudo snort -i eth0 -c /etc/snort/snort.lua -A alert_json -l ~/snort-ids-lab/evidence/json --lua "alert_json = {file = true, fields = 'timestamp proto src_ap dst_ap rule msg action'}"

Explanation:

-A alert_json enables JSON alert output
-l ~/snort-ids-lab/evidence/json sets the output directory
fields = 'timestamp proto src_ap dst_ap rule msg action' keeps the JSON output focused on the most useful fields

8. Test traffic generation

A second machine was used to generate controlled traffic toward the monitored host.

Repeated SSH attempts
for i in {1..6}; do nc -zv 192.168.0.101 22; sleep 1; done

SMB probe 
nc -zv 192.168.0.101 445

RDP probe
nc -zv 192.168.0.101 3389

HTTP admin probe
curl http://192.168.0.101/admin

TCP port scan
sudo nmap -Pn -sS 192.168.0.101

Explanation:

nc was used to generate direct TCP connection attempts
curl was used to generate HTTP traffic
nmap -Pn -sS was used to generate a TCP SYN port scan without host discovery

9. Observed results

The project successfully generated alerts for the configured use cases.

Examples of observed alerts included:

LOCAL Repeated SSH Connection Attempts
LOCAL SMB Probe Detected
LOCAL RDP Probe Detected
LOCAL Web Admin Probe
(port_scan) TCP portscan

JSON evidence files confirmed that the alerts were triggered against the monitored host 192.168.0.101.

10. Project file collection

Relevant project files were copied into a dedicated project directory.

Example commands used:
mkdir -p ~/snort-ids-lab/{rules,configs,notes,evidence}
cp /etc/snort/rules/local.rules ~/snort-ids-lab/rules/
cp /etc/snort/snort.lua ~/snort-ids-lab/configs/

Explanation:

rules stores the custom rule file
configs stores the Snort configuration
notes stores the project documentation
evidence stores alert output and validation files
Final result

The project successfully demonstrated a working Snort-based detection lab on Kali Linux. Custom rules for SSH, SMB, RDP and HTTP probing were implemented and validated, while the built-in port scan inspector was used to detect scanning activity. Alerts were successfully recorded both in terminal output and in JSON evidence files.

Skills demonstrated

This project demonstrates practical skills in:

Snort installation and configuration
custom IDS rule creation
Linux command-line administration
live network traffic inspection
JSON alert logging
basic alert tuning and suppression
testing and validation from a second machine
technical documentation of a cybersecurity lab project
