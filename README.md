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

| Product | Filter keywords | Jira key | Jira pattern |
|---------|----------------|----------|--------------|
| Smartpricing | prezzi, calendario, strategia, revenue, tariffa, occupancy | `PRD` | `PRD-\d+` |
| Smartchat | smartchat, AI risposta, messaggi non, conversazione, whatsapp | `SMARTCHAT` | `SMARTCHAT-\d+` |
| Smartconnect | channel manager, sincronizzazione, sync, OTA, disponibilità | `PAY` | `PAY-\d+` |
| SmartPMS | smartpms, ciaobooking, prenotazioni, overbooking, alloggiati web | `CIAOB` | `CIAOB-\d+` |

---

## Open issues analysis

A core feature of this skill is the **analysis of pending actions on open tickets**. For every ticket that is still open (or recently closed but relevant), the skill reads the last two comments to classify its status:

| Status | Meaning | Badge colour |
|--------|---------|--------------|
| `cliente` | Customer must reply to an agent — agent is waiting for data, screenshot, or video | Blue — "Awaiting reply: [agent name]" |
| `support` | Support or Tech must communicate the outcome to the customer, or Tech must confirm to Support | Amber — "Support/Tech must [action]" |
| `silente` | Customer has not replied after the agent's request — ticket stalled | Grey — "Silent ticket — customer has not responded" |
| `chiuso_visibilità` | Formally closed but included for visibility (recent or significant issue) | Grey ✓ — "Closed — included for visibility" |

For each open issue the skill also:
- Identifies the **agent in charge** by reading the last support comment in the thread (never assumed from chat context)
- Extracts any **linked Jira reference** using the product-specific key (e.g. `PRD-123`, `SMARTCHAT-456`)
- Describes the **full resolution path** when multiple steps are required — e.g. *"Jira resolved by Tech → Tech must confirm to [agent] → [agent] communicates outcome to customer"*

All open issues are surfaced on **slide 7** of the deck, each with ticket number, severity badge, title, agent in charge, description, and a colour-coded action badge.

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
