# Lab 144 – Investigation Timeline

**Date:** 24 November 2026  
**Host:** DESKTOP-9MMM37V  
**Evidence ID:** EVD-144-001  
**Timezone:** IST (UTC+05:30)

| Time (IST) | Activity | Result |
|---|---|---|
| 09:50 | Created Lab 144 workspace and subdirectories | Success |
| 09:50:55 | Captured initial system and firewall baseline | Success |
| 09:50 onward | Exported process and network snapshots | Success |
| 09:52:34 | Created controlled PowerShell test script | Success |
| 09:52:43 | Started separate PowerShell test process | PID 600 |
| 09:52 onward | Confirmed the test process was running | Confirmed |
| After process start | Checked for an existing enabled lab firewall rule | No matching rule found |
| After rule check | Created outbound firewall block rule | Success |
| After rule creation | Verified rule configuration and remote address | Confirmed |
| After rule verification | Terminated controlled test process | Success |
| After termination | Checked process ID 600 | No matching process |
| Post-containment | Rechecked firewall rule configuration | Enabled and configured |
| Post-containment | Inspected active TCP connections | No connection to 192.0.2.10 observed |
| Post-containment | Searched for the test script in process command lines | No matching process |
| Post-containment | Exported scheduled task inventory | Success |
| Post-containment | Exported startup command inventory | Success |
| 10:05:48 | Generated post-containment validation summary | Success |
| After validation | Removed temporary firewall rule | Success |
| Final check | Queried for the lab firewall rule | No matching rule |

## Key Investigation Events

### 1. Baseline Established

The endpoint's system configuration, running processes, and active TCP connections were captured before the controlled simulation.

### 2. Controlled Activity Introduced

A harmless PowerShell script was launched as a separate process with PID 600.

### 3. Containment Configuration Applied

A temporary outbound firewall block rule was configured for the reserved address `192.0.2.10`.

The rule's configuration was verified.

### 4. Process Terminated

The controlled process was stopped and subsequent process checks returned no matching process.

### 5. Post-Containment Inspection

The analyst inspected network connections, process command lines, scheduled tasks, startup entries, and the test artifact.

The test artifact remained present.

### 6. Validation and Cleanup

A post-containment summary was exported.

The temporary firewall rule was removed and its absence was verified.

## Final Outcome

The controlled process termination was confirmed.

The firewall rule configuration was verified before cleanup.

The investigation records were generated successfully.

An actual blocked network connection and corresponding Sysmon/Wazuh telemetry were not established by the available results.

**Conclusion:** The lab demonstrated successful controlled process containment and configuration validation, with clearly documented limits on the broader security conclusions.
