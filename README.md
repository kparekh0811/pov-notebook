# pov-notebook

A Claude Code slash command that generates a Datadog PoV notebook markdown file from a Technical Evaluation Plan Excel spreadsheet.

## What it does

Given a `.xlsx` PoV evaluation plan, the `/pov-notebook` command extracts:

- Customer name, POV dates, and team contacts (Datadog + customer side)
- Environment details (cloud, containers, languages, databases, integrations)
- Success criteria with expected outcomes and products in scope
- PoC use cases with priorities
- Full schedule / mutual action plan with dates and attendees
- Tech plan phases and configuration checklist
- Deployment prerequisites and network requirements

And outputs a fully structured `.md` notebook file ready to import into Datadog Notebooks.

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

The output `.md` file is written to the same directory as the input file, named `<customer-name>-pov-notebook.md`.

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
