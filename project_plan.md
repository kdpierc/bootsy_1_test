# 🏭 Atlas Stability Tracking — Project Plan
**Owner:** kdpierc (Supply Chain Technology Operations, Walmart)
**Created:** Day 3 of onboarding
**Purpose:** Consolidate, track, monitor, and trend defects/issues for Atlas WMS across Distribution Centers

---

## 📌 Background & Context

- **System Being Tracked:** Atlas (Warehouse Management System deployed across Walmart Distribution Centers)
- **Team:** Supply Chain Technology Operations
- **Problem Statement:** Issues were not being discovered until escalated from the field. The team needs proactive visibility into Atlas stability across DCs.
- **Issue Sources:**
  - Jira tickets (prioritized in a spreadsheet)
  - ServiceNow tickets
  - Field escalations from DCs
- **Issue Types:** Issues may result in code defects, enhancements, or training issues
- **Goal:** Consolidate all information from all sources into one place, log and monitor issues, identify trends, and bring visibility to ensure issues are being worked

---

## 🗂️ Issue Tracking Fields (Master Issue Log)

Every issue should capture the following fields:

| Field | Purpose | Example Values |
|-------|---------|----------------|
| Issue ID | Unique identifier | JIRA-1234, SN-5678 |
| Source | Where it came from | Jira, ServiceNow, Field Escalation |
| DC / Site | Which distribution center | DC 6094, DC 7210 |
| Date Reported | When it came in | 2025-01-15 |
| Issue Type | Classification | Code Defect, Enhancement, Training, Data Issue |
| Severity | How bad is it | P1-Critical, P2-High, P3-Medium, P4-Low |
| System / Module | What part of Atlas | Receiving, Putaway, Picking, Shipping |
| Description | What happened | Short summary |
| Status | Where it stands | New, In Progress, Blocked, Resolved, Closed |
| Owner | Who is working it | Dev team, Ops, Training team |
| Date Resolved | When fixed | 2025-01-20 |
| Root Cause | Why it happened | Config error, Code bug, User error |
| Resolution | How it was fixed | Patch deployed, Training conducted |
| Linked Ticket | Cross-reference | ServiceNow linked to Jira |
| Notes | Anything extra | Field comments, workarounds |

---

## 📊 KPI Health Measures — Organized by Category

### 🛒 Category 1: Order Fulfillment Health
*Is Atlas creating and dropping orders correctly?*

| KPI | What It Measures | Alert If... |
|-----|-----------------|-------------|
| EDF Order Drop % | % of EDF orders successfully dropped into Atlas | Drop below threshold |
| Outs + Atlas SSTK Orders Created | Standing stock orders created vs. outs | High outs with low SSTK creation |
| Bypassed Orders | Orders that skipped normal Atlas flow | Any significant volume |
| % of DA Cases Received w/ active pick order | Direct Allocation cases received properly tied to picks | % drops |
| % of XDK Cases Transmitted w/ active pick order | Cross-dock cases transmitted with valid pick orders | % drops |

### 📦 Category 2: Inventory Integrity
*Does Atlas know where inventory actually is?*

| KPI | What It Measures | Alert If... |
|-----|-----------------|-------------|
| EI vs. Atlas | External Inventory system count vs. Atlas count | Variance grows |
| # of Hung Allocations (Inventory Allocated w/o Picks Created) | Inventory allocated in Atlas but no picks created | Count increases |
| # of Outs by Reason | Stockouts broken down by root cause | Spike in any reason |

### 🤖 Category 3: Automation & Systems Integration Health
*Is Atlas talking correctly to all downstream systems?*

| KPI | What It Measures | Alert If... |
|-----|-----------------|-------------|
| % of Automation Containers Inducted vs. PAC'd | Containers successfully inducted into automation | % drops |
| % of Automation Picks created in Atlas, transmitted to SYM | Atlas picks transmitted to SYM (Symbotic) | % drops |
| % of BP Picks created in Atlas transmitted to Intelligrated | Break Pack picks transmitted to Intelligrated | % drops |
| % of Voice Picks created in Atlas transmitted to Honeywell | Voice picks transmitted to Honeywell | % drops |
| % of MOCC Messages Honored by Atlas (SYM) | Atlas honoring MOCC messages from Symbotic | % drops |
| % of Repack Close Messages Honored by Atlas (Intelligrated) | Atlas honoring Repack Close from Intelligrated | % drops |
| % of Pallet Close Messages Honored by Atlas (Honeywell) | Atlas honoring Pallet Close from Honeywell | % drops |

### 💰 Category 4: Financial & Invoice Reconciliation
*Is what Atlas says we shipped matching what we invoiced?*

| KPI | What It Measures | Alert If... |
|-----|-----------------|-------------|
| GRS vs. Atlas Loaded vs. Invoiced | Gross Receipts vs. what Atlas loaded vs. invoiced | Variance grows |
| In Transit vs. Invoiced | Cases in transit vs. invoiced cases | Gap widens |
| Invoice % Variance | % difference between expected and actual invoices | Exceeds threshold |
| Account Health (True Ups, Rewrites, Voids, Shipvoids) | Volume of account corrections needed | Any spike |

---

## 🏗️ Folder Structure Plan (GitHub Repo: bootsy_1_test)

```
📁 bootsy_1_test/
├── 📁 tracking/
│   ├── master_issue_log.csv        ← All issues from Jira, ServiceNow, Field
│   └── kpi_health_tracker.csv      ← Daily/weekly KPI snapshot
├── 📁 sql/
│   ├── order_fulfillment_kpis.sql  ← Category 1 queries
│   ├── inventory_integrity.sql     ← Category 2 queries
│   ├── automation_health.sql       ← Category 3 queries
│   └── invoice_reconciliation.sql  ← Category 4 queries
├── 📁 reports/
│   └── weekly_stability_report.md  ← Summary for manager
├── project_plan.md                 ← THIS FILE (master context doc)
└── README.md
```

---

## ✅ Next Steps (Pick Up Here!)

### Immediate (Waiting On):
- [ ] **Get BigQuery access confirmed** — Ask manager what GCP project/datasets are available
- [ ] **Get GCP project name(s)** — Need exact project ID to search BigQuery
- [ ] **Confirm Google account setup** — Walmart email linked to GCP?

### Once BigQuery Access is Confirmed:
- [ ] Search BigQuery for Atlas-related tables (`%atlas%`)
- [ ] Search BigQuery for ServiceNow data (`%servicenow%`)
- [ ] Search BigQuery for Jira data (`%jira%`)
- [ ] Search BigQuery for order/fulfillment data (`%order%`, `%allocation%`)
- [ ] Search BigQuery for invoice/GRS data (`%invoice%`, `%grs%`)
- [ ] Map each KPI to a BigQuery data source
- [ ] Write SQL queries for each KPI category
- [ ] Build master issue log CSV template
- [ ] Build KPI health tracker CSV template

### Ongoing:
- [ ] Weekly KPI snapshots saved to GitHub
- [ ] Issue log updated as new tickets come in
- [ ] Weekly stability report generated for manager

---

## 🔁 How to Resume This Session

### Step 1 — Open a new BigQuery Explorer session

### Step 2 — Paste this message to the new agent:
```
Hi! I am continuing a previous project. Please read my project plan file from my GitHub repo
to get full context on what we are working on. The file is at:
https://github.com/kdpierc/bootsy_1_test/blob/main/project_plan.md

My local repo is at: /Users/kpierc1/bootsy_1_test

Here is a summary of where we left off:
- I am on the Supply Chain Technology Operations team at Walmart (new employee, day 3)
- We are building an Atlas WMS stability tracking system
- We set up this GitHub repo to store queries, KPI trackers, and issue logs
- I was waiting to hear from my manager about BigQuery/GCP access
- Once I have BigQuery access confirmed, next step is to search for Atlas, ServiceNow,
  Jira, and supply chain data in BigQuery
- See the project_plan.md file for full KPI list, issue tracking fields, and next steps
```

### Step 3 — Tell the agent your BigQuery access status
Let the agent know:
- What GCP project(s) you have access to
- Whether your Walmart email is linked to GCP
- Any dataset names your team mentioned

---

*Last Updated: Day 3 of onboarding*
*GitHub Repo: https://github.com/kdpierc/bootsy_1_test*
