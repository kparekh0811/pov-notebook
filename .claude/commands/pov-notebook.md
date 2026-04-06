# PoV Notebook Generator

Generate a Datadog PoV notebook markdown file from a Technical Evaluation Plan Excel file.

**Usage:** `/pov-notebook <path-to-xlsx>`

## Instructions

The user has provided a path to a PoV Technical Evaluation Plan Excel file (`.xlsx`). Your job is to:

1. **Read every sheet** in the Excel file using Python with openpyxl. Install it first if needed via `pip3 install --break-system-packages openpyxl -q`. Extract all non-empty cell values from every sheet.

2. **Parse the following key data** from the sheets (sheet names may vary slightly between files — use fuzzy matching):
   - **Summary / Scope sheet**: Customer name, POV start/end dates, Datadog SE and AE names/emails, customer contacts (name, email, team/role), environment/tech stack (cloud provider, containers/orchestration, languages, databases, integrations), number of environments, team size, products in scope
   - **Success Criteria sheet**: Each criterion, its expected outcome, and which products it covers
   - **Use Cases / PoC Use Cases sheet**: Each use case and its priority (P1, P2, etc.)
   - **Mutual Action Plan / Schedule sheet**: All tasks/meetings with dates, owners, and status
   - **Pre-Requisites / Architecture sheet**: Network requirements, assumptions, any architecture notes
   - **Tech Plan sheet**: Phases and configuration items that need to be completed

3. **Generate a markdown notebook** following this exact structure and format (adapt content to what was found in the file):

```markdown
---
title: 'PoV Notebook - <Customer Name>'
author: <DD SE Name>
modified: <today's date in ISO format>
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

(For each criterion from the sheet, render as a checkbox section with sub-items for expected outcomes and a Products line)

- [ ] **<Criterion Title>**
  - [ ] <Expected outcome bullet 1>
  - [ ] <Expected outcome bullet 2>
  - [ ] *Products: <product list>*

## PoC Use Cases

| Use Case | Priority | Status |
|---|---|---|
| <use case> | P1 | |
(repeat for all use cases, sorted by priority)

## Schedule

### Phase 0 — Pre-Work
(pre-work items from tech plan or action plan)

- [ ] <item>

### Phase 1 — Configuration / Data Collection
(group tech plan items by product area as sub-sections)

#### <Product Area>
- [ ] <config item>

### Phase 2 — Working Sessions & Check-ins

| Date | Event | Attendees |
|---|---|---|
| <date> | <event name + duration> | <DD attendees> · <Customer attendees> |
(all meetings from the mutual action plan, sorted by date)

### Phase 3 — PoV Review
- [ ] End of PoV review
- [ ] Confirm final scope & volume

## Deployment Resources

(Include relevant doc links based on tech stack found — e.g. GCP, AWS, Azure, K8s, specific integrations)

* [<Resource Name>](<url>)

## FAQs

| Question | Answer | Documentation |
|---|---|---|
```

4. **Write the output file** to the same directory as the input `.xlsx` file, named `<customer-name-lowercase-hyphenated>-pov-notebook.md`.

5. **Report back** with:
   - The output file path
   - A brief summary of what was found (customer, dates, # success criteria, # use cases, # scheduled events)
   - Any data that was missing or unclear from the Excel file

**Important notes:**
- If a sheet name doesn't match exactly, look for sheets with similar names (e.g. "Mutual Action Plan" vs " Mutual Action Plan" — watch for leading/trailing spaces)
- Skip formula strings like `=DAYS(...)` — treat them as empty
- Convert datetime objects to human-readable date strings (e.g. `April 30, 2026`)
- If a field is missing from the Excel, omit that section or leave a `TODO` placeholder
- Do not hallucinate content — only include what is actually in the file

The user's Excel file path is: $ARGUMENTS
