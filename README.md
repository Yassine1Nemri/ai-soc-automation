# AI-Driven SOC Automation

An intelligent Security Operations Center (SOC) automation
project integrating security monitoring, AI-assisted alert
analysis, incident management, and security orchestration.

## Architecture

The project integrates four main components:

- **Wazuh:** Security monitoring and SSH brute-force detection.
- **Ollama (Qwen3:4B):** Local AI-assisted security alert analysis.
- **Shuffle:** Security workflow orchestration and automation.
- **DFIR-IRIS:** Security alert management and incident escalation.

## Workflows

### 1. Detection and Alert Creation

Wazuh detects SSH brute-force activity and forwards the alert
for AI-assisted analysis. Shuffle orchestrates the analysis,
creates an alert in DFIR-IRIS, and sends an email notification.

### 2. Alert Triage and Escalation

An IRIS webhook notifies Shuffle when an analyst changes an
alert's status.

- **In Progress:** Escalate the alert and create an incident case.
- **Closed:** Record a comment explaining the rejection.

### 3. Human-Approved Incident Response

The project is being extended to support analyst-approved
containment actions before execution.

No response action should be executed without explicit
authorization from the analyst.

## Technologies

- Wazuh
- Ollama
- Qwen3:4B
- Shuffle SOAR
- DFIR-IRIS
- Linux
- REST APIs
- Python

## Project Status

The detection, AI-assisted analysis, alert creation, email
notification, and alert escalation workflows are operational.

Human-approved response is under development.

## Security

Credentials, API tokens, private keys, and sensitive
infrastructure information must not be committed to Git.

## Author

Yassine Nemri
