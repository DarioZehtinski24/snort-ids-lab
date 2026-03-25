\# Snort Detection Lab on Kali Linux

## Overview
This project demonstrates the deployment and configuration of Snort on Kali Linux as an intrusion detection system in a small lab environment. The lab was extended with multiple custom detection rules and built-in port scan detection in order to simulate realistic network monitoring scenarios.

## Objective
The objective of this project was to:
- install and validate Snort on Kali Linux
- configure custom local detection rules
- monitor live network traffic
- detect suspicious network activity from a second machine
- log alerts in JSON format for documentation and evidence collection

## Detection Use Cases
The lab includes the following detection scenarios:
- repeated SSH connection attempts to port 22
- SMB probe detection on port 445
- RDP probe detection on port 3389
- HTTP probing for the `/admin` URI
- built-in TCP port scan detection

## Custom Rules
The custom rules used in this project are stored in:

rules/local.rules

The main custom detection logic is:


SSH rule: detects repeated TCP SYN attempts to port 22 from the same source
SMB rule: detects TCP SYN probes to port 445
RDP rule: detects TCP SYN probes to port 3389
HTTP rule: detects requests containing /admin
Port Scan Detection

In addition to custom rules, the project uses Snort’s built-in port_scan inspector to detect TCP port scanning activity. This was tested with Nmap from a second machine.

Logging and Evidence

Snort alerts were collected in JSON format for easier documentation and validation. The project contains:

configuration files
custom rules
notes and documentation
evidence files with alert outputs
Project Structure

snort-ids-lab/
├── README.md
├── configs/
│   └── snort.lua
├── rules/
│   └── local.rules
├── notes/
│   ├── project_notes.txt
│   ├── results_summary.md
│   └── setup_and_execution.md
└── evidence/
    └── json/
        └── evidence_explanation.md

Test Environment
Sensor machine: Kali Linux with Snort installed
Traffic generator: second machine in the same network
Monitored interface: eth0
Validation Method

The rules and detection logic were validated by generating controlled traffic from a second machine:

repeated connection attempts to SSH
probes to SMB and RDP ports
HTTP request to /admin
TCP port scan with Nmap
Result

The project successfully demonstrated a working Snort-based detection lab. Custom rules and built-in port scan detection were validated in a controlled environment, and alerts were recorded both in terminal output and JSON log files.

Skills Demonstrated
Snort installation and configuration
custom IDS rule development
Linux command-line usage
live traffic inspection
JSON alert logging
basic network threat detection
technical documentation of a cybersecurity lab project
