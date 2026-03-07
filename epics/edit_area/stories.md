# Stories — Epic 05 — Add / Edit Area

## Sequenza di implementazione

```
story-05-01 → Form Add Area: layout con 3 campi e CTA
story-05-02 → Pill selector tipo area (Health / Study / Reduce / Finance)
story-05-03 → Stepper frequenza 1–7
story-05-04 → Integrazione Supabase: creazione area → redirect Dashboard
story-05-05 → Modalità Edit: form pre-compilato + "Save changes" + "Archive area"
```

> **Dipendenze:** Richiede Epic 02 (Dashboard) completato. La nuova area creata deve apparire come card nella Dashboard.

---

## story-05-01 — Form Add Area

BetonMe è un'app di osservazione del benessere. Costruisci la schermata Add Area (route `/areas/new`).

**Cosa mostra:**
- Header: `"Add area"` con back navigation verso `/`
- Form con esattamente 3 campi:
  1. **Nome area** — input testo, placeholder: `"e.g. Morning walk"`
  2. **Tipo** — pill selector (costruito nella story-05-02)
  3. **Frequenza settimanale** — stepper (costruito nella story-05-03)
- CTA full-width in fondo: `"Start observing"`

**Behavior:**
- CTA `"Start observing"` disabilitata se nome vuoto o tipo non selezionato
- La frequenza ha sempre un valore valido (default 7), quindi non blocca la CTA
- Nome con soli spazi → validazione fallisce, trattato come nome vuoto
- Tipo non selezionato al tap su CTA → errore inline: `"Please select a type"`

**Note:**
- Mai usare: `"Create goal"`, `"Set target"`, `"Begin challenge"`, `"Track habit"`
- Il form non ha campi aggiuntivi oltre i 3 specificati

---

## story-05-02 — Pill selector tipo area

Continua Epic 05 di BetonMe. Costruisci il componente pill selector per il tipo area.

**4 pill selezionabili (una alla volta):**
- `"Health"` — stile teal
- `"Study"` — stile grigio
- `"Reduce"` — stile warm
- `"Finance"` — stile scuro

**Behavior:**
- Solo un tipo selezionabile alla volta
- Pill selezionata: bordo e background attivi secondo il brand system
- Pill non selezionata: stile muted
- La selezione abilita la CTA (se anche il nome è compilato)

**Note:**
- I colori di ogni pill sono definiti nel brand system (`<AreaTypePill>`) — usare esattamente quelli

---

## story-05-03 — Stepper frequenza

Continua Epic 05 di BetonMe. Costruisci lo stepper per la frequenza settimanale.

**Cosa mostra:**
- Label: `"How many days per week?"`
- Stepper: bottone `–` · valore numerico centrale · bottone `+`
- Range: 1–7, default 7

**Behavior:**
- Il bottone `–` è disabilitato quando il valore è 1
- Il bottone `+` è disabilitato quando il valore è 7
- Ogni controllo ha `min-height: 44px` per accessibilità touch

---

## story-05-04 — Integrazione Supabase: creazione area

Continua Epic 05 di BetonMe. Implementa il salvataggio dell'area su Supabase.

**Al tap su `"Start observing"`:**
1. CTA mostra spinner + opacity ridotta (stato loading)
2. Inserimento record in `areas`: `{ user_id, name, type, frequency_per_week }` (`archived_at` = null)
3. Redirect immediato a `/` (Dashboard)

**Cosa NON mostrare:**
- Nessuna schermata di conferma o celebrazione
- Nessun toast del tipo "Area created!"
- Nessun testo motivazionale
- La nuova card appare silenziosamente nella Dashboard

**Edge case:**
- Errore di rete → CTA torna abilitata, messaggio di errore inline non bloccante

---

## story-05-05 — Modalità Edit

Continua Epic 05 di BetonMe. Aggiungi la modalità modifica area (route `/areas/:id/edit`).

**Come si accede:**
- Dal link testuale `"Edit area"` nella schermata Area Detail

**Differenze rispetto alla modalità Add:**
- Header: `"Edit area"` (invece di `"Add area"`)
- I 3 campi sono pre-compilati con i dati dell'area esistente
- CTA: `"Save changes"` (invece di `"Start observing"`)
- In fondo alla schermata: link testuale `"Archive area"` (non un bottone)

**Behavior "Save changes":**
- Aggiornamento record in `areas` con i nuovi valori
- Redirect all'Area Detail aggiornata (`/areas/:id`)
- Nessun messaggio di conferma

**Behavior "Archive area":**
- Nessun modale di conferma in MVP — azione immediata
- Aggiornamento record: `archived_at = now()`
- L'area scompare dalla Dashboard e dalla lista aree
- Redirect a `/` (Dashboard)
- L'area non viene eliminata dal database

**Note:**
- Modifica del tipo su area con dati storici: consentita, i dati esistenti rimangono
