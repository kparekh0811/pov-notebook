# PoV Notebook + Homerun Eval Plan Generator

Generate two outputs from a Datadog PoV Technical Evaluation Plan Excel file:
1. A Datadog Notebook markdown file (`.md`)
2. A Homerun-compatible eval plan CSV (`.csv`) ready to import

**Usage:** `/pov-notebook <path-to-xlsx>`

---

## Step 1 — Read the Excel File

Read every sheet in the Excel file using Python with openpyxl. Install it first if needed:
```
pip3 install --break-system-packages openpyxl -q
```

Extract all non-empty cell values from every sheet. Strip leading/trailing whitespace from sheet names when matching. Skip formula strings like `=DAYS(...)`. Convert datetime objects to human-readable strings (e.g. `April 30, 2026`).

Parse the following data (sheet names vary — use fuzzy/partial matching):

- **Summary / Scope sheet**: Customer name, POV start/end dates, Datadog SE and AE names/emails, customer contacts (name, email, team/role), environment/tech stack (cloud provider, containers/orchestration, languages, databases, integrations), number of environments, team size, products in scope
- **Success Criteria sheet**: Each criterion, its expected outcome, and which products it covers
- **Use Cases / PoC Use Cases sheet**: Each use case and its priority (P1, P2, etc.)
- **Mutual Action Plan / Schedule sheet**: All tasks/meetings with dates, owners, and status
- **Pre-Requisites / Architecture sheet**: Network requirements, assumptions
- **Tech Plan sheet**: Phases, configuration items, estimated time, **responsible party** (customer team/person), target completion date, status, and notes. The responsible party column name may vary (e.g. "Responsible Party GFiber", "Responsible Party", "Owner") — capture it regardless of the customer name in the header.

---

## Step 2 — Generate the Notebook Markdown

Write a `.md` file to the same directory as the input `.xlsx`, named `<customer-name-lowercase-hyphenated>-pov-notebook.md`.

```markdown
---
title: 'PoV Notebook - <Customer Name>'
author: <DD SE Name>
modified: <today's date ISO format>
tags: []
metadata:
  type: Documentation
time:
  live_span: 1h
template_variables: []
---

## Datadog Evaluation

`Welcome to your Datadog trial environment! All the information you will need to get started can be found in this notebook. If you have any questions, please reach out. We are here to support you.`

### Account team

* <DD SE Name>, Solutions Engineer, <email>
* <DD AE Name>, Account Executive, <email>

### <Customer Name> Contacts

* <Contact Name>, <Role/Team>, <email>
(repeat for each contact)

### POV Dates

* **Start:** <start date>
* **End:** <end date>
* **Scope:** <number of environments, team size, any scope notes>

## Environment Details

* Cloud environment: **<cloud provider>**
* Containers and orchestration: **<e.g. GKE, EKS, AKS>**
* Databases: **<list>**
* Programming languages/frameworks: **<list>**
* Integrations in scope: **<list>**
* Teams involved: **<list>**

## Success Criteria

- [ ] **<Criterion Title>**
  - [ ] <Expected outcome bullet 1>
  - [ ] <Expected outcome bullet 2>
  - [ ] *Products: <product list>*

## PoC Use Cases

| Use Case | Priority | Status |
|---|---|---|
| <use case> | P1 | |

## Schedule

### Phase 0 — Pre-Work

| Task | Responsible Party | Target Date | Status |
|---|---|---|---|
| <item> | <Datadog (SE) \| Customer Team \| Both> | <date or blank> | |

### Phase 1 — Configuration / Data Collection

Render each product area as a sub-section with a table. The table must always have a **Responsible Party** column so ownership is explicit and trackable. Use the value from the Tech Plan sheet's responsible party column when present; if blank, default based on the item type:
- Items that require customer access, credentials, or environment changes → `<Customer Name> Team`
- Items that are Datadog configuration, instrumentation guidance, or documentation → `Datadog (SE)`
- Items that require both → `Both`

#### <Product Area>

| Task | Responsible Party | Target Date | Status |
|---|---|---|---|
| <config item> | <Datadog (SE) \| Customer Team \| Both> | <date or blank> | |

### Phase 2 — Working Sessions & Check-ins

| Date | Event | Attendees |
|---|---|---|
| <date> | <event + duration> | <DD attendees> · <Customer attendees> |

### Phase 3 — PoV Review

| Task | Responsible Party | Target Date | Status |
|---|---|---|---|
| End of PoV review | Datadog (SE) | | |
| Confirm final scope & volume | Both | | |

## Deployment Resources

* [<Resource Name>](<url>)

## FAQs

| Question | Answer | Documentation |
|---|---|---|
```

---

## Step 3 — Generate the Homerun Eval Plan CSV

Write a `.csv` file to the same directory as the input `.xlsx`, named `<customer-name-lowercase-hyphenated>-homerun-evalplan.csv`.

### CSV format

Always include this header row:
```
Depth Value,Status,Assignee,Due Date,Status Text,Roll up children's status,Name,Description,Success Criteria,Shared
```

Column rules:
- `Depth Value`: hierarchical dot-notation number (`1`, `1.1`, `1.1.1`, `2`, `2.1`, etc.)
- `Status`: `Not Started` for all rows
- `Assignee`: Use the responsible party value from the Tech Plan sheet when present. If blank, apply the same default logic as the notebook: customer-side setup tasks → customer contact name or `<Customer> Team`; Datadog guidance/instrumentation → `Datadog (SE)`; shared tasks → `Both`. Fall back to `Not Assigned` only if ownership truly cannot be determined.
- `Due Date`: `No due date` (use actual date only if a specific milestone date is known from the Mutual Action Plan)
- `Status Text`: empty
- `Roll up children's status`: `true` for any row that has children, `false` for leaf items
- `Name`: item name
- `Description`: description text, doc links formatted as `Text ( https://... )`
- `Success Criteria`: success criteria text if applicable, else empty
- `Shared`: `false`

Escape commas and newlines in values by wrapping in double quotes; escape internal double quotes by doubling them (`""`).

### Template location

A Homerun template CSV is at `homerun/evalplan-template.csv` in the same repository as this command file. Locate it by finding the directory of this command file (`.claude/commands/`) and going up two levels to the repo root, then into `homerun/`.

Load all rows from the template. Use them as the source for Phase 0 (trial setup), Phase 1 (configuration), Phase 2 (UI exploration), and Phase 3 (review).

### Filter template rows based on the customer's tech stack

**Always include:**
- Phase 0: Sign up / trial setup rows
- Phase 2: Dashboards & Visualizations, Alerting
- Phase 3: POV Review

**Include conditionally — only if the customer uses that technology:**

| Customer Tech Stack | Template section(s) to include |
|---|---|
| GCP | `GCP` infrastructure section |
| AWS | `AWS` infrastructure section |
| Azure | `Azure` infrastructure section |
| Kubernetes / GKE / EKS / AKS | `Kubernetes (OpenShift)` section |
| Linux hosts | `Host Based Agent - Linux` section |
| Windows hosts | `Host Based Agent - Windows` section |
| Node.js | APM `Node.js` section |
| Java / Kotlin | APM `Java` section |
| .NET / C# | APM `.NET` section |
| Python | APM `Python` section |
| RUM / Session Replay / browser | `RUM` section |
| Synthetics | `Synthetic Testing` section |
| Logs | `Log Collection` section |
| Security / CSPM / SIEM | `Security` section |
| Network monitoring | `Networks` section |
| Cloud cost | `Cloud Cost Management` section |

When a section is included, include its parent row and all its children. When excluded, drop the entire subtree.

### Add customer-specific sections at the top

Prepend two new sections before the template rows, then renumber everything sequentially:

**Section 1 — Success Criteria** (from Excel Success Criteria sheet):
```
1,Not Started,Not Assigned,No due date,,true,Success Criteria,,,false
1.1,Not Started,Not Assigned,No due date,,false,"<Criterion Title>","<Expected Outcome>","<Success Criteria text>",false
1.2,...
```

**Section 2 — PoC Use Cases** (from Excel Use Cases sheet):
```
2,Not Started,Not Assigned,No due date,,true,PoC Use Cases,,,false
2.1,Not Started,Not Assigned,No due date,,false,"<Use Case Name>","Priority: P1 — <description if any>",,false
2.2,...
```

Then the filtered template sections follow as sections 3, 4, 5, 6 (renumber their depth values to start at 3).

Renumber all depth values consistently so the hierarchy is clean end-to-end.

---

## Step 4 — Report back

After writing both files:
- Output the path of both files
- Summary: customer name, POV dates, # success criteria, # use cases, # meetings scheduled
- List which template sections were **included** and which were **excluded** in the Homerun CSV, and why
- Call out any data that was missing or unclear from the Excel

---

**General notes:**
- Do not hallucinate content — only use what is in the Excel file
- If a field is missing, omit the section or mark it `TODO`
- If a sheet name doesn't match exactly, try stripping whitespace and case-insensitive matching

The user's Excel file path is: $ARGUMENTS
