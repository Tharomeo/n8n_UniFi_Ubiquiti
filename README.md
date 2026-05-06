<table>
  <tr>
    <td><img src="https://img.icons8.com/fluency/64/network.png" width="60"/></td>
    <td><h1>Ubiquiti Network Monitor</h1></td>
  </tr>
</table>

> Automated network monitoring and alerting system for Ubiquiti devices using n8n, Supabase and email notifications.

![n8n](https://img.shields.io/badge/n8n-EA4B71?style=for-the-badge&logo=n8n&logoColor=white)
![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Status](https://img.shields.io/badge/Status-Active-28a745?style=for-the-badge)

---

## 📌 About

This project automates the monitoring of Ubiquiti network devices. It detects when a device goes offline, sends an instant alert, records the outage in a database, and notifies again when the device is restored — including how long it was down.

At the end of each month, a summarized report is automatically sent with outage statistics for all devices.

---

## ✨ Features

- 🔁 **Scheduled polling** — continuously checks all devices on a set interval
- 📴 **Outage detection** — identifies offline devices and sends immediate email alerts
- ✅ **Restoration detection** — notifies when a device comes back online
- ⏱️ **Downtime tracking** — calculates and records the total duration of each outage
- 🗄️ **Persistent storage** — logs all outages to a Supabase database
- 📊 **Monthly report** — automated email with outage ranking and stats per device
- 🔔 **Duplicate alert prevention** — checks if an alert was already sent before notifying again

---

## 🔄 How it works

### Workflow 1 — Real-time Monitor (`Ubiquiti_New`)

```
Schedule Trigger
      │
      ▼
Fetch all Ubiquiti devices (HTTP Request)
      │
      ▼
Loop over each device
      │
      ├── [OFFLINE] ──────────────────────────────────────┐
      │         │                                          │
      │   Check if already notified?                       │
      │         ├── No  → Log to Supabase → Send alert email
      │         └── Yes → Skip                            │
      │                                                    │
      └── [ONLINE] ─────────────────────────────────────┐ │
                │                                        │ │
          Was it previously offline?                     │ │
                ├── Yes → Calculate downtime             │ │
                │         → Update Supabase record       │ │
                │         → Send restoration email       │ │
                └── No  → Skip                           │ │
```

### Workflow 2 — Monthly Report (`Relatório_Mensal_Ubiquiti`)

```
Schedule Trigger (monthly)
      │
      ▼
Fetch all outage records from Supabase
      │
      ▼
Process data in JavaScript
  - Filter outages > 5 minutes
  - Calculate total downtime per device
  - Rank by number of outages and downtime
      │
      ▼
Generate HTML email report
      │
      ▼
Send report to the team
```

---

## 📧 Notifications

**Outage alert** — sent immediately when a device goes offline
**Restoration alert** — sent when the device comes back online, including total downtime
**Monthly report** — sent on the first day of each month with:

| Metric | Description |
|---|---|
| Total downtime | Combined offline time across all devices |
| Most outages | Device with the highest number of outages |
| Longest downtime | Device that stayed offline the longest |
| Per-device breakdown | Individual outage count and downtime |

---

## 🗄️ Database

Data is stored in **Supabase** using the `device_outages` table.

| Column | Description |
|---|---|
| `device_name` | Name of the Ubiquiti device |
| `start_time` | Timestamp when the outage began |
| `duration_minutes` | Total downtime in minutes |

---

## 🛠️ Stack

| Tool | Role |
|---|---|
| **n8n** | Workflow automation engine |
| **Ubiquiti API** | Device status polling via HTTP |
| **Supabase** | Outage log database |
| **JavaScript** | Data processing and HTML report generation |
| **SMTP / Resend** | Email delivery |

---

## 🚀 Setup

**1. Import the workflows into your n8n instance**

- `Ubiquiti_New.json` — real-time monitor
- `Relatório_Mensal_Ubiquiti.json` — monthly report

**2. Configure credentials in n8n**

- `Supabase` — connect your Supabase project
- `SMTP` — configure your email sender (e.g. Resend)

**3. Create the database table in Supabase**

```sql
create table device_outages (
  id uuid primary key default gen_random_uuid(),
  device_name text,
  start_time timestamptz,
  duration_minutes int
);
```

**4. Activate both workflows**

<div align="center">

![built with n8n](https://img.shields.io/badge/Built_with-n8n-EA4B71?style=flat-square&logo=n8n&logoColor=white)

</div>
