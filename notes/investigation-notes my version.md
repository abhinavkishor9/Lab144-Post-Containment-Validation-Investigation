# Investigation Notes
## 1. Investigation Objective

The objective was to demonstrate how a SOC analyst validates an endpoint after containment actions.

The investigation focused on:

- Establishing an endpoint baseline.
- Simulating a controlled suspicious process.
- Applying a narrowly scoped firewall containment rule.
- Terminating the controlled process.
- Verifying the process termination.
- Inspecting potential residual activity.
- Recording post-containment results.
- Removing the temporary firewall rule.

## 2. Lab Environment

| Attribute | Value |
|---|---|
| Hostname | DESKTOP-9MMM37V |
| Operating System | Windows 11 Pro |
| PowerShell | 7.7.0 |
| Sysmon | Installed |
| Wazuh Agent | 001 |
| Wazuh Manager | 4.13.1 |
| Evidence ID | EVD-144-001 |

## 3. Workspace Preparation

Created:

`C:\DFIR-Lab144`

Subdirectories:

- baseline
- simulation
- containment
- validation
- evidence
- logs

All six directories were successfully created.

## 4. Baseline Collection

The initial endpoint state was captured before the simulation.

### System Baseline

| Attribute | Recorded Value |
|---|---|
| Hostname | DESKTOP-9MMM37V |
| User | DESKTOP-9MMM37V\Dell |
| Capture time | 2026-11-24 09:50:55 +05:30 |
| PowerShell | 7.7.0 |
| Domain firewall | Enabled |
| Private firewall | Enabled |
| Public firewall | Enabled |

The baseline was exported to:

`C:\DFIR-Lab144\baseline\system-baseline.csv`

### Process Baseline

The running-process inventory was collected using `Get-Process` and exported to:

`C:\DFIR-Lab144\baseline\process-baseline.csv`

### Network Baseline

The active TCP connection inventory was collected using `Get-NetTCPConnection` and exported to:

`C:\DFIR-Lab144\baseline\network-baseline.csv`

These files provide an initial reference for comparing endpoint activity.

## 5. Controlled Process Simulation

Created:

`C:\DFIR-Lab144\simulation\lab144-test.ps1`

The script contained a harmless infinite loop with a five-second sleep interval.

The script did not perform network communication, modify security settings, or execute malicious actions.

### Process Details

| Attribute | Value |
|---|---|
| Process name | pwsh |
| Process ID | 600 |
| Start time | 2026-11-24 09:52:43 |
| Test script | C:\DFIR-Lab144\simulation\lab144-test.ps1 |

The process was confirmed running through `Get-Process`.

## 6. Firewall Containment Simulation

Before creating the rule, the active firewall policy was checked for an existing enabled rule with the same display name.

No matching rule was returned.

A temporary outbound block rule was created:

`Lab144-Containment-Test`

### Rule Configuration

| Attribute | Value |
|---|---|
| Display name | Lab144-Containment-Test |
| Direction | Outbound |
| Action | Block |
| Enabled | True |
| Profile | Any |
| Remote address | 192.0.2.10 |
| Policy store source | PersistentStore |
| Primary status | OK |

The address `192.0.2.10` is reserved for documentation and testing.

The firewall rule was successfully created and its configuration was independently inspected using:

- `Get-NetFirewallRule`
- `Get-NetFirewallAddressFilter`

The configured remote address was confirmed as `192.0.2.10`.

**Interpretation:** The rule configuration was verified. An actual network connection attempt was not performed, so successful traffic blocking was not independently demonstrated.

## 7. Process Containment

The controlled test process was terminated using:

`Stop-Process -Id 600 -Force`

A subsequent lookup using `Get-Process` returned no process information for PID 600.

### Process Validation

| Attribute | Result |
|---|---|
| Test process ID | 600 |
| Process found after termination | No |
| Still running | False |

This confirmed that the specific controlled process was no longer running.

It does not establish that every potentially suspicious process on the endpoint was stopped.

## 8. Post-Containment Validation

### A. Process Validation

The process ID was checked again.

Result:

`StillRunning = False`

### B. Firewall Rule Validation

The rule was queried again.

Result:

| Attribute | Result |
|---|---|
| Rule found | Yes |
| Enabled | True |
| Direction | Outbound |
| Action | Block |

### C. Address Filter Validation

The associated firewall address filter returned:

`192.0.2.10`

This confirmed that the intended destination was configured in the rule.

### D. Network Connection Inspection

`Get-NetTCPConnection` was used to inspect current TCP connections.

The output included:

- Listening sockets.
- Bound sockets.
- Established localhost connections.
- Established external HTTPS connections.
- The established Wazuh agent connection to `192.168.174.128:1514`.

No connection to `192.0.2.10` appeared in the supplied output.

The absence of a matching connection is not proof that a connection attempt was blocked because no controlled connection attempt was made.

### E. Process Command-Line Inspection

The following query was used to search for a process referencing the test script:

`Get-CimInstance Win32_Process`

The result contained no matching process.

This provided an additional check that the test script was no longer running.

## 9. Residual Activity Inspection

### Scheduled Tasks

The scheduled task inventory was exported to:

`C:\DFIR-Lab144\validation\scheduled-tasks.csv`

### Startup Entries

The startup command inventory was exported to:

`C:\DFIR-Lab144\validation\startup-entries.csv`

These inventories were collected for review.

The supplied results do not establish whether every entry was benign or whether persistence was absent.

### Test Artifact

The test script was checked after process termination.

| Attribute | Value |
|---|---|
| Path | C:\DFIR-Lab144\simulation\lab144-test.ps1 |
| Length | 91 bytes |
| Creation time | 2026-11-24 09:52:34 |
| Last write time | 2026-11-24 09:52:34 |

The artifact remained present.

This demonstrates that stopping a process does not automatically remove its associated file.

## 10. Post-Containment Summary

A validation summary was generated with evidence ID:

`EVD-144-001`

### Recorded Results

| Field | Result |
|---|---|
| Hostname | DESKTOP-9MMM37V |
| Validation time | 2026-11-24 10:05:48 +05:30 |
| Test process ID | 600 |
| Process stopped | True |
| Firewall rule found | True |
| Test artifact exists | True |

The summary was exported to:

`C:\DFIR-Lab144\validation\post-containment-summary.csv`

## 11. Firewall Rule Cleanup

After validation, the temporary rule was removed using:

`Remove-NetFirewallRule -DisplayName "Lab144-Containment-Test"`

A subsequent lookup returned no matching rule.

The cleanup was successful.

The test script and investigation records were retained in the lab workspace.

## 12. Findings

### Confirmed

- The investigation workspace was created.
- Initial system, process, and network snapshots were collected.
- The controlled PowerShell process was started successfully.
- The temporary outbound firewall rule was created.
- The rule's enabled state, direction, action, and remote address were verified.
- The controlled process was terminated.
- The post-containment process check returned no matching process.
- No process matching the test script command line was returned.
- Scheduled task and startup inventories were exported.
- The test artifact remained present.
- The post-containment summary was exported.
- The temporary firewall rule was removed successfully.

### Not Established

- Whether an actual connection to `192.0.2.10` would have been blocked during a connection attempt.
- Whether all suspicious activity on the endpoint was absent.
- Whether all scheduled tasks and startup entries were benign.
- Whether Sysmon events for this exercise were ingested into Wazuh.
- Whether the endpoint was free of persistence or other compromise.
- Whether the endpoint was ready for production recovery.

