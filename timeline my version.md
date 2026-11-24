# Investigation Timeline
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

