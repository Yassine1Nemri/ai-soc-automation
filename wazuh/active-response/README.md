# Wazuh Active Response

Wazuh Active Response is intended for controlled containment
actions following security alerts.

In this project, response actions require explicit analyst
approval through the Shuffle and DFIR-IRIS workflow.

## Planned actions

- Block a suspicious source IP.
- Disable a confirmed compromised local account.

## Safety

No response action should execute automatically based solely
on the detection of a brute-force attempt.

The response must target the correct agent and be logged for
incident investigation.
