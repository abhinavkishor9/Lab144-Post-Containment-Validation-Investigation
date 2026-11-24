# Lab144-Post-Containment-Validation-Investigation
## Overview
### 1. What Is Post-Containment Validation?

Post-containment validation is the process of checking whether incident response containment actions have achieved their intended purpose.

During a security incident, an organization may isolate an endpoint, disable a compromised account, block a suspicious IP address, or terminate a malicious process.

However, executing a containment action does not automatically prove that the threat has been contained.

The SOC analyst must collect evidence to verify the outcome and identify any remaining suspicious activity.

### 2. Why Is Validation Necessary?

Consider an endpoint where a suspicious PowerShell process has been identified.

The response team terminates the process and blocks a suspected command-and-control destination.

The analyst must determine:

- Did the process actually stop?
- Was the firewall rule applied correctly?
- Is there still suspicious network activity?
- Are other related processes running?
- Does persistence remain?
- Are new suspicious files being created?
- Is endpoint telemetry still being collected?

Without these checks, the organization may incorrectly assume that the incident is contained.

### 3. Containment, Eradication, and Recovery

These are related but distinct incident response activities.

| Phase | Purpose | Example |
|---|---|---|
| Containment | Limit incident impact and spread | Isolate an affected endpoint |
| Eradication | Remove the threat and its persistence | Remove confirmed malicious components |
| Recovery | Restore normal operations | Reconnect a validated endpoint |
| Post-containment validation | Verify containment and investigate remaining activity | Inspect processes, connections, and persistence |

Post-containment validation supports decisions about eradication and recovery. It does not replace either phase.

### 4. Establishing an Endpoint Baseline

A baseline is a record of the endpoint's state before or at the beginning of an investigation.

It may include:

- Running processes.
- Active network connections.
- Firewall configuration.
- Logged-in users.
- Scheduled tasks.
- Startup entries.
- Relevant security events.

The baseline provides a reference for later comparison.

Without a baseline, analysts may misinterpret normal endpoint activity or overlook changes introduced during an incident.

### 5. Process Containment

A suspicious process may be terminated as part of containment.

The analyst should verify:

1. The correct process was identified.
2. Its process ID and command line were recorded.
3. The termination action succeeded.
4. The process is no longer running.
5. Related or child processes are not continuing the activity.

A process stopping does not prove that the associated threat has been eradicated.

Persistence mechanisms or other components may still exist.

### 6. Network Containment

Network containment can include:

- Blocking a suspicious destination IP.
- Blocking a malicious domain through an appropriate control.
- Restricting outbound communication.
- Isolating an endpoint from the network.
- Applying network access controls.

When a firewall rule is used, the analyst should verify:

- The rule exists.
- The rule is enabled.
- The direction is correct.
- The action is correct.
- The destination or application scope is correct.
- The rule is being enforced by the intended firewall policy.

**Important:** Verifying rule configuration is not the same as proving that a connection was successfully blocked. A controlled connection test or suitable firewall telemetry is needed to demonstrate the actual enforcement outcome.

### 7. Residual Activity Investigation

After containment, analysts should look for signs that the incident may still be active.

Common areas include:

#### Processes

Inspect running processes, command lines, parent-child relationships, and suspicious execution paths.

#### Network Connections

Review active connections and identify unexpected remote addresses, ports, and owning processes.

#### Scheduled Tasks

Investigate unexpected scheduled tasks that could be used to maintain persistence.

#### Startup Entries

Review startup commands and autorun locations for suspicious execution.

#### Files

Inspect suspicious files, their paths, timestamps, hashes, and associated creation or modification events.

No single finding should be treated as proof of compromise without supporting context.

### 8. Endpoint Telemetry and SIEM Correlation

Sysmon and Wazuh can help validate endpoint activity.

Relevant Sysmon event IDs include:

| Event ID | Description |
|---|---|
| 1 | Process creation |
| 3 | Network connection |
| 5 | Process termination |
| 11 | File creation |

Event availability depends on Sysmon configuration and the endpoint collection pipeline.

Wazuh provides centralized access to collected endpoint telemetry.

The analyst should correlate timestamps, process IDs, file paths, host identity, and event details.

A missing event does not automatically establish that an activity did not occur.

### 9. Evidence-Based Validation

Post-containment findings should be separated into three categories.

#### Confirmed

Directly supported by available evidence.

Example: A process lookup after termination returned no process for the recorded process ID.

#### Plausible

Consistent with the evidence but not fully demonstrated.

Example: A configured firewall block rule may prevent traffic to its specified destination.

#### Unknown

Not established by the available evidence.

Example: Whether all persistence mechanisms have been removed.

This distinction prevents analysts from overstating the success of containment.

### 10. Recovery Readiness

Before an affected endpoint is returned to normal operation, the response team should consider:

- Whether containment remains effective.
- Whether the threat has been eradicated.
- Whether persistence has been investigated.
- Whether security controls are functioning.
- Whether endpoint telemetry is available.
- Whether required patches or configuration changes have been applied.
- Whether the recovery action has been authorized.

A single successful process termination is not sufficient to declare an endpoint clean.

### 11. Key Takeaways

- Containment must be validated after it is applied.
- Baselines provide useful comparison points.
- Process termination does not establish eradication.
- Firewall configuration does not independently prove traffic blocking.
- Residual processes and persistence must be investigated.
- Sysmon and Wazuh provide useful supporting telemetry.
- Missing telemetry should be treated as a potential visibility gap.
- Findings must distinguish confirmed, plausible, and unknown conditions.

**Core principle:** Validate the response action, investigate what remains, and base recovery decisions on evidence.

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

## Lab Objectives

### Primary Objective

Develop the ability to assess an endpoint after containment by verifying response actions, inspecting residual activity, and documenting whether the available evidence supports the intended containment outcome.

### Technical Objectives

#### 1. Endpoint Baseline Collection

- Capture basic endpoint identity and configuration.
- Record firewall profile states.
- Export the running-process inventory.
- Export active TCP connections.
- Understand why baseline information is important during incident response.

#### 2. Controlled Process Investigation

- Create a harmless PowerShell test script.
- Launch the script as a separate process.
- Identify its process ID and start time.
- Verify that the process is running before containment.

#### 3. Network Containment Configuration

- Create a narrowly scoped outbound firewall block rule.
- Use a reserved documentation IP address as the simulated destination.
- Inspect the rule's enabled state, direction, action, and profile.
- Verify the associated remote address filter.
- Distinguish rule configuration from actual traffic-blocking validation.

#### 4. Process Containment Validation

- Terminate the controlled test process.
- Verify that the recorded process ID is no longer running.
- Search process command lines for the test script.
- Understand the limitations of process termination as an eradication measure.

#### 5. Residual Activity Inspection

- Inspect current network connections.
- Collect scheduled task information.
- Collect startup command information.
- Verify whether the controlled test artifact remains present.
- Understand why suspicious-looking entries require contextual investigation.

#### 6. Validation Documentation

- Generate a post-containment summary.
- Record the validation timestamp and endpoint identity.
- Document process and firewall validation results.
- Preserve baseline and post-containment records.
- Record limitations and unverified findings.

#### 7. Containment Cleanup

- Remove the temporary lab-specific firewall rule.
- Verify that the rule is no longer present.
- Avoid modifying unrelated firewall settings.

### Learning Outcomes

After completing this lab, the analyst should be able to:

- Explain the purpose of post-containment validation.
- Differentiate containment from eradication and recovery.
- Establish and use an endpoint baseline.
- Verify process termination using process identifiers.
- Inspect firewall rule configuration.
- Explain the difference between a configured block rule and a confirmed blocked connection.
- Investigate common locations associated with persistence.
- Recognize limitations in endpoint telemetry.
- Document findings without making unsupported claims.

### Success Criteria

The exercise is considered successful when:

- The endpoint baseline is collected.
- A controlled test process is started and identified.
- The temporary firewall rule is created and inspected.
- The controlled process is terminated.
- Subsequent checks confirm that the test process is no longer running.
- Residual activity inventories are collected.
- A post-containment summary is generated.
- The temporary firewall rule is removed and its absence verified.

### Investigation Quality Criteria

The analyst must also distinguish between:

- Confirmed containment results.
- Configurations that were verified but not tested through actual traffic.
- Activities that were not investigated.
- Security conclusions that cannot be established from the available evidence.

## Lab Scenario

### Scenario: Verifying Endpoint Security After Containment

A SOC team identifies suspicious PowerShell activity on a Windows endpoint during an incident investigation.

The response team decides to contain the activity by stopping the identified process and applying a network restriction against a suspected destination.

Once the containment actions are applied, the SOC analyst is assigned to verify whether the endpoint's state reflects the intended response.

The analyst must not assume that the incident is resolved simply because the process has been stopped or a firewall rule has been created.

### Investigation Context

The exercise uses a Windows 11 endpoint monitored through Sysmon and Wazuh.

A harmless PowerShell script is introduced to represent a controlled suspicious process. The analyst records its process ID and execution details before applying containment.

A temporary outbound firewall rule is configured for a reserved documentation IP address. The rule is intended to represent a network restriction that might be applied during an actual incident.

The exercise does not involve real malware, external attacker infrastructure, or deliberate disruption of the endpoint's normal network connectivity.

### Investigation Tasks

The analyst is expected to:

1. Capture the endpoint's initial system configuration.
2. Record running processes and active network connections.
3. Create and launch a controlled PowerShell process.
4. Record its process ID and execution details.
5. Apply a narrowly scoped outbound firewall block rule.
6. Verify that the firewall rule has the intended configuration.
7. Terminate the controlled process.
8. Confirm that the process is no longer running.
9. Inspect active connections and search for remaining instances of the test script.
10. Collect scheduled task and startup command inventories.
11. Verify whether the test artifact remains on the endpoint.
12. Generate a post-containment validation record.
13. Remove the temporary firewall rule and verify cleanup.

### Investigation Considerations

The analyst must distinguish between verifying that a security control is configured and demonstrating that it successfully enforced the intended restriction.

For example, an enabled outbound firewall block rule confirms its configuration. To demonstrate actual traffic blocking, an appropriate controlled connection attempt or firewall telemetry would also be required.

Similarly, terminating one process does not prove that all malicious activity has stopped. Other processes, persistence mechanisms, or compromised accounts may require separate investigation.

Endpoint telemetry should be correlated with the available evidence rather than treated as a complete record of every action.

### Expected Outcome

The analyst should be able to demonstrate that the controlled process was successfully terminated, the firewall rule was configured as intended, and post-containment checks were performed.

The investigation should also identify what remains unverified, including actual network-block enforcement, persistence analysis, and any missing endpoint telemetry.

The final assessment must be based on the evidence collected and should not declare the endpoint clean or ready for production recovery solely because the controlled process stopped.

**Investigation principle:** Containment is an action. Validation is the evidence-based process of determining whether that action achieved its intended result.

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

