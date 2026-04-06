# pov-notebook

A Claude Code slash command that generates two outputs from a Datadog PoV Technical Evaluation Plan Excel spreadsheet:
1. A Datadog Notebook markdown file ready to import into Datadog Notebooks
2. A Homerun-compatible eval plan CSV ready to import into Presales Homerun

## What it does

Given a `.xlsx` PoV evaluation plan, the `/pov-notebook` command extracts:

- Customer name, POV dates, and team contacts (Datadog + customer side)
- Environment details (cloud, containers, languages, databases, integrations)
- Success criteria with expected outcomes and products in scope
- PoC use cases with priorities
- Full schedule / mutual action plan with dates and attendees
- Tech plan phases and configuration checklist
- Deployment prerequisites and network requirements

And outputs two files to the same directory as the input Excel file.

## Installation

**Option 1 — Project-level (shared, recommended for teams):**

Clone this repo and open Claude Code from inside it. The command is automatically available when Claude Code is running in this directory.

```bash
git clone https://github.com/kparekh0811/pov-notebook
cd pov-notebook
claude
```

**Option 2 — User-level (always available personally):**

Copy the command file to your personal Claude commands directory:

```bash
cp .claude/commands/pov-notebook.md ~/.claude/commands/pov-notebook.md
```

## Usage

In Claude Code, run:

```
/pov-notebook /path/to/your-eval-plan.xlsx
```

Two files are written to the same directory as the input Excel:

| Output file | Purpose |
|---|---|
| `<customer>-pov-notebook.md` | Import into Datadog Notebooks |
| `<customer>-homerun-evalplan.csv` | Import into Presales Homerun |

## Requirements

- [Claude Code](https://claude.ai/code) CLI or VS Code / JetBrains extension
- Python 3 (for reading the Excel file — `openpyxl` is auto-installed by the command if missing)

## Excel template compatibility

The command is designed to work with Datadog's standard PoV Technical Evaluation Plan template. It handles common variations in sheet naming (leading/trailing spaces, slight name differences) across different versions of the template.

Expected sheets:
| Sheet | Content |
|---|---|
| POV Summary & Scope | Customer info, contacts, tech stack, products |
| Success Criteria | Evaluation criteria and expected outcomes |
| PoC Use Cases | Use cases with priority |
| Mutual Action Plan | Schedule, meetings, owners |
| Tech Plan | Phase-by-phase configuration checklist |
| Pre-Requisites & Architecture | Network requirements, assumptions |

## Homerun CSV format

The Homerun eval plan CSV uses a hierarchical format that maps directly to Homerun's eval plan import structure.

### Columns

| Column | Description |
|---|---|
| `Depth Value` | Hierarchical dot-notation position (e.g. `1`, `1.1`, `1.1.2`) |
| `Status` | `Not Started`, `In Progress`, or `Complete` |
| `Assignee` | Assigned user (defaults to `Not Assigned`) |
| `Due Date` | Date or `No due date` |
| `Status Text` | Free-text status note |
| `Roll up children's status` | `true` if the row has children, `false` for leaf items |
| `Name` | Item name |
| `Description` | Item description; doc links formatted as `Text ( https://... )` |
| `Success Criteria` | Acceptance criteria for the item |
| `Shared` | Whether the item is shared with the customer (`true`/`false`) |

### Structure

The generated CSV follows this top-level structure:

| Section | Depth | Source |
|---|---|---|
| Success Criteria | `1` | Pulled from Excel Success Criteria sheet |
| PoC Use Cases | `2` | Pulled from Excel Use Cases sheet |
| Phase 0: Sign up / Trial Setup | `3` | From Homerun template |
| Phase 1: Configuration / Data Collection | `4` | From Homerun template, filtered by tech stack |
| Phase 2: Inside the UI | `5` | From Homerun template |
| Phase 3: POV Review | `6` | From Homerun template |

### Tech stack filtering

Phase 1 sections are automatically included or excluded based on the customer's tech stack detected from the Excel. For example, a GCP/GKE customer will get GCP, Kubernetes, and GCP-specific log/APM sections — AWS, Azure, and Windows sections are dropped.

| Tech stack detected | Sections included |
|---|---|
| GCP | GCP infrastructure |
| AWS | AWS infrastructure |
| Azure | Azure infrastructure |
| GKE / EKS / AKS / Kubernetes | Kubernetes section |
| Linux hosts | Host Based Agent - Linux |
| Windows hosts | Host Based Agent - Windows |
| Java / Kotlin | APM Java |
| Node.js | APM Node.js |
| .NET / C# | APM .NET |
| Python | APM Python |
| RUM / Session Replay | RUM section |
| Synthetics | Synthetic Testing section |
| Logs | Log Collection section |
| Security | Security section |
| Network monitoring | Networks section |
| Cloud cost | Cloud Cost Management section |

The base Homerun template is stored at `homerun/evalplan-template.csv` in this repo and can be updated as new products or sections are added.
