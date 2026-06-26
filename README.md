# Skill: Recap Supporto — CSM Briefing (Ask Product)

Claude Code skill for generating PowerPoint recap presentations of support history, ready for CSM (Customer Success Manager) meetings at Smartness/Smartpricing.

**Key difference from the base skill:** this version always asks which product to analyse before proceeding — it does not default to Smartchat.

---

## What it does

Given one or more HTML exports from Zendesk, the skill:

1. **Asks** which product to analyse: Smartpricing, Smartchat, Smartconnect, or SmartPMS — and waits for explicit confirmation before any analysis
2. **Parses** all tickets from the HTML files, deduplicates them by ticket number (same ticket can appear across multiple files), and filters only tickets relevant to the chosen product
3. **Filters** using product-specific include/exclude keyword rules applied to the ticket title and first 800 characters — cross-product tickets (e.g. a Smartpricing ticket that mentions Smartchat in passing) are handled correctly
4. **Classifies** filtered tickets into product-specific thematic clusters, each with a ticket count and a RESOLVED / OPEN / PARTIAL status badge
5. **Detects escalations and declared crises** — identifies timeline events, flags crises 🔴, and builds a chronological escalation map
6. **Analyses open issues** — reads the last two comments of every open ticket to determine who needs to act next and what the resolution path is (see [Open issues analysis](#open-issues-analysis))
7. **Extracts verbatim quotes** from customers expressing frustration or satisfaction, used on the "real problem" slide
8. **Generates** a `.pptx` file with Smartness brand in 8 standard slides, with the product name on the cover and in the filename
9. **Validates** the output via visual QA: LibreOffice → PDF → per-slide image rendering, checking for text truncation or overlap on at least cover, cluster slide, open issues, and CSM summary

Output: `/mnt/user-data/outputs/CustomerName_{Product}_Report_CSM.pptx`

---

## Product selection

At startup the skill asks:

> Per quale prodotto devo generare il report?
> 1. Smartpricing — prezzi, calendario, strategie di revenue
> 2. Smartchat — AI messaging, WhatsApp, conversazioni guest
> 3. Smartconnect — marketing, newsletter, shop, campagne
> 4. SmartPMS — gestione prenotazioni, CiaoBooking, PMS

The product is locked for the entire session — not re-asked.

---

## Supported products

| Product | Jira key | Jira pattern |
|---------|----------|--------------|
| Smartpricing | `PRD` | `PRD-\d+` |
| Smartchat | `SMARTCHAT` | `SMARTCHAT-\d+` |
| Smartconnect (marketing) | `PAY` | `PAY-\d+` |
| SmartPMS | `CIAOB` | `CIAOB-\d+` |

### Ticket filtering — include keywords

| Product | Include when title or first 800 chars contain… |
|---------|-----------------------------------------------|
| Smartpricing | smartpricing, prezzo/prezzi, calendario prezzi, strategia, revenue, tariffa, occupancy, minimum stay, prezzo minimo, raccomandazione prezzo |
| Smartchat | smartchat, AI risposta, messaggi non, conversazione, whatsapp (in Smartchat context), knowledge base, KB, risposta automatica, bot |
| Smartconnect | smartconnect, newsletter, shop, campagna/campagne, email marketing, invio email, template, comunicazione, promozione, lista contatti, iscritti, marketing |
| SmartPMS | smartpms, ciaobooking, pms, prenotazioni, overbooking, alloggiati web, ROSS1000, check-in online, fatturazione, extra, folio, city tax, tourist tax |

Tickets that belong exclusively to a different product are always excluded.

---

## Cluster classification

Tickets are grouped into product-specific thematic clusters. Each cluster shows a ticket count and a RESOLVED / OPEN / PARTIAL badge.

**Smartpricing clusters:** Calendario prezzi non aggiornato / ritardi · Raccomandazioni fuori mercato / anomalie algoritmo · Minimum stay / restrizioni non applicate · Occupancy / disponibilità incongruente · Integrazione con PMS / channel manager (lato pricing) · Accesso account / credenziali · Feature Request · Altro

**Smartchat clusters:** Messaggi non recapitati (Booking.com / Airbnb) · AI risponde con info sbagliate / confonde proprietà · Piattaforma lenta / non accessibile / down · Chat non associata a proprietà o prenotazione · Invio file/video WhatsApp non funzionante · Accesso account / credenziali · Gestione KB / proprietà in Smartchat · Anomalie AI varie (numeri telefono, checkout, indirizzi) · Feature Request

**Smartconnect clusters:** Newsletter non inviata / problemi di delivery · Template email — errori di personalizzazione o visualizzazione · Shop / prodotti — anomalie o mancata sincronizzazione · Campagne marketing — configurazione o invio fallito · Liste contatti / iscritti — gestione e sincronizzazione · Accesso account / credenziali · Feature Request · Altro

**SmartPMS clusters:** Prenotazioni non sincronizzate / mancanti · Overbooking / conflitti prenotazione · Check-in online non funzionante · Alloggiati Web / ROSS1000 — invio non riuscito · City tax / extra / folio — anomalie · Fatturazione / emissione documenti · Accesso account / credenziali · Feature Request · Altro

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
| 1 | Cover | KPIs: total tickets, confirmed bugs, declared crises, open issues — plus period and product name |
| 2 | Customer profile | Who they are, who contacts Support, how they use the product |
| 3 | Main clusters | 2×3 grid — each cluster with ticket count and RESOLVED / OPEN / PARTIAL badge |
| 4 | Escalation timeline | Chronological vertical timeline — declared crises marked 🔴, key events marked with date |
| 5 | Distribution | Horizontal bar chart — ticket count per category (omitted if too few data points) |
| 6 | The real problem | Dark slide — 1–2 verbatim customer quotes + 3 numbered analytical insights |
| 7 | Still-open issues | Each ticket: number, severity, title, agent in charge, description, colour-coded action badge |
| 8 | CSM summary | 5 operational points: what to raise in the meeting, what to monitor, what needs follow-up |

Slides are adaptive: slide 5 is removed if data is too sparse; slide 7 shows "No open issues" if all tickets are resolved.

---

## Pre-generation checklist

Before writing any code, the skill verifies:

1. Product confirmed — correct filter will be applied
2. Agent names — read from last thread comment, never assumed
3. Jira references — product-specific pattern scanned across all open ticket bodies
4. Resolution path — direct (1 step) or multi-step (Tech → Agent → Customer) for each open issue
5. Tickets closed but worth including for visibility
6. Verbatim customer quotes for slide 6

---

## QA

Every generated deck is converted to PDF via LibreOffice and rendered as per-slide images. Visual checks are performed on at minimum: cover, cluster slide, open issues, CSM summary. If text is truncated or overlapping, font size is reduced — row heights are never increased beyond slide limits.

Partial updates (single slide fix) regenerate only the affected slide for QA.

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
