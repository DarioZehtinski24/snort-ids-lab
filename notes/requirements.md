# Snort Detection Lab – Requirements

## Purpose
This file describes the software, network, and functional requirements for reproducing the Snort detection lab project.

## 1. System Requirements

### Sensor machine
The sensor machine must meet the following requirements:
- Kali Linux installed and running
- Snort 3 installed and operational
- administrative (`sudo`) access
- active monitored network interface, such as `eth0`

### Test machine
A second machine is required to generate validation traffic. It may be:
- Kali Linux
- another Linux system
- Windows system with equivalent networking tools

The second machine must be reachable on the same network as the Snort sensor.

## 2. Software Requirements

### Required software on the Snort sensor
- Kali Linux
- Snort 3
- standard Linux terminal utilities
- Nano or another text editor

### Required software on the test machine
- Netcat (`nc`) for TCP probe generation
- Nmap for port scanning tests
- Curl for HTTP request testing

## 3. Network Requirements
The lab requires:
- two machines on the same reachable network
- IP connectivity between the sensor and the test machine
- a known IP address for the monitored host
- live traffic visible on the monitored interface

Example monitored host:
- `192.168.0.101`

Example test machine:
- `192.168.0.147`

## 4. Functional Requirements
The project must support the following functions:
- loading custom Snort rules from `local.rules`
- validating Snort configuration before execution
- monitoring live traffic on the selected interface
- generating alerts for custom rules
- generating alerts for built-in TCP port scan detection
- writing alert output in JSON format
- storing evidence and documentation in the project folder

## 5. Detection Requirements
The lab must detect the following scenarios:

### 5.1 Repeated SSH connection attempts
- destination port: `22`
- protocol: TCP
- logic: repeated SYN attempts from the same source within a defined time window

### 5.2 SMB probe detection
- destination port: `445`
- protocol: TCP
- logic: SYN-based probe to SMB port

### 5.3 RDP probe detection
- destination port: `3389`
- protocol: TCP
- logic: SYN-based probe to RDP port

### 5.4 HTTP `/admin` probing
- protocol: HTTP
- logic: request containing `/admin` in the URI

### 5.5 TCP port scan detection
- built-in Snort `port_scan` inspector
- logic: scanning activity directed toward the monitored host

## 6. Configuration Requirements
The project requires:
- a valid `snort.lua` configuration file
- a valid `local.rules` file
- local rules included in the `ips` section of `snort.lua`
- built-in rules enabled if port scan alerting is required
- optional suppression of noisy built-in alerts for cleaner output

## 7. Logging Requirements
The project requires:
- terminal-based alert visibility during testing
- JSON alert logging for evidence collection
- evidence stored inside the project folder

Recommended JSON fields:
- `timestamp`
- `proto`
- `src_ap`
- `dst_ap`
- `rule`
- `msg`
- `action`

## 8. Documentation Requirements
The project documentation should include:
- project overview
- setup summary
- results summary
- requirements description
- evidence explanation
- configuration and rule files

## 9. Expected Outcome
A successful implementation of the project should:
- validate Snort configuration successfully
- detect the defined SSH, SMB, RDP, and HTTP scenarios
- detect TCP port scan activity
- generate evidence in JSON format
- provide a reproducible cybersecurity lab project suitable for GitHub and CV use
