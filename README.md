# Lab 144 – Post-Containment Validation Investigation

## Overview

This lab demonstrates how a SOC analyst validates an endpoint after containment actions have been applied.

The exercise uses a controlled PowerShell process and a temporary Windows Firewall rule to simulate selected incident response activities. The objective is to verify whether the test process has stopped, confirm the firewall rule configuration, inspect potential residual activity, and document the endpoint's post-containment state.

The investigation was performed on a Windows 11 endpoint using PowerShell 7.7.0.

A baseline of system, process, and network information was collected before the simulation. A harmless PowerShell script was then launched as a separate process. A narrowly scoped outbound firewall rule was created for a reserved documentation IP address, and the controlled process was terminated.

Post-containment checks confirmed that the test process was no longer running, the firewall rule was enabled with the intended blocking configuration, and the test artifact remained present. The temporary firewall rule was subsequently removed.

## Lab Environment

| Component | Details |
|---|---|
| Operating System | Windows 11 Pro |
| Hostname | DESKTOP-9MMM37V |
| PowerShell | 7.7.0 |
| Endpoint Monitoring | Sysmon |
| SIEM | Wazuh 4.13.1 |
| Wazuh Agent ID | 001 |
| Evidence ID | EVD-144-001 |
| Test Process ID | 600 |
| Simulated Destination | 192.0.2.10 |

## Repository Structure

The investigation workspace was created at:

`C:\DFIR-Lab144`

Directories:

- `baseline` – Initial endpoint, process, and network snapshots.
- `simulation` – Controlled PowerShell test script.
- `containment` – Reserved for containment-related records.
- `validation` – Post-containment inspection results.
- `evidence` – Reserved for supporting evidence.
- `logs` – Reserved for investigation logs.

## Investigation Workflow

1. Prepared the investigation workspace.
2. Captured the initial endpoint configuration.
3. Exported running-process and network-connection snapshots.
4. Created a harmless PowerShell test script.
5. Started the script as a separate PowerShell process.
6. Created a temporary outbound firewall block rule for `192.0.2.10`.
7. Verified the firewall rule configuration.
8. Terminated the controlled test process.
9. Confirmed that the process was no longer running.
10. Inspected network connections, scheduled tasks, startup entries, and the test artifact.
11. Generated a post-containment validation summary.
12. Removed the temporary firewall rule.

## Key Findings

### 1. Baseline Collection

The initial endpoint baseline recorded:

- Hostname: DESKTOP-9MMM37V
- User: DESKTOP-9MMM37V\Dell
- PowerShell: 7.7.0
- Domain firewall profile: Enabled
- Private firewall profile: Enabled
- Public firewall profile: Enabled

Process and network snapshots were exported successfully.

### 2. Controlled Process Simulation

A harmless script was created:

`C:\DFIR-Lab144\simulation\lab144-test.ps1`

The script contained a simple loop with a five-second sleep interval. It was launched as a separate PowerShell process.

| Attribute | Value |
|---|---|
| Process name | pwsh |
| Process ID | 600 |
| Start time | 2026-11-24 09:52:43 |

### 3. Firewall Containment Simulation

A temporary outbound firewall rule was created:

`Lab144-Containment-Test`

Configuration:

| Attribute | Value |
|---|---|
| Direction | Outbound |
| Action | Block |
| Enabled | True |
| Profile | Any |
| Remote address | 192.0.2.10 |

The rule was successfully created and its configuration was verified.

The destination is a reserved documentation address. No real attacker infrastructure was contacted.

### 4. Process Containment

The controlled process with PID 600 was terminated.

A subsequent process lookup returned no matching process.

The post-containment check recorded:

`ProcessStopped = True`

### 5. Residual Activity Inspection

The investigation included:

- Active TCP connection inspection.
- Process lookup using the test script command line.
- Scheduled task inventory.
- Startup command inventory.
- Test artifact verification.

No process matching `lab144-test.ps1` was returned by the command-line inspection.

The test artifact remained present after process termination.

### 6. Post-Containment Summary

The recorded validation results were:

| Check | Result |
|---|---|
| Process stopped | True |
| Firewall rule found during validation | True |
| Test artifact exists | True |

The results were exported to:

`C:\DFIR-Lab144\validation\post-containment-summary.csv`

### 7. Firewall Rule Cleanup

After validation, the temporary firewall rule was removed.

A subsequent lookup returned no matching rule.

No other firewall rules were intentionally modified.

## Investigation Limitations

- The test process was a harmless controlled PowerShell process, not malware.
- The firewall rule configuration was verified, but an actual connection attempt to the reserved destination was not performed.
- No matching connection to `192.0.2.10` was observed in the supplied network snapshot.
- Scheduled task and startup command inventories were collected, but the results were not individually analyzed for malicious persistence.
- Sysmon and Wazuh telemetry was not supplied as part of the final validation results.
- Process termination does not establish eradication of a real threat.
- The investigation does not establish that the entire endpoint is clean or ready for production recovery.

## Key SOC Takeaways

- Containment actions must be validated rather than assumed successful.
- A stopped process does not prove that persistence has been removed.
- Firewall rule configuration and actual traffic blocking are separate validation points.
- Baseline comparisons help identify changes in endpoint state.
- Missing telemetry does not automatically mean that no activity occurred.
- Evidence should distinguish confirmed findings from unverified assumptions.

## Conclusion

This lab demonstrated a controlled post-containment investigation using PowerShell and Windows Firewall.

The test process was successfully terminated, the temporary firewall rule was verified with the intended configuration, and the test artifact remained available for inspection.

The firewall rule was removed after the validation exercise.

The investigation reinforces an important incident response principle:

**Never assume containment succeeded just because a response action was executed. Validate its outcome and document what the available evidence actually proves.**
