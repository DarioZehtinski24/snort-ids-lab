# Snort Detection Lab – Results Summary

## Overview
This document summarizes the validated results of the Snort detection lab.

## Successfully validated detections
The following detection scenarios were successfully validated in the lab environment:

- Repeated SSH connection attempts
- SMB probe detection
- RDP probe detection
- HTTP probing for the `/admin` URI
- Built-in TCP port scan detection

## Observed custom alerts
The following custom alert messages were configured and successfully triggered during testing:

- `LOCAL Repeated SSH Connection Attempts`
- `LOCAL SMB Probe Detected`
- `LOCAL RDP Probe Detected`
- `LOCAL Web Admin Probe`

## Observed built-in alert
The built-in Snort port scan inspector successfully generated:

- `(port_scan) TCP portscan`

## Logging results
Alerts were successfully observed in two forms:
- terminal output with readable alert messages
- JSON evidence files containing selected alert fields

The JSON format included:
- timestamp
- protocol
- source address and port
- destination address and port
- rule identifier
- alert message
- action

## Evidence collection
The project contains evidence in the following form:
- Snort configuration file
- local custom rules
- documentation files
- JSON alert output files

## Main outcome
The project successfully demonstrated that Snort can be configured on Kali Linux to detect both custom-defined network events and built-in scan-related behavior in a controlled lab environment.

## Technical outcome
The project confirmed the following:
- Snort was installed and validated successfully
- custom local rules were loaded correctly
- the built-in port scan inspector was operational
- JSON logging was functional
- alerts could be generated from a second machine in the same network
- the detection lab was documented in a structured and reproducible way
