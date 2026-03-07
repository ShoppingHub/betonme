# Epic 05 — Add / Edit Area

## Obiettivo
Permettere all'utente di creare o modificare un'area di vita in meno di 10 secondi, con un form minimale a 3 campi e un tono neutro e osservativo.

---

## Behavior

Il form ha esattamente 3 campi. Niente di più. La creazione non genera schermate di celebrazione — al completamento l'utente torna alla Dashboard e vede la nuova card.

La stessa schermata è usata sia per la creazione (Add) che per la modifica (Edit) — il titolo dell'header cambia di conseguenza.

---

## Flussi

### Creazione Area
1. L'utente tappa "Add your first area" (empty state) o un pulsante "Add area"
2. Vede il form con 3 campi
3. Compila nome, tipo, frequenza
4. Tappa "Start observing"
5. L'area viene creata su Supabase
6. Redirect alla Dashboard — la nuova card è visibile
7. Nessuna schermata di celebrazione

### Modifica Area
1. L'utente tappa "Edit area" dall'Area Detail
2. Il form si apre pre-compilato con i dati dell'area
3. L'utente modifica i campi desiderati
4. Tappa "Save changes"
5. Redirect all'Area Detail aggiornata

### Archiviazione Area (da Edit)
1. In modalità Edit, link testuale "Archive area" in fondo
2. Nessuna cancellazione hard in MVP — l'area viene archiviata (`archived_at = now()`)
3. Le aree archiviate non compaiono nella Dashboard

---

## Campi del Form

| Campo | Tipo | Placeholder / Opzioni | Default |
|---|---|---|---|
| Nome area | Text input | `"e.g. Morning walk"` | vuoto |
| Tipo | 4 pill selector | Health · Study · Reduce · Finance | nessuno |
| Frequenza / settimana | Stepper 1–7 | — | 7 |

---

## Specifiche Pill Tipo

| Tipo | Stile |
|---|---|
| Health | `bg-[#7DA3A0]/20 text-[#7DA3A0] border border-[#7DA3A0]/40` |
| Study | `bg-[#8C9496]/20 text-[#B9C0C1] border border-[#8C9496]/40` |
| Reduce | `bg-[#BFA37A]/20 text-[#BFA37A] border border-[#BFA37A]/40` |
| Finance | `bg-[#1F4A50] text-[#EAEAEA] border border-[#7DA3A0]/30` |

Tutte le pill: `rounded-full px-3 py-1 text-[14px] font-medium border`  
Solo un tipo selezionabile alla volta.

---

## CTA

**Creazione:** `"Start observing"` — `bg-[#7DA3A0] text-[#0F2F33]`, full-width, `min-h-[44px]`  
**Modifica:** `"Save changes"` — stesso stile  
**Disabilitato:** se nome vuoto o tipo non selezionato

> ⚠️ Mai usare: "Create goal", "Set target", "Begin challenge", "Track habit"

---

## Stati UI

| Stato | Comportamento |
|---|---|
| Form vuoto | CTA disabilitata |
| Nome compilato, tipo selezionato | CTA abilitata |
| Loading (submit) | Spinner inline nel bottone, CTA disabilitata |
| Errore validazione | Messaggio inline sotto il campo, `text-[#E24A4A] text-sm` |
| Successo | Redirect — nessun messaggio |

---

## Edge Case

- Nome area con solo spazi → validazione fallisce (richiede almeno 1 carattere non-spazio)
- Tipo non selezionato al submit → errore inline "Please select a type"
- Modifica del tipo di un'area con dati storici → consentito, i dati storici rimangono
- Frequenza = 1 → ammesso, lo stepper non va sotto 1
- Frequenza = 7 → ammesso, lo stepper non va sopra 7

---

## Acceptance Criteria

- [ ] Il form ha esattamente 3 campi (nome, tipo, frequenza)
- [ ] La CTA è disabilitata senza nome e tipo compilati
- [ ] Al submit, l'area è creata e la Dashboard mostra la nuova card
- [ ] Nessuna schermata di celebrazione dopo la creazione
- [ ] In modalità Edit, il form è pre-compilato
- [ ] Il tipo selezionato ha lo stile visivo corretto (pill attiva)
- [ ] Lo stepper frequenza è limitato tra 1 e 7
- [ ] Il link "Archive area" in modalità Edit archivia l'area senza cancellarla

---

## Copy UI

| Elemento | Stringa |
|---|---|
| Header (creazione) | `"Add area"` |
| Header (modifica) | `"Edit area"` |
| Placeholder nome | `"e.g. Morning walk"` |
| Label frequenza | `"How many days per week?"` |
| CTA creazione | `"Start observing"` |
| CTA modifica | `"Save changes"` |
| Link archiviazione | `"Archive area"` |

---

## Note UI

- Brand tokens: `brand-system/betonme_brand_system_lovable.md`
- `<AreaTypePill>` per i selettori tipo
- Stepper frequenza: frecce +/– con valore centrale, `min-h-[44px]` per ogni controllo

---

## Stories

- `story-05-01` — Layout form con 3 campi e CTA
- `story-05-02` — Pill selector per tipo area
- `story-05-03` — Stepper frequenza 1–7
- `story-05-04` — Integrazione Supabase: creazione e modifica area
- `story-05-05` — Archiviazione area
