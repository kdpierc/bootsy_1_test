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

## 🏗️ Systems Architecture & Inventory Lifecycle

> *Researched via Walmart Confluence (HAW space) — Day 3 of onboarding*

### How a Piece of Inventory Travels Through Atlas

#### 📥 Step 1: Receiving (Inbound)
```
Supplier ships → ASN published to Atlas (GDM → ASN_INBOUND_TOPIC)
  ↓
UWMS Receiving validates ASN → hands off to automation (Symbotic or Witron)
  ↓
Receiving confirmation → Inventory Server Cloud
  ↓
Putaway task created → Worker confirms → Inventory record created in Atlas
```

#### 📦 Step 2: Inventory Lives in Atlas
- Atlas **Inventory Server Cloud** holds real-time inventory using Event Sourcing + CQRS
- Knows exactly where every item is, quantity, and status
- Cached in Redis for fast lookups (L1: Caffeine 10s TTL → L2: Redis 5min TTL)
- Auto-releases reservations after 30 min if order not fulfilled

#### 🛒 Step 3: Order Comes In (Allocation)
```
Customer order arrives → Allocation Service reserves inventory
  ↓
Pick request created → sent to Hawkeye WCS (automation middleman)
  ↓
Hawkeye routes to the right automation system:
  ├── Symbotic (SYM)      → robotic AS/RS systems
  ├── Intelligrated        → Pick-to-Light / Voice Pick
  └── Honeywell            → Voice picking
```

#### 🚚 Step 4: Pick → Pack → Ship
```
Automation confirms picks back to Atlas via Kafka
  ↓
Container loaded → Watcher system tracks loading events
  ↓
YMS (Yard Management) gate-out → Order marked COMPLETE ✅
  ↓
DC-Fin / SAP → Financial reconciliation (GRS vs. invoiced)
```

### 🔌 Key Systems That Integrate with Atlas

| System | Role | Integration Method |
|--------|------|-------------------|
| **Hawkeye WCS** | Central traffic cop — orchestrates all automation | Kafka |
| **Symbotic (SYM)** | Robotic AS/RS storage & retrieval (DC 6020 pilot) | Kafka via Hawkeye |
| **Intelligrated** | Pick-to-Light, Voice Pick, sortation | SFTP flat-file → migrating to MQ |
| **Honeywell** | Voice picking systems | Kafka via Hawkeye |
| **GDM** | Sends ASNs (what's inbound) | Kafka |
| **DD (Direct Delivery)** | Cross-dock allocation at SDCs | Kafka (async) |
| **DC-Fin / SAP** | Financial & invoice reconciliation | Kafka → REST API |
| **SLIP** | Middleware: translates new Kafka events → legacy GLS flat files | Apache Camel microservices |
| **Witron** | Automated conveyor & palletizing (GDC facilities) | Hawkeye |
| **Grey Orange** | Autonomous Mobile Robots (AMRs) | Hawkeye |

### 🚌 Key Kafka Brokers

| Broker | Purpose |
|--------|---------|
| **Atlas** | Primary inter-service communication |
| **Hawkeye** | WCS / automation system integration |
| **Watcher** | Loading and gate operations |
| **Optima** | Transportation and trip management |

### 📚 Key Confluence Pages (Bookmark These!)

| Topic | Link | Space |
|-------|------|-------|
| Atlas WMS System Overview | [🔗](https://confluence.walmart.com/display/HAW/Atlas+WMS+System+Overview) | HAW |
| Walmart Order Lifecycle E2E | [🔗](https://confluence.walmart.com/pages/viewpage.action?pageId=3426766207) | CPGSCOPE |
| Atlas Inventory Server Cloud | [🔗](https://confluence.walmart.com/display/HAW/Atlas+Inventory+Server+Cloud) | HAW |
| Atlas UWMS Receiving | [🔗](https://confluence.walmart.com/display/HAW/Atlas+UWMS+Receiving) | HAW |
| Atlas Allocation Order Service | [🔗](https://confluence.walmart.com/display/HAW/Atlas+Allocation+Order+Service) | HAW |
| Atlas NDOF Trip Execution (FES) | [🔗](https://confluence.walmart.com/pages/viewpage.action?pageId=3508289978) | HAW |
| Atlas Kafka Topics Index | [🔗](https://confluence.walmart.com/display/HAW/Atlas+Kafka+Topics+Index) | HAW |
| Atlas WMS @ RDC (Symbotic pilot) | [🔗](https://confluence.walmart.com/display/SCTA/Atlas+WMS+@+RDC) | SCTA |
| Intelligrated Integration Details | [🔗](https://confluence.walmart.com/display/SCHI/3.+INTELLIGRATED) | SCHI |
| SLIP Integration Platform | [🔗](https://confluence.walmart.com/display/SASCTA/SLIP+-+Supply+Chain+Logistics+Integration+Platform) | SASCTA |
| DD ↔ WMS (Atlas) Integration | [🔗](https://confluence.walmart.com/display/SASCTA/DD+-+WMS+%28Atlas%29+Integration) | SASCTA |
| Atlas Deployment Methodology | [🔗](https://confluence.walmart.com/display/HAW/Atlas+Deployment+Methodology) | HAW |

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

## 🎫 Jira Project Ecosystem (Confirmed)

> *Researched via Jira agent — May 7, 2026*

| Project Key | Focus | Activity |
|-------------|-------|----------|
| **NMS** | OneAtlas Platform — WMS core | 🔴 Very High (9,255/90d) |
| **IUF** | International Unit Fulfillment | 🔴 Very High (4,887/90d) |
| **GLSMAV** | E2E Automation Testing for Atlas | 🔴 Very High (821 bugs/30d) |
| **BLACKBIRD** | Atlas eComm FCs + Symbotic | 🟠 High (520/90d) |
| **OPIF** | Atlas parent Epics & Platform Infra | 🟠 High |
| **LAIE** | Legacy Atlas Integration Engineering | 🟡 Medium-High |
| **SAGLS** | Sam's Club Atlas GLS Support | 🟡 Medium |
| **AG** | Atlas GDC (Grocery DC) | 🟡 Medium |
| **RTII** | Retail Tech Intl Integration (Symbotic rollouts) | 🟡 Medium |
| **CEMPIF** | Canada/EMEA Atlas Platform Infra | 🟡 Medium |
| **MOBILE** | Mobile Device Mgmt (Honeywell/TC) | 🟢 Low |
| **SCTA** | Supply Chain Tech & Atlas RDC | 🟢 Low |
| **BEDA** | Business Engineering Data Analytics | 🟢 Low |

**Key Issue Volumes (90-day snapshot, May 2026):**
- Receiving: 3,166 issues | Picking/OF: 1,323 | Shipping: 1,281
- Symbotic integration: 726 | Putaway: 301 | Intelligrated: 96
- Open P1-Critical issues: **5** | Open bugs (30d): **325+**

---

## ✅ Next Steps (Pick Up Here!)

### Immediate (Waiting On):
- [ ] **Get BigQuery access confirmed** — Ask manager what GCP project/datasets are available
- [ ] **Get GCP project name(s)** — Need exact project ID to search BigQuery
- [ ] **Confirm Google account setup** — Walmart email linked to GCP?

### ✅ Completed (No BigQuery Needed!):
- [x] **Researched Atlas systems architecture** via Confluence (HAW space)
- [x] **Discovered 15+ active Jira projects** for Atlas WMS
- [x] **Built master issue log CSV** — seeded with 30 real Jira tickets (`tracking/master_issue_log.csv`)
- [x] **Built Atlas Stability Dashboard** — HTML report with charts (`reports/atlas_stability_dashboard.html`)

### Once BigQuery Access is Confirmed:
- [ ] Search BigQuery for Atlas-related tables (`%atlas%`)
- [ ] Search BigQuery for ServiceNow data (`%servicenow%`)
- [ ] Search BigQuery for Jira data (`%jira%`)
- [ ] Search BigQuery for order/fulfillment data (`%order%`, `%allocation%`)
- [ ] Search BigQuery for invoice/GRS data (`%invoice%`, `%grs%`)
- [ ] Map each KPI to a BigQuery data source
- [ ] Write SQL queries for each KPI category
- [ ] Build KPI health tracker CSV template (`tracking/kpi_health_tracker.csv`)

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
- I am on the Supply Chain Technology Operations team at Walmart (new employee, ~Day 3)
- We are building an Atlas WMS stability tracking system
- We set up this GitHub repo to store queries, KPI trackers, and issue logs
- We researched Atlas architecture via Confluence — see the Systems Architecture section
  in project_plan.md for full inventory lifecycle, systems integrations, and Kafka topics
- I was waiting on BigQuery/GCP access (getting a 400 error — IT needs to provision it)
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

*Last Updated: May 7, 2026 — added Jira project ecosystem, completed master issue log CSV + HTML stability dashboard*
*GitHub Repo: https://github.com/kdpierc/bootsy_1_test*
