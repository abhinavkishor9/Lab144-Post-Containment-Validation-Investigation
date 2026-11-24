# Troubleshooting Notes
## 1. Firewall Rule Creation

### Observation

The temporary firewall rule was created successfully.

Rule name:

`Lab144-Containment-Test`

The command returned:

- PrimaryStatus: OK
- Status: The rule was parsed successfully from the store.
- Enabled: True
- Direction: Outbound
- Action: Block

### Interpretation

Windows Firewall accepted the rule configuration.

The rule was stored in the local PersistentStore.

The `EnforcementStatus` field returned `NotApplicable` in the creation output.

This field should not be interpreted on its own as proof that the firewall rule failed. The subsequent rule inspection confirmed that the rule existed and was enabled.

### Validation

The rule was inspected using:

`Get-NetFirewallRule`

The remote address was independently checked using:

`Get-NetFirewallAddressFilter`

Both checks returned the expected configuration.

## 2. Network Connection Output

### Observation

The network connection inventory contained numerous entries, including:

- Bound sockets.
- Listening sockets.
- Established localhost connections.
- Established HTTPS connections.
- The Wazuh agent connection to the manager.

### Interpretation

A large number of active connections is not automatically suspicious.

The analyst should examine:

- Remote address.
- Remote port.
- Connection state.
- Owning process.
- Process identity.
- Expected endpoint activity.
- Relevant baseline differences.

The established Wazuh connection to `192.168.174.128:1514` was consistent with the configured agent-to-manager communication.

No connection to the reserved test address `192.0.2.10` appeared in the supplied output.

### Limitation

No actual connection attempt to the test address was made.

Therefore, the network snapshot cannot establish that the firewall successfully blocked a connection.

## 3. Process Termination Validation

### Observation

After running:

`Stop-Process -Id 600 -Force`

The subsequent `Get-Process` lookup returned no process information.

The command-line search for `lab144-test.ps1` also returned no matching process.

### Interpretation

The controlled test process was no longer running.

This confirms termination of the identified process, not eradication of a real threat.

## 4. Test Artifact Remained Present

### Observation

The file:

`C:\DFIR-Lab144\simulation\lab144-test.ps1`

remained present after the process was terminated.

### Interpretation

Process termination and file removal are separate operations.

The script was deliberately retained as investigation evidence.

No deletion was necessary for this lab.

## 5. Firewall Rule Cleanup

### Observation

The temporary firewall rule was removed successfully.

A subsequent lookup returned no matching rule.

### Interpretation

The lab-specific firewall rule was no longer present.

No other firewall rules were intentionally removed or modified.

## 6. Telemetry Validation Limitation

### Observation

The final results supplied for this lab did not include corresponding Sysmon or Wazuh event records.

### Interpretation

The PowerShell process and file activity were performed, but the available evidence does not establish whether the corresponding Sysmon events were collected and ingested into Wazuh.

Relevant event IDs for a future telemetry review include:

| Event ID | Description |
|---|---|
| 1 | Process creation |
| 3 | Network connection, when configured |
| 5 | Process termination |
| 11 | File creation |

Event availability depends on the active Sysmon configuration and Wazuh collection pipeline.

A missing event should be investigated rather than treated as proof that the underlying activity did not occur.

## 7. Baseline and Post-Containment Comparison

### Observation

The lab collected initial system, process, and network snapshots.

It also collected scheduled task and startup command inventories after containment.

### Interpretation

The snapshots provide useful investigation records, but they are not a complete before-and-after comparison of every endpoint component.

The scheduled task and startup inventories were exported, but their entries were not individually analyzed.

Therefore, the lab does not establish that persistence was absent.

