---
name: recap-supporto-ask-product
description: >
  Skill per generare presentazioni PowerPoint di riepilogo storico supporto per la CSM (Customer Success Manager)
  di Smartness, partendo da export Zendesk in HTML. Usa questa skill ogni volta che l'utente fornisce uno o più
  file HTML di export Zendesk e chiede un report, recap, presentazione, briefing o riepilogo per un cliente,
  per la CSM, per un meeting, o per analizzare lo storico ticket di un cliente. Trigger anche su: "genera il report
  per X", "fai la presentazione per il meeting con Y", "riassumi i ticket di Z", "prepara il briefing CSM".
  A differenza della skill base, questa versione chiede SEMPRE all'utente per quale prodotto generare il report
  prima di procedere all'analisi: Smartpricing, Smartchat, Smartconnect, SmartPMS.
  La skill produce un file .pptx scaricabile con brand Smartness (viola/indigo), strutturato per slide,
  con cluster di problematiche, timeline, issue aperte con action owner, e riepilogo operativo per la CSM.
---

# Skill: Recap Supporto — Briefing CSM (Ask Product)

Genera presentazioni PowerPoint professionali di riepilogo storico supporto per la CSM,
partendo da export Zendesk HTML. Brand Smartness. Output: file .pptx scaricabile.

**Differenza rispetto alla skill base:** questa skill non ha un prodotto predefinito.
Prima di qualsiasi analisi chiede sempre all'utente per quale prodotto generare il report.

---

## 0. SELEZIONE PRODOTTO (step obbligatorio — da eseguire PRIMA di qualsiasi analisi)

Appena l'utente fornisce i file HTML Zendesk, **prima di parsare qualsiasi ticket**, porre esattamente questa domanda:

> Per quale prodotto devo generare il report?
> 1. **Smartpricing** — prezzi, calendario, strategie di revenue
> 2. **Smartchat** — AI messaging, WhatsApp, conversazioni guest
> 3. **Smartconnect** — sincronizzazione channel manager
> 4. **SmartPMS** — gestione prenotazioni, CiaoBooking, PMS
>
> Rispondi con il numero o il nome del prodotto.

Attendere la risposta dell'utente. Non procedere senza conferma esplicita del prodotto.
Una volta confermato il prodotto, **tenerlo fisso per l'intera sessione** e non ri-chiedere.

---

## 1. INPUT ATTESO

L'utente fornirà:
- Uno o più file `.html` export Zendesk (uno per cliente)
- Il prodotto da analizzare (raccolto nello step 0)
- Il nome del cliente (o ricavarlo dai ticket)
- Eventualmente: colori brand, nome agenti specifici, correzioni su ticket aperti

---

## 2. PARSING DEI TICKET

### 2.1 Estrazione
Usare `BeautifulSoup` per parsare ogni file HTML. Per ogni ticket estrarre:
- Numero ticket (`#XXXXX`)
- Titolo
- Tutti i commenti con autore, data, corpo (primi 300-400 chars per commento)
- Riferimenti Jira (pattern specifici per prodotto — vedere sezione 2.4)
- Deduplicare ticket per numero (stessi ticket appaiono in file diversi)

### 2.2 Filtro prodotto
Applicare il filtro corrispondente al prodotto selezionato nello step 0.

#### SMARTPRICING
**Includere** ticket che nel titolo o nei primi 800 chars contengono:
- "smartpricing", "smart pricing", "prezzo", "prezzi", "calendario prezzi", "strategia", "revenue", "tariffa", "tariffe", "occupancy", "minimum stay", "prezzo minimo", "raccomandazione prezzo"

**Escludere** ticket che riguardano esclusivamente:
- Smartchat / AI messaggi (no Smartpricing)
- Smartconnect / sync channel manager (no Smartpricing)
- SmartPMS / prenotazioni / CiaoBooking (no Smartpricing)
- Fatturazione / subscription (Chargebee, variazione sub) — a meno che non citino esplicitamente Smartpricing

#### SMARTCHAT
**Includere** ticket che nel titolo o nei primi 800 chars contengono:
- "smartchat", "smart chat", "AI risposta", "messaggi non", "conversazione", "whatsapp" in contesto Smartchat, "knowledge base", "KB", "risposta automatica", "bot"

**Escludere** ticket che riguardano esclusivamente:
- SmartPMS / CiaoBooking (prenotazioni, overbooking, Alloggiati Web, ROSS1000)
- Smartpricing (prezzi, calendario, strategie)
- Smartconnect (sync channel manager)
- Fatturazione / subscription
- Gestione immobili (aggiunta/rimozione proprietà, solo se non menziona Smartchat)

#### SMARTCONNECT
**Includere** ticket che nel titolo o nei primi 800 chars contengono:
- "smartconnect", "smart connect", "channel manager", "sincronizzazione", "sync", "connettore", "OTA", "disponibilità", "disponibilita", "aggiornamento disponibilità", "mismatch prenotazioni", "doppia prenotazione" in contesto channel manager

**Escludere** ticket che riguardano esclusivamente:
- Smartchat / AI messaggi
- Smartpricing (prezzi, calendario)
- SmartPMS (gestione PMS, fatturazione)

#### SMARTPMS
**Includere** ticket che nel titolo o nei primi 800 chars contengono:
- "smartpms", "smart pms", "ciaobooking", "ciaoBooking", "pms", "prenotazioni", "overbooking", "alloggiati web", "alloggiati", "ROSS1000", "check-in online", "check in online", "fatturazione", "extra", "folio", "city tax", "tourist tax"

**Escludere** ticket che riguardano esclusivamente:
- Smartchat / AI messaggi
- Smartpricing (prezzi)
- Smartconnect (sync channel manager, non prenotazioni)

### 2.3 Classificazione per cluster
Raggruppare i ticket in cluster tematici in base al prodotto selezionato.

#### Cluster SMARTPRICING
- Calendario prezzi non aggiornato / ritardi
- Raccomandazioni fuori mercato / anomalie algoritmo
- Minimum stay / restrizioni non applicate
- Occupancy / disponibilità incongruente
- Integrazione con PMS / channel manager (lato pricing)
- Accesso account / credenziali
- Feature Request
- Altro

#### Cluster SMARTCHAT
- Messaggi non recapitati (Booking.com / Airbnb)
- AI risponde con info sbagliate / confonde proprietà
- Piattaforma lenta / non accessibile / down
- Chat non associata a proprietà o prenotazione
- Invio file/video WhatsApp non funzionante
- Accesso account / credenziali
- Gestione KB / proprietà in Smartchat
- Anomalie AI varie (numeri telefono, checkout, indirizzi)
- Feature Request

#### Cluster SMARTCONNECT
- Disponibilità non sincronizzata su OTA
- Doppia prenotazione / overbooking da mancata sync
- Prezzi non aggiornati su channel
- Errori connessione / timeout API channel manager
- Nuova connessione OTA / setup
- Accesso account / credenziali
- Feature Request
- Altro

#### Cluster SMARTPMS
- Prenotazioni non sincronizzate / mancanti
- Overbooking / conflitti prenotazione
- Check-in online non funzionante
- Alloggiati Web / ROSS1000 — invio non riuscito
- City tax / extra / folio — anomalie
- Fatturazione / emissione documenti
- Accesso account / credenziali
- Feature Request
- Altro

Per ogni cluster: conteggio ticket, lista ticket chiave, status (RISOLTO / APERTO / PARZIALE).

### 2.4 Pattern Jira per prodotto
Cercare nel testo del ticket i seguenti pattern (chiavi reali da Jira Smartness):

| Prodotto | Chiave Jira | Pattern regex |
|----------|-------------|---------------|
| Smartpricing | `PRD` | `PRD-\d+` |
| Smartchat | `SMARTCHAT` | `SMARTCHAT-\d+` |
| Smartconnect | `PAY` | `PAY-\d+` |
| SmartPMS | `CIAOB` | `CIAOB-\d+` |

---

## 3. ANALISI ISSUE APERTE

Per ogni ticket potenzialmente aperto, leggere l'**ultimo commento** e il **penultimo** per determinare:

### 3.1 Categorie di stato
| Stato | Descrizione | Badge |
|-------|-------------|-------|
| `cliente` | Il cliente deve ancora rispondere a un agente (ha chiesto dati, screenshot, video) | Blu — "Attende risposta: [nome agente]" |
| `support` | Support/Tech deve comunicare esito al cliente, o Tech deve confermare a Support | Arancione — "Support/Tech deve [azione]" |
| `silente` | Cliente non ha risposto dopo richiesta dell'agente — ticket fermo | Grigio — "Ticket silente — cliente non ha risposto" |
| `chiuso_visibilita` | Ticket formalmente chiuso ma incluso per visibilità (problematica recente o rilevante) | Grigio con ✓ — "Chiuso — incluso per visibilità" |

### 3.2 Regole di assegnazione agente
- Leggere chi ha scritto l'ultimo messaggio del Support nel thread
- **Non assumere il nome dell'agente dal contesto della chat** — verificare sempre dall'ultimo commento del ticket
- Se il bot interno linka un Jira (pattern del prodotto selezionato), riportarlo
- Se il Jira è marcato Done ma il ticket Zendesk è ancora in sospeso, specificare il flusso atteso
- Ticket gestiti da un prodotto diverso da quello selezionato: escludere dalla lista issue

### 3.3 Flusso multi-step da segnalare
Quando il percorso verso la chiusura richiede più step interni, descriverlo esplicitamente nella card:
> "Jira risolto lato Tech — Tech deve confermare a [agente] che il fix ha effettivamente risolto il problema. Solo dopo [agente] potrà comunicare l'esito al cliente."

---

## 4. STRUTTURA PRESENTAZIONE (8 slide standard)

| # | Titolo | Contenuto |
|---|--------|-----------|
| 1 | Cover | KPI (ticket totali, bug confermati, crisi dichiarate, issue aperte), periodo, **nome prodotto selezionato** |
| 2 | Profilo cliente | Chi è, chi contatta il Support, come usa il prodotto |
| 3 | Cluster principali | Griglia 2×3 con badge RISOLTO/APERTO/PARZIALE per ogni cluster |
| 4 | Timeline escalation | Timeline verticale con crisi dichiarate (marcate 🔴), eventi chiave |
| 5 | Distribuzione | Bar chart orizzontale con conteggio ticket per categoria |
| 6 | Il problema reale | Slide scura con citazioni dirette dai ticket + 3 insight numerati |
| 7 | Issue ancora aperte | Lista ticket con: numero, severità, titolo, agente in carico, descrizione, badge azione |
| 8 | Riepilogo per CSM | 5 punti operativi con cosa portare nel meeting |

Il nome del prodotto selezionato deve apparire sulla **Cover (slide 1)** e nel titolo del file di output.

Adattare il numero di slide se il cliente ha storico più corto (rimuovere slide 5 se pochi dati) o se non ci sono issue aperte (slide 7 può mostrare "Nessuna issue aperta").

---

## 5. BRAND SMARTNESS

### 5.1 Palette colori
```javascript
const C = {
  navy:    "1E1B55",   // bg slide scure (cover, "Il problema reale", riepilogo)
  navyDk: "141240",   // bg slide riepilogo (più scuro)
  ice:    "7C6FF0",   // viola Smartness — accent principale (badge, numeri KPI su scuro)
  white:  "FFFFFF",
  off:    "EEEEFF",   // lavanda chiara — bg slide di contenuto
  red:    "D63B2F",   // errori, badge APERTO, severità ALTO
  redLt:  "FDE8E6",
  amber:  "C07A10",   // badge PARZIALE, severità MEDIO, azioni "support"
  amberLt:"FEF3DC",
  green:  "2E7D32",   // badge RISOLTO
  greenLt:"E8F5E9",
  gray:   "7A7A9A",   // testo secondario, badge silente
  grayLt: "F0EFFF",   // lavanda molto chiara — bg card contenuto
  text:   "2D2A6E",   // testo primario su sfondi chiari (indigo scuro)
  muted:  "6B6890",   // testo secondario su sfondi chiari
};
```

### 5.2 Slide scure vs chiare
- **Scure** (`C.navy` bg): Cover (slide 1), "Il problema reale" (slide 6), Riepilogo CSM (slide 8)
  - Testo titoli: `C.white`
  - Label sezione (supra-titolo): `"C8C6FF"` (lavanda chiara)
  - Numeri KPI: `C.white`
  - Card interne: `"2D2A6E"` (leggermente più chiaro del bg)
- **Chiare** (`C.off` o `C.white` bg): tutte le altre
  - Testo: `C.text` (`2D2A6E`)
  - Label sezione: `C.navy`

### 5.3 Font e dimensioni
- Titolo principale slide: 24-28px bold
- Supra-titolo (label sezione): 10px bold, charSpacing: 2
- Body text: 9.5-11px
- Card title: 11-12px bold
- Badge: 8-9px bold, centrato

### 5.4 Layout generale
```javascript
pres.layout = "LAYOUT_16x9";  // sempre
// Shadow standard
const makeShadow = () => ({ type: "outer", color: "000000", blur: 8, offset: 2, angle: 45, opacity: 0.08 });
```

---

## 6. SLIDE 7 — ISSUE APERTE (dettaglio implementativo)

Ogni riga ha altezza `ROW_H = 0.80"`, gap `ROW_GAP = 0.05"`, partendo da `y = 1.38"`.

Per ogni ticket:
```
[#XXXXX badge] [SEVERITÀ badge] [Titolo bold]           In carico: Nome Cognome (corsivo)
               [Descrizione — una/due righe]
[→ Badge azione colorato]
```

Colori riga per tipo azione:
- `cliente`: bg `F0EFFF`, badge bg `7C6FF0` (viola), testo badge `FFFFFF`
- `support`: bg `FEF3DC`, badge bg `FAC775` (ambra), testo badge `854F0B`
- `silente`: bg `F0EFFF`→`grayLt`, badge bg `D3D1C7`, testo badge grigio
- `chiuso_visibilita`: bg `grayLt`, badge bg `D3D1C7`, testo badge grigio con ✓

Legenda in alto con 3 pill: CLIENTE DEVE RISPONDERE | SUPPORT DEVE COMUNICARE ESITO | TICKET SILENTE

---

## 7. CHECK OBBLIGATORI PRIMA DI GENERARE

Prima di scrivere il codice pptxgenjs, rispondere a queste domande leggendo i ticket:

1. **Prodotto confermato**: ricordare sempre il prodotto scelto nello step 0 — non filtrare per il prodotto sbagliato.
2. **Nomi agenti corretti**: chi ha davvero firmato gli ultimi messaggi? Non assumere mai nomi dal contesto della conversazione con l'utente.
3. **Jira presenti**: cercare nel testo i pattern Jira del prodotto selezionato e annotarli per ogni ticket aperto.
4. **Flusso chiusura**: per ogni issue aperta, il percorso verso la chiusura è diretto (1 step) o multi-step (Tech → Agente → Cliente)?
5. **Ticket "chiuso per visibilità"**: ci sono ticket formalmente chiusi ma così recenti o significativi da includere comunque nella slide?
6. **Citazioni per slide 6**: trovare 1-2 frasi verbatim del cliente che esprimano frustrazione o soddisfazione.

---

## 8. WORKFLOW COMPLETO

```
0. ask_product() → attendere conferma utente: Smartpricing | Smartchat | Smartconnect | SmartPMS
1. parse_tickets(html_files) → lista ticket deduplicate
2. filter_by_product(tickets, product) → solo ticket del prodotto selezionato
3. classify_clusters(tickets, product) → cluster con conteggio e status (cluster specifici per prodotto)
4. identify_open_issues(tickets) → issue aperte con owner, Jira (pattern per prodotto), flusso
5. extract_quotes(tickets) → citazioni per slide 6
6. build_timeline(tickets) → eventi chiave ordinati per data
7. generate_pptx(all_above, product) → file .pptx (nome prodotto in cover e nome file)
8. qa_render(pptx) → PDF → immagini → controllo visivo ogni slide
9. present_files(pptx)
```

### QA obbligatoria (step 8)
Convertire sempre il .pptx in PDF con LibreOffice e renderizzare le slide come immagini:
```bash
python /mnt/skills/public/pptx/scripts/office/soffice.py --headless --convert-to pdf output.pptx
pdftoppm -jpeg -r 150 output.pdf slide
```
Verificare visivamente almeno: cover, slide cluster, slide issue aperte, slide riepilogo CSM.
Se testo è troncato o sovrapposto: ridurre font size o accorciare testo, non aumentare row height oltre i limiti della slide.

---

## 9. AGGIORNAMENTI PARZIALI

Quando l'utente chiede di modificare solo una slide o una riga:
- Modificare solo il blocco JS corrispondente nel file sorgente
- Rigenerare solo le slide modificate per QA (usare `-f N -l N` in pdftoppm)
- Non rigenerare l'intero deck se non necessario

---

## 10. ERRORI COMUNI DA EVITARE

| Errore | Correzione |
|--------|------------|
| Procedere senza chiedere il prodotto | Chiedere SEMPRE nello step 0 prima di qualsiasi analisi |
| Filtrare per il prodotto sbagliato | Rileggere la risposta utente allo step 0 prima del filtro |
| Assumere il nome dell'agente dal contesto chat | Leggere sempre l'ultimo commento del ticket |
| Usare cluster sbagliati per il prodotto | Ogni prodotto ha la sua lista cluster — vedere sezione 2.3 |
| Pattern Jira sbagliati | Usare i pattern della tabella 2.4 per il prodotto selezionato |
| Testo che sovrasta il badge azione | Accorciare description a max 2 righe (28px height) |
| Colori invertiti (testo scuro su bg scuro) | Slide navy: testo white/C8C6FF; slide chiare: testo 2D2A6E |
| Nome prodotto assente in cover e nome file | Cover slide 1 e nome file devono riportare il prodotto |
| Nome cliente corretto | Leggere il nome esatto dal dominio email nei ticket |

---

## 11. NOME FILE OUTPUT

```
/mnt/user-data/outputs/NomeCliente_{Prodotto}_Report_CSM.pptx
```

Dove `{Prodotto}` è uno di: `Smartpricing`, `Smartchat`, `Smartconnect`, `SmartPMS`

---

## 12. LIBRERIE E TOOL

```javascript
// Generazione PPTX
const pptxgen = require("pptxgenjs");  // npm install -g pptxgenjs

// Parsing HTML (Python)
from bs4 import BeautifulSoup  // pip install beautifulsoup4
```
