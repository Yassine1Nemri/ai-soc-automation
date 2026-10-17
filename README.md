<div align="center">

# 🛡️ AI-Driven SOC Automation

### AI-Assisted Security Monitoring, SOAR & Incident Management

An intelligent SOC automation project integrating Wazuh, local AI, Shuffle SOAR, and DFIR-IRIS to streamline security alert analysis and incident handling.

**Detection · Local AI · Orchestration · Incident Management**

<br>

![Linux](https://img.shields.io/badge/Platform-Linux-FCC624?style=flat-square&logo=linux&logoColor=black)
![Wazuh](https://img.shields.io/badge/SIEM-Wazuh-005571?style=flat-square)
![Ollama](https://img.shields.io/badge/Local%20AI-Ollama-black?style=flat-square)
![Qwen3](https://img.shields.io/badge/Model-Qwen3%3A4B-7C3AED?style=flat-square)
![Shuffle](https://img.shields.io/badge/SOAR-Shuffle-6C47FF?style=flat-square)
![DFIR-IRIS](https://img.shields.io/badge/Incident%20Management-DFIR--IRIS-167D9A?style=flat-square)

</div>

---

## 📌 Overview

This project demonstrates how security monitoring, local AI analysis, SOAR automation, and incident management can work together in a practical SOC environment.

The objective is to automate repetitive analyst tasks while preserving human oversight over alert triage and incident handling.

The implementation includes custom SSH detection rules, AI-assisted alert analysis using Ollama and Qwen3:4B, automated alert creation in DFIR-IRIS, email notifications, and analyst-driven incident escalation.

## 🏗️ Architecture

The project integrates four main technologies:

| Component | Purpose |
|---|---|
| **Wazuh SIEM** | Detects suspicious SSH authentication activity |
| **Ollama + Qwen3:4B** | Analyzes alert evidence using a locally running language model |
| **Shuffle SOAR** | Orchestrates the detection, analysis, and incident-management workflows |
| **DFIR-IRIS** | Manages security alerts and incident cases |

Email notifications provide the analyst with the alert summary, original evidence, and decision guidance.

The architecture consists of two workflows.

### Workflow 1 — Detection, AI Analysis & Notification

```text
Linux Web Server
       |
       v
   Wazuh SIEM
       |
       | Alert Webhook
       v
   Shuffle SOAR
       |
       | HTTP POST
       v
 Ollama + Qwen3:4B
       |
       | JSON Summary
       v
   Python Parsing
       |
       v
 Create DFIR-IRIS Alert
       |
       v
 Email Notification
       |
       v
   SOC Analyst
```

### Workflow 2 — Human-in-the-Loop Escalation

```text
   Analyst Updates Alert
            |
            v
       DFIR-IRIS
            |
            | Status Webhook
            v
       Shuffle SOAR
            |
            v
      Status Decision
        /         \
       /           \
 In Progress       Closed
      |                |
      v                v
 Create IRIS       Add Comment
     Case          to Alert
```

---

# 🔍 1. Security Detection — Wazuh

Wazuh monitors SSH authentication activity and applies three custom detection rules designed to identify suspicious login patterns.

| Rule ID | Detection | Severity |
|---|---|---:|
| `100100` | Five failed SSH password attempts from the same IP within 60 seconds | 10 |
| `100101` | Five attempts using nonexistent usernames from the same IP within 60 seconds | 10 |
| `100102` | Successful SSH authentication following repeated failed attempts | 12 |

Rule `100102` is particularly important because a successful login following repeated authentication failures can indicate a potentially compromised account.

However, this behavior is a suspicious activity indicator, not proof of compromise.

The rules also include MITRE ATT&CK mappings to help contextualize the detected behavior.

### Wazuh — Security Event Analysis

The Wazuh dashboard shows the distribution of SSH-related events across the custom rules, severity levels, MITRE ATT&CK techniques, and monitored agent.

![Wazuh SSH detection dashboard](screenshots/Screenshot%20From%202026-10-17%2020-44-43.png)

### Rule 100102 — Successful Login After Repeated Failures

This investigation focuses on successful SSH authentication following repeated failures, associated with the account `testuser` in the observed lab event.

![Wazuh Rule 100102](screenshots/Screenshot%20From%202026-10-17%2020-25-03.png)

---

# 🤖 2. Local AI Analysis — Ollama & Qwen3:4B

Instead of relying on a cloud-based AI API, this project uses **Ollama to run Qwen3:4B locally** for AI-assisted security alert analysis.

When Wazuh generates a security alert, Shuffle sends relevant evidence to the local Ollama API for analysis.

The model generates a concise summary that is parsed by the workflow and included in the subsequent incident-management process.

## How the AI Integration Works

```text
     Wazuh Alert
          |
          v
      Shuffle SOAR
          |
          | HTTP POST
          v
   Ollama API /api/chat
          |
          v
      Qwen3:4B
     Local Model
          |
          | JSON Response
          v
     Python Parse
          |
          v
   DFIR-IRIS + Email
```

### Model Configuration

| Setting | Value |
|---|---|
| Runtime | Ollama |
| Model | `qwen3:4b` |
| API endpoint | `/api/chat` |
| Streaming | Disabled |
| Temperature | `0` |
| Maximum generated tokens | `200` |
| Output format | Structured JSON |
| Expected output field | `summary` |

The workflow requests a JSON response containing a single `summary` field.

### Evidence-Based Analysis

The model receives relevant alert information, including the Wazuh rule ID, source IP, target username, previous authentication-failure evidence, and the current log event.

Its prompt instructs it to:

- Explain what happened in two short sentences.
- Describe why the event matters or identify what remains unknown.
- Use only the supplied evidence.
- Avoid inventing details.
- Avoid claiming that an account is compromised unless the evidence supports that conclusion.
- Return a structured JSON response.

The AI supports alert interpretation; it does not independently authorize containment actions.

### 📸 AI Integration in Action

The following execution screenshot demonstrates the actual Shuffle workflow processing an alert, including the workflow's processing and successful DFIR-IRIS request.

![Shuffle AI-assisted workflow execution](screenshots/Screenshot%20From%202026-10-17%2020-29-01.png)

**Additional evidence to capture:** Open the successful HTTP node that calls Ollama in Shuffle and capture its response showing the model name `qwen3:4b` and the generated JSON `summary`.

Save that screenshot as:

` screenshots/ollama-qwen3-response.png `

Then add it directly below this paragraph:

```markdown
![Ollama Qwen3:4B response](screenshots/ollama-qwen3-response.png)
```

This dedicated screenshot will make the local AI integration easier for recruiters to verify.

---

# ⚙️ 3. Automated SOC Workflow — Shuffle SOAR

The first Shuffle workflow connects Wazuh detection, local AI analysis, incident alert creation, and email notification.

### Processing Steps

**1. Detection:** Wazuh identifies suspicious SSH authentication activity.

**2. AI analysis:** Shuffle submits relevant alert evidence to Ollama and Qwen3:4B.

**3. Response parsing:** A Python node processes the AI-generated response.

**4. Alert creation:** Shuffle sends the processed information to DFIR-IRIS through its API.

**5. Notification:** The workflow sends an email notification to the analyst.

### Shuffle — Complete Workflow

The workflow connects the alert webhook, HTTP request, Python processing, DFIR-IRIS alert creation, and email notification.

![Complete BruteForce-SSH workflow](screenshots/Screenshot%20From%202026-10-17%2020-29-35.png)

### Successful Execution

The execution results show the processing stages and the successful HTTP response from the DFIR-IRIS alert creation endpoint.

![Shuffle workflow execution results](screenshots/Screenshot%20From%202026-10-17%2020-29-01.png)

---

# 📧 4. Automated Email Notification

Following alert processing, Shuffle sends an email to the SOC analyst.

The notification provides the alert classification, associated detection information, original evidence, and guidance to support the analyst's decision.

The following two screenshots are complementary parts of the same email.

### Alert Summary

The first screenshot displays the SOC alert notification, including the alert title, severity classification, Wazuh rule, and MITRE ATT&CK information.

![SOC alert email summary](screenshots/Screenshot%20From%202026-10-17%2020-45-29.png)

### Original Evidence & Analyst Decision Guide

The second screenshot continues the email, presenting the original SSH authentication evidence and guidance for evaluating the alert in DFIR-IRIS.

![SOC alert email evidence and decision guide](screenshots/Screenshot%20From%202026-10-17%2020-45-55.png)

Together, these screenshots demonstrate how the workflow gives the analyst both technical evidence and contextual decision support.

---

# 🚨 5. Human-in-the-Loop Incident Management

The second Shuffle workflow connects analyst-driven alert status changes in DFIR-IRIS to incident-management actions.

A webhook receives the status update, and a decision node directs the workflow according to the selected status.

### Analyst Decision: In Progress

When the alert is marked `In Progress`, the workflow calls the DFIR-IRIS API to escalate the alert and create an incident case.

### Analyst Decision: Closed

When the alert is marked `Closed`, the workflow adds a comment to the existing alert instead of creating an escalation case.

### Shuffle — Human-in-the-Loop Workflow

![Human-in-the-Loop Approval workflow](screenshots/Screenshot%20From%202026-10-17%2020-29-52.png)

### DFIR-IRIS — Incident Case

The resulting investigation case documents the incident and provides a centralized place for reviewing the associated security activity.

![DFIR-IRIS incident case](screenshots/Screenshot%20From%202026-10-17%2020-41-56.png)

### Investigation Details — Rule 100102

The investigation details show the successful SSH login following repeated failures and provide the analyst with information for evaluating appropriate next steps.

![DFIR-IRIS Rule 100102 investigation](screenshots/Screenshot%20From%202026-10-17%2020-42-17.png)

---

# 🔐 6. Planned Development — Analyst-Approved Containment

The current workflows provide security detection, AI-assisted analysis, email notification, alert management, and incident escalation.

The next development stage is to extend the existing workflow with analyst-approved containment actions.

Planned capabilities include:

- Blocking a suspicious source IP.
- Disabling a confirmed compromised local account.
- Requiring explicit authorization before executing a response.
- Recording response actions for investigation and auditing.

**Status: Under development.**

These containment capabilities are not presented as operational features of the current implementation.

The objective is to ensure that security-sensitive actions remain under explicit analyst control.

---

# 🧰 7. Technology Stack

| Technology | Purpose |
|---|---|
| Wazuh | Security monitoring and detection |
| Ollama | Local language-model runtime |
| Qwen3:4B | AI-assisted alert analysis |
| Shuffle SOAR | Workflow orchestration and automation |
| DFIR-IRIS | Alert management and case escalation |
| REST APIs | Communication between security components |
| Webhooks | Event-driven workflow execution |
| Python | Alert response parsing and processing |
| JSON | Detection rules, AI prompt configuration, and structured responses |
| Linux | Security lab environment |

---

# 📂 8. Repository Structure

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
│   └── ...
├── .gitignore
└── README.md
```

---

# 🎯 9. Skills Demonstrated

- SIEM monitoring and custom detection rule configuration
- SSH brute-force detection and authentication-event correlation
- MITRE ATT&CK mapping
- Local language-model deployment with Ollama
- AI-assisted security alert analysis
- SOAR workflow development
- REST API and webhook integration
- Automated alert creation and email notification
- Incident triage and case escalation
- Human-in-the-loop automation design
- Security workflow documentation

---

# 🔒 10. Security Considerations

- AI-generated summaries are supporting information, not authoritative security decisions.
- Suspicious activity does not automatically prove account compromise.
- Containment actions should require explicit analyst authorization.
- Credentials, API tokens, passwords, session cookies, and private keys must not be committed to the repository.
- Screenshots should be reviewed for sensitive information before publication.
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
