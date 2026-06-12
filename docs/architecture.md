# System Architecture

## AI Sales Coordinator System

The AI Sales Coordinator System is an AI-powered CRM automation project built with n8n, OpenAI, Google Sheets, Gmail, and Telegram.

The system is designed for a real estate demo company, Premier Realty. It automates the lead management process from inquiry capture to AI qualification, agent assignment, CRM tracking, follow-up automation, and escalation alerts.

---

## Architecture Overview

The system is divided into three connected n8n workflows.

The workflows are connected through Google Sheets, which acts as the central CRM database.

```text
Lead Inquiry
   ↓
Workflow 1: Lead Intake & Sales Coordination
   ↓
Google Sheets CRM
   ↓
Workflow 2: Follow-Up Automation Engine
   ↓
Workflow 3: Escalation & Lead Recovery Engine
```

---

## Workflow 1: Lead Intake & Sales Coordination

This workflow starts when a new lead submits a property inquiry.

### Purpose

To capture, qualify, assign, notify, and log a new lead automatically.

### Flow

```text
Lead Inquiry Form
↓
Normalize Lead Data
↓
AI Lead Qualification Engine
↓
Parse AI Qualification JSON
↓
Merge Lead + AI Qualification
↓
Hot Lead?
├─ Hot Residential/Condo → Assign Anna Santos
├─ Hot Commercial/Land → Assign Bob Reyes
└─ Not Hot → Route Warm or Cold?
    ├─ Warm → Assign Bob Reyes
    └─ Cold → Assign Carl Cruz
↓
CRM: Create Lead Record
↓
Notify Assigned Agent
↓
Activity Log: Agent Notified
↓
Send Lead Acknowledgement Email
↓
Activity Log: Acknowledgement Email Sent
↓
Follow-Up: Create First Follow-Up Task
↓
Activity Log: Follow-Up Scheduled
```

### Key Outputs

- New lead record in the Leads sheet
- Agent notification sent through Telegram
- Lead acknowledgement email sent through Gmail
- First follow-up task created in the Follow-Ups sheet
- Activity history recorded in the Activity Log sheet

---

## Workflow 2: Follow-Up Automation Engine

This workflow runs on a schedule and checks the Follow-Ups sheet for pending follow-up tasks.

### Purpose

To automatically send follow-up emails and update the CRM when follow-ups are completed.

### Flow

```text
Schedule Trigger
↓
Get Follow-Up Tasks
↓
Due Follow-Up?
├─ True → Send Follow-Up Email
│        ↓
│        Update Follow-Up Status
│        ↓
│        Activity Log: Follow-Up Sent
└─ False → Do nothing
```

### Current Testing Logic

During testing, the workflow checks:

```text
Status = Pending
Lead Email is not empty
```

### Production Logic

For production, the workflow should also check:

```text
Scheduled Date <= current date/time
```

Recommended production condition:

```javascript
{{ DateTime.fromFormat($json["Scheduled Date"], 'yyyy-MM-dd HH:mm:ss') <= $now }}
```

### Key Outputs

- Follow-up email sent through Gmail
- Follow-Up row updated from Pending to Sent
- Sent Date recorded
- Follow-Up Sent activity logged

---

## Workflow 3: Escalation & Lead Recovery Engine

This workflow runs on a schedule and checks pending follow-ups that need attention.

### Purpose

To alert the manager or assigned agent when a follow-up needs recovery or escalation.

### Flow

```text
Schedule Trigger
↓
Get Pending Follow-Ups
↓
Overdue Follow-Up?
├─ True → Notify Manager / Agent
│        ↓
│        Update Escalation Status
│        ↓
│        Activity Log: Escalation Sent
└─ False → Do nothing
```

### Current Testing Logic

During testing, the workflow checks:

```text
Status = Pending
Lead Email is not empty
```

### Production Logic

For production, the workflow should check:

```text
Status = Pending
Lead Email is not empty
Scheduled Date < current date/time
Escalation Status is not Escalated
```

### Key Outputs

- Telegram escalation alert sent
- Follow-Up row marked as Escalated
- Escalated Date recorded
- Escalation Sent activity logged

---

## Google Sheets CRM Structure

Google Sheets is used as the CRM database and workflow coordination layer.

### Leads Sheet

Stores the main lead record.

Example fields:

```text
Lead ID
Date Created
Name
Email
Phone
Property Type
Budget
Timeline
Message
Lead Score
Temperature
Priority
Qualification Reason
Recommended Action
Suggested Agent Type
Assigned Agent
Agent Email
Agent Phone
Agent Specialty
Lead Status
Follow-Up Status
Appointment Status
Outcome
Escalation Status
Assigned Date
```

### Activity Log Sheet

Stores a timestamped audit trail of system actions.

Example activity types:

```text
Agent Notified
Acknowledgement Email Sent
Follow-Up Scheduled
Follow-Up Sent
Escalation Sent
```

### Follow-Ups Sheet

Stores scheduled follow-up tasks.

Example fields:

```text
Lead ID
Lead Name
Lead Email
Follow-Up Number
Scheduled Date
Status
Sent Date
Assigned Agent
Email Type
Priority
Notes
Escalation Status
Escalated Date
```

### Dashboard Sheet

Displays CRM metrics such as:

```text
Total Leads
Hot Leads
Warm Leads
Cold Leads
Pending Follow-Ups
Sent Follow-Ups
Escalated Follow-Ups
Total Activities Logged
Leads Per Agent
Pipeline Status
Follow-Up Status
Activity Log Summary
```

---

## Data Flow

```text
Webhook Lead Data
↓
Normalized Lead Data
↓
AI Qualification Data
↓
Merged Lead Record
↓
Google Sheets CRM
↓
Follow-Up Queue
↓
Scheduled Follow-Up Workflow
↓
Escalation Workflow
```

Google Sheets acts as the shared data layer between workflows.

---

## AI Lead Qualification

OpenAI analyzes each lead and returns structured JSON.

Example output:

```json
{
  "lead_score": 85,
  "lead_temperature": "Hot",
  "priority": "High",
  "qualification_reason": "The lead has clear buying intent, a strong budget, and an urgent timeline.",
  "recommended_action": "Schedule a property viewing as soon as possible.",
  "suggested_agent_type": "Residential"
}
```

The workflow parses this JSON and uses the fields for routing and CRM updates.

---

## Lead Routing Logic

```text
Hot + Residential/Condo → Anna Santos
Hot + Commercial/Land → Bob Reyes
Warm → Bob Reyes
Cold → Carl Cruz
```

This allows the sales team to prioritize high-value and urgent leads.

---

## Automation Tools Used

| Tool | Purpose |
|---|---|
| n8n | Workflow automation and orchestration |
| OpenAI | AI lead qualification and scoring |
| Google Sheets | CRM database, follow-up queue, activity log, dashboard |
| Gmail | Automated lead acknowledgement and follow-up emails |
| Telegram | Agent and manager notifications |
| Webhook / Hoppscotch | Lead inquiry testing |
| Docker | Local n8n environment |

---

## Production Notes

Before using this system in production, the following improvements are recommended:

1. Add a unique Follow-Up ID to avoid matching only by Lead ID.
2. Add date-based filtering for due follow-ups.
3. Add escalation filtering to prevent repeated escalation alerts.
4. Change schedule triggers from testing intervals to daily or hourly production schedules.
5. Replace demo/test emails with real lead emails.
6. Store sensitive credentials only inside n8n credentials, never inside workflow JSON files.
7. Move from Google Sheets to a database such as Supabase if the CRM grows larger.

---

## Security Notes

The GitHub version of the workflow files should be sanitized.

Do not upload:

```text
API keys
Telegram bot tokens
Google client secrets
Real customer data
Private Gmail addresses
Production webhook URLs with secrets
```

Use placeholders instead:

```text
YOUR_GOOGLE_SHEET_ID
YOUR_TELEGRAM_CHAT_ID
YOUR_OPENAI_CREDENTIAL
YOUR_GMAIL_CREDENTIAL
YOUR_GOOGLE_SHEETS_CREDENTIAL
```

---

## Summary

This architecture separates the system into multiple workflows so each automation has a clear responsibility:

```text
Workflow 1 → Handles new leads
Workflow 2 → Handles scheduled follow-ups
Workflow 3 → Handles overdue follow-up escalation
```

This makes the project easier to maintain, test, document, and present as a professional CRM automation system.
