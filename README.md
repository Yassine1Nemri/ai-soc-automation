<div align="center">

# 🛡️ AI-Driven SOC Automation

### AI-Assisted Security Monitoring, SOAR & Incident Management

A practical SOC automation project integrating Wazuh, local AI, Shuffle SOAR, and DFIR-IRIS to streamline security alert analysis and incident handling.

**Detection · AI Analysis · Orchestration · Incident Response**

<br>

![Linux](https://img.shields.io/badge/Platform-Linux-FCC624?style=flat-square&logo=linux&logoColor=black)
![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-005571?style=flat-square)
![Ollama](https://img.shields.io/badge/AI-Ollama-black?style=flat-square)
![Shuffle](https://img.shields.io/badge/SOAR-Shuffle-6C47FF?style=flat-square)
![DFIR-IRIS](https://img.shields.io/badge/Incident%20Management-DFIR--IRIS-167D9A?style=flat-square)

</div>

---

## 📌 Overview

This project demonstrates the integration of security monitoring, local AI-assisted analysis, security orchestration, and incident management in a practical SOC environment.

The objective is to automate repetitive security operations while keeping the analyst involved in triage and incident handling.

The implementation covers SSH brute-force detection, AI-generated alert summaries, automated alert creation in DFIR-IRIS, email notifications, and analyst-driven incident escalation.

## 🏗️ Architecture

| Component | Role |
|---|---|
| **Wazuh** | Detects suspicious SSH authentication activity using custom rules |
| **Ollama + Qwen3:4B** | Analyzes alert evidence locally and generates concise summaries |
| **Shuffle SOAR** | Orchestrates security workflows and integrates the components |
| **DFIR-IRIS** | Manages security alerts and incident cases |
| **Email notifications** | Delivers alert information and decision guidance to the analyst |

The AI model runs locally through Ollama. The workflows communicate with the other components through HTTP requests and webhooks.

---

# 🔍 1. Security Monitoring & Detection

Three custom Wazuh rules cover different SSH authentication scenarios.

| Rule ID | Detection | Severity Level |
|---|---|---|
| `100100` | Five failed SSH password attempts from the same IP within 60 seconds | 10 |
| `100101` | Five SSH attempts using nonexistent usernames from the same IP within 60 seconds | 10 |
| `100102` | Successful SSH authentication following repeated failed attempts | 12 |

Rule `100102` is particularly important because a successful login following repeated failures may indicate account compromise. It is a suspicious activity indicator, not proof of compromise.

The rules also include MITRE ATT&CK mappings to help contextualize the detected activity.

### Wazuh — Detection Dashboard

The dashboard shows events associated with the three custom SSH rules, including the distribution of events by severity, MITRE ATT&CK technique, and agent.

![Wazuh dashboard showing SSH detection rules](screenshots/Screenshot%20From%202026-10-17%2020-44-43.png)

### DFIR-IRIS — Correlated SSH Alerts

The investigation includes correlated alerts for an SSH brute-force attempt and a successful login following repeated authentication failures.

![Correlated SSH alerts in DFIR-IRIS](screenshots/Screenshot%20From%202026-10-17%2020-31-40.png)

---

# ⚙️ 2. Automated SOC Workflow

The first Shuffle workflow automates the handling of incoming Wazuh alerts, from AI-assisted analysis to incident alert creation and email notification.

### Workflow Architecture

```text
       Wazuh Detection
              |
              v
       Shuffle Webhook
              |
              v
      Ollama + Qwen3:4B
       AI Alert Analysis
              |
              v
       Parse AI Output
              |
              v
     Create DFIR-IRIS Alert
              |
              v
      Email Notification
```

### Processing Steps

1. **Detection:** Wazuh identifies suspicious SSH authentication activity.
2. **AI analysis:** Shuffle sends relevant alert evidence to the local Qwen3:4B model through Ollama.
3. **Response parsing:** The workflow extracts the generated summary for use in subsequent steps.
4. **Alert creation:** Shuffle submits the processed information to DFIR-IRIS.
5. **Notification:** An email containing the security alert and supporting information is sent to the analyst.

### Shuffle — Complete Workflow

The screenshot shows the connected workflow used to process incoming alerts.

![Complete BruteForce-SSH workflow in Shuffle](screenshots/Screenshot%20From%202026-10-17%2020-29-35.png)

### Shuffle — Execution Results

The execution results demonstrate successful processing, including the AI request and HTTP response from the DFIR-IRIS alert creation endpoint.

![Shuffle workflow execution results](screenshots/Screenshot%20From%202026-10-17%2020-29-01.png)

---

# 📧 3. Automated Email Notification

After processing an alert, the workflow sends an email notification to the SOC analyst.

The notification includes the alert classification, relevant Wazuh detection information, original evidence, and guidance to support the analyst's decision.

The following two screenshots are complementary parts of the same email.

### Email Notification — Alert Summary

The first screenshot shows the notification, including the alert title, severity classification, associated Wazuh rule, and MITRE ATT&CK technique.

![SOC email notification and alert classification](screenshots/Screenshot%20From%202026-10-17%2020-45-29.png)

### Email Notification — Evidence & Analyst Decision Guide

The second screenshot continues the same email, showing the original SSH authentication evidence and the decision guidance for handling the alert in DFIR-IRIS.

![SOC email containing original evidence and analyst decision guidance](screenshots/Screenshot%20From%202026-10-17%2020-45-55.png)

Together, these screenshots demonstrate how the workflow provides both technical evidence and decision support rather than simply forwarding a raw security event.

---

# 🚨 4. Human-in-the-Loop Incident Management

The second Shuffle workflow connects alert status updates in DFIR-IRIS to incident-management actions.

The workflow receives a status update through a webhook and evaluates the analyst's decision.

### Workflow Logic

```text
     Analyst Updates Alert
      Status in DFIR-IRIS
               |
               v
        Shuffle Webhook
               |
               v
         Status Decision
           /        \
          /          \
  In Progress       Closed
       |                |
       v                v
  Create IRIS       Add Comment
      Case          to the Alert
```

### Analyst Decision: In Progress

When the analyst changes an alert's status to `In Progress`, Shuffle calls the DFIR-IRIS API to escalate the alert and create an incident case.

### Analyst Decision: Closed

When the alert is marked `Closed`, Shuffle adds a comment to the alert instead of creating an escalation case.

### Shuffle — Human-in-the-Loop Workflow

![Human-in-the-Loop Approval workflow in Shuffle](screenshots/Screenshot%20From%202026-10-17%2020-29-52.png)

### DFIR-IRIS — Incident Case

The following screenshot shows the resulting investigation case, including its severity and incident summary.

![DFIR-IRIS SSH brute-force investigation case](screenshots/Screenshot%20From%202026-10-17%2020-41-56.png)

### DFIR-IRIS — Investigation Details

The investigation details show the successful SSH login following repeated authentication failures, associated with Wazuh rule `100102` and MITRE ATT&CK technique `T1078`.

The evidence helps the analyst investigate the event and evaluate appropriate containment actions.

![DFIR-IRIS investigation details for rule 100102](screenshots/Screenshot%20From%202026-10-17%2020-42-17.png)

---

# 🤖 5. Local AI-Assisted Alert Analysis

The project uses **Ollama with Qwen3:4B** to analyze security alert evidence locally.

The analysis prompt is designed to:

- Summarize the event in two concise sentences.
- Explain why the event matters or what remains unknown.
- Use only the evidence supplied to the model.
- Avoid inventing details or claiming an account is compromised without sufficient evidence.
- Keep analysis separate from decisions to execute security responses.

The model supports the analyst by making alert information easier to review. It does not replace investigation or human judgment.

---

# 🔐 6. Human-Approved Response — Next Development Stage

The current workflows support detection, AI-assisted analysis, notification, and incident escalation.

The next development objective is to integrate analyst-approved containment actions into the existing workflow.

Planned actions include:

- Blocking a suspicious source IP.
- Disabling a confirmed compromised local account.
- Requiring explicit analyst authorization before executing a response.
- Recording response actions for investigation and auditing.

**Status:** Human-approved containment is still under development. These actions are not presented as fully implemented capabilities.

The intended approach is to require an explicit analyst decision before a containment action is executed.

---

# 🧰 Technology Stack

| Technology | Purpose |
|---|---|
| Wazuh | Security monitoring and detection |
| Ollama | Local AI model execution |
| Qwen3:4B | AI-assisted security alert analysis |
| Shuffle SOAR | Security workflow orchestration |
| DFIR-IRIS | Alert management and incident case handling |
| REST APIs | Communication between security components |
| Webhooks | Event-driven workflow execution |
| Python | Workflow data processing |
| JSON | Detection rules, AI prompts, and workflow configuration |
| Linux | Security lab environment |

---

# 📂 Repository Structure

```text
ai-soc-automation/
├── ai-analysis/
│   └── prompts/
│       └── alert-analysis.json
├── shuffle/
│   └── workflows/
│       ├── BruteForce-SSH.json
│       └── Human-in-the-Loop Approval.json
├── wazuh/
│   ├── active-response/
│   │   └── README.md
│   └── rules/
│       └── local_rules.xml
├── screenshots/
│   └── ... screenshots used in this README
├── .gitignore
└── README.md
```

---

# 🎯 Skills Demonstrated

- SIEM monitoring and custom detection rule configuration
- SSH brute-force detection and authentication-event correlation
- MITRE ATT&CK mapping
- Local LLM deployment using Ollama
- AI-assisted alert analysis
- SOAR workflow development
- REST API and webhook integration
- Automated alert creation and email notification
- Incident triage and escalation
- Human-in-the-loop security automation
- Security workflow design and documentation

---

# 🔒 Security Considerations

- AI-generated summaries are supporting information, not authoritative security decisions.
- Suspicious activity does not automatically prove account compromise.
- Containment actions should require explicit analyst authorization.
- Credentials, API tokens, passwords, and session cookies must not be committed to the repository.
- Screenshots should be checked for sensitive information before publication.
- Testing should be performed in an authorized lab environment.

---

# 👨‍💻 Author

**Yassine Nemri**

Cybersecurity Engineering Student  
Interests: SOC Operations · Security Automation · AI-Assisted Threat Analysis

- **GitHub:** [@Yassine1Nemri](https://github.com/Yassine1Nemri)
- **Project:** [AI-Driven SOC Automation](https://github.com/Yassine1Nemri/ai-soc-automation)

---

<div align="center">

**Built to explore practical SOC automation through security engineering, local AI, and controlled incident handling.**

</div>
