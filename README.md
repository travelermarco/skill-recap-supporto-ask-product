# Skill: Recap Supporto — CSM Briefing (Ask Product)

Claude Code skill for generating PowerPoint recap presentations of support history, ready for CSM (Customer Success Manager) meetings at Smartness/Smartpricing.

**Key difference from the base skill:** this version always asks which product to analyse before proceeding — it does not default to Smartchat.

---

## What it does

Given one or more HTML exports from Zendesk, the skill:

1. **Asks** the user which product to analyse: Smartpricing, Smartchat, Smartconnect, or SmartPMS
2. **Parses** all tickets, deduplicates them, and filters only tickets for the chosen product
3. **Classifies** them into product-specific thematic clusters
4. **Analyses** open issues — determining owner, resolution path, and any linked Jira references (patterns vary by product)
5. **Generates** a `.pptx` file with Smartness brand (violet/indigo palette) in 8 standard slides, with the product name on the cover and in the filename
6. **Validates** the output via visual QA (LibreOffice → PDF → per-slide image rendering)

Output: `/mnt/user-data/outputs/CustomerName_{Product}_Report_CSM.pptx`

---

## Supported products

| Product | Filter keywords | Jira patterns |
|---------|----------------|---------------|
| Smartpricing | prezzi, calendario, strategia, revenue, tariffa, occupancy | `SMARTPRI-\d+`, `SP-\d+` |
| Smartchat | smartchat, AI risposta, messaggi non, conversazione, whatsapp | `SMARTCHAT-\d+`, `CIAOB-\d+` |
| Smartconnect | channel manager, sincronizzazione, sync, OTA, disponibilità | `SMARTCON-\d+`, `CONNECT-\d+` |
| SmartPMS | smartpms, ciaobooking, prenotazioni, overbooking, alloggiati web | `SMARTPMS-\d+`, `PMS-\d+` |

---

## Triggers

Use this skill when the user:
- Provides one or more `.html` Zendesk export files
- Asks for: report, recap, presentation, briefing, ticket summary, CSM meeting prep
- Says: "generate the report for X", "prepare the presentation for Y", "summarise Z's tickets"

The skill will immediately ask which product to focus on before any analysis.

---

## Presentation structure (8 slides)

| # | Slide | Content |
|---|-------|---------|
| 1 | Cover | Total KPIs, period, **product name** |
| 2 | Customer profile | Who they are, who contacts Support, how they use the product |
| 3 | Main clusters | Grid with RESOLVED / OPEN / PARTIAL badges (product-specific clusters) |
| 4 | Escalation timeline | Declared crises 🔴, key events |
| 5 | Distribution | Horizontal bar chart — ticket count per category |
| 6 | The real problem | Dark slide with verbatim customer quotes |
| 7 | Still-open issues | Tickets with owner, Jira ref, action badge (CUSTOMER / SUPPORT / SILENT) |
| 8 | CSM summary | 5 operational points to bring into the meeting |

---

## Smartness brand

- **Palette**: navy `#1E1B55`, violet `#7C6FF0`, off-white `#EEEEFF`
- **Dark slides**: Cover, "The real problem", CSM Summary
- **Light slides**: all others
- **Layout**: 16:9, generated with pptxgenjs

---

## Dependencies

```bash
npm install -g pptxgenjs      # PPTX generation
pip install beautifulsoup4    # Zendesk HTML parsing
```

QA: LibreOffice + `pdftoppm`

---

## Files

| File | Description |
|------|-------------|
| `SKILL.md` | Full skill definition (frontmatter + operational instructions) |
| `README.md` | This file |
