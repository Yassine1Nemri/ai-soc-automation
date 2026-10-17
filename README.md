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

This project demonstrates the integration of security monitoring, local AI-assisted analysis, workflow orchestration, and incident management in a practical SOC environment.

The objective is to reduce repetitive analyst tasks while maintaining visibility and human oversight throughout the incident-handling process.

The current implementation includes SSH brute-force detection, AI-generated alert summaries, automated alert creation in DFIR-IRIS, email notifications, and status-based case escalation.

## 🏗️ Architecture

The project is built around four main components:

| Component | Role |
|---|---|
| **Wazuh** | Detects suspicious activity and generates security alerts |
| **Ollama + Qwen3:4B** | Analyzes alert evidence locally and generates concise summaries |
| **Shuffle SOAR** | Connects the components and automates workflows |
| **DFIR-IRIS** | Manages security alerts and incident cases |

Ollama runs locally, allowing AI-assisted analysis without sending alert data to a cloud-based AI API.

---

# 🔍 Workflow 1 — Detection, AI Analysis & Alert Creation

This workflow automates the initial handling of SSH brute-force alerts, from detection to analyst notification.

### ⚙️ Workflow

```text
SSH Authentication Failures
             |
             v
      Wazuh Detection
        Rule 100100
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

### 🔄 Processing Steps

1. **Detection:** Wazuh identifies repeated SSH authentication failures from the same source IP within 60 seconds.
2. **AI analysis:** Shuffle sends relevant alert evidence to the locally running Qwen3:4B model through Ollama.
3. **Response parsing:** The workflow parses the generated summary for subsequent processing.
4. **Alert creation:** Shuffle submits the processed information to DFIR-IRIS.
5. **Notification:** An email notification is sent to the analyst.

### 📸 Implementation Evidence

**Wazuh — SSH Brute-Force Detection**

<!-- Add your Wazuh detection screenshot here -->

![Wazuh Detection](screenshots/wazuh-detection.png)

**Shuffle — Automated Detection Workflow**

<!-- Add your Shuffle workflow diagram here -->

![Brute-Force Workflow](screenshots/bruteforce-workflow.png)

**Shuffle — Workflow Execution**

<!-- Add your execution results showing the successful HTTP requests here -->

![Brute-Force Execution](screenshots/bruteforce-execution.png)

**DFIR-IRIS — Generated Alert**

<!-- Add your IRIS alert screenshot here -->

![DFIR-IRIS Alert](screenshots/iris-alert.png)

---

# 🚨 Workflow 2 — Alert Triage & Case Escalation

The second workflow connects analyst decisions in DFIR-IRIS to automated incident-management actions in Shuffle.

A webhook receives alert status updates and directs execution according to the new status.

### ⚙️ Workflow

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

### 🔄 Processing Steps

**When the status is `In Progress`:**

- Shuffle processes the status update.
- The workflow calls the DFIR-IRIS API to escalate the alert.
- A corresponding incident case is created.

**When the status is `Closed`:**

- Shuffle adds a comment to the alert.
- The workflow records the closure-related action without creating an escalation case.

### 📸 Implementation Evidence

**Shuffle — Status-Based Escalation Workflow**

<!-- Add your second Shuffle workflow screenshot here -->

![Escalation Workflow](screenshots/escalation-workflow.png)

**DFIR-IRIS — Incident Case**

<!-- Add the resulting IRIS case screenshot here -->

![DFIR-IRIS Case](screenshots/iris-case.png)

---

# 🤖 Local AI-Assisted Analysis

The project uses **Ollama with Qwen3:4B** to analyze security alert evidence locally.

The analysis prompt is designed to:

- Summarize the security event in two concise sentences.
- Explain the significance of the event or identify what remains unknown.
- Use only the information available in the supplied evidence.
- Avoid inventing details or claiming an account is compromised without sufficient evidence.
- Keep AI-generated analysis separate from automated response decisions.

The model supports the SOC analyst by making alert information easier to review. It does not replace investigation or human judgment.

---

# 🔐 Human-in-the-Loop Incident Response

An additional development objective is to extend the existing incident-management workflow with analyst-approved containment actions.

Potential response actions include:

- Blocking a suspicious source IP.
- Disabling a confirmed compromised local account.
- Requiring explicit authorization before executing a response.
- Recording response actions for investigation and auditing.

**Status:** Human-approved containment is under development. These actions are not represented as operational capabilities of the current workflows.

The intended approach is to keep the analyst in control of security-sensitive actions rather than automatically responding to every detection.

---

# 🧰 Technology Stack

| Technology | Purpose |
|---|---|
| Wazuh | Security monitoring and detection |
| Ollama | Local AI model execution |
| Qwen3:4B | AI-assisted alert analysis |
| Shuffle SOAR | Workflow automation and orchestration |
| DFIR-IRIS | Alert management and incident case handling |
| REST APIs | Communication between security components |
| Webhooks | Event-driven workflow execution |
| Python | Workflow data processing |
| JSON | Rules, prompts, and workflow configuration |
| Linux | Lab environment |

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
│   └── ...
├── .gitignore
└── README.md
```

---

# 🎯 Skills Demonstrated

- SIEM alert detection and rule configuration
- SSH brute-force detection and authentication-event analysis
- AI integration for security alert analysis
- Local LLM deployment using Ollama
- SOAR workflow development
- REST API and webhook integration
- Automated alert creation and email notifications
- Incident triage and case escalation
- Human-in-the-loop security automation
- Security workflow design and documentation

---

# 🔒 Security Considerations

- AI-generated summaries are treated as supporting information, not authoritative security decisions.
- Response actions should require explicit analyst authorization.
- API credentials, passwords, session cookies, and private keys must never be committed to the repository.
- Screenshots should be reviewed before publication to remove sensitive infrastructure details.
- Testing should be performed in an authorized lab environment.

---

# 👨‍💻 Author

**Yassine Nemri**

Cybersecurity Engineering Student  
Interests: SOC Operations · Security Automation · AI-Assisted Threat Analysis

- GitHub: [@Yassine1Nemri](https://github.com/Yassine1Nemri)
- Project: [AI-Driven SOC Automation](https://github.com/Yassine1Nemri/ai-soc-automation)

---

<div align="center">

**Built to explore practical SOC automation through security engineering, local AI, and controlled incident handling.**

</div>
