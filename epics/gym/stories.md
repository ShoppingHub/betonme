# Stories — Epic 11 — Gym Card (Scheda Palestra)

## Sequenza di implementazione — ✅ tutte completate

```
story-11-01 → Rilevamento area Gym/Palestra e rendering sezione Gym Card      ✅
story-11-02 → Aggiunta esercizio via bottom sheet e lista sessione di oggi    ✅
story-11-03 → Modifica ed eliminazione esercizio                              ✅
story-11-04 → Storico sessioni collassabile                                   ✅
```

> **Dipendenze:** Richiede Epic 03 (Check-in) per il completamento automatico. Epic 04 (Area Detail) per la posizione della sezione. Epic 08 (i18n) per i label.

---

## story-11-01 — Rilevamento e rendering Gym Card

BetonMe è un'app di osservazione del benessere. Aggiungi una sezione "Scheda Palestra / Gym Card" all'Area Detail, visibile automaticamente quando l'area si chiama "Gym" o "Palestra" (case insensitive) ed è di tipo `health`.

**Quando appare:**
- `area.type === "health"` AND `area.name.toLowerCase()` è `"gym"` o `"palestra"`
- In tutti gli altri casi la sezione non è presente

**Dove appare:**
- Sotto il CalendarHeatmap e il bottone "Log today / Registra oggi", all'interno dell'Area Detail
- Titolo sezione: `"Scheda Palestra"` (IT) / `"Gym Card"` (EN)
- Separata dal contenuto precedente da un divider

**Struttura iniziale della sezione (nessun esercizio oggi):**
- Sottotitolo: `"Sessione di oggi"` (IT) / `"Today's session"` (EN)
- Empty state: `"Nessun esercizio ancora. Inizia ad aggiungere."` (IT) / `"No exercises yet. Start adding."` (EN)
- CTA: `"Aggiungi esercizio"` (IT) / `"Add exercise"` (EN)

**Nuove tabelle DB necessarie:**
- `gym_sessions`: id, area_id, user_id, date
- `gym_exercises`: id, session_id, name, sets (int), reps (int), weight_kg (decimal, nullable), notes (text, nullable), order (int)

---

## story-11-02 — Aggiunta esercizio via bottom sheet

Continua Epic 11 di BetonMe. Implementa il flusso di aggiunta esercizio.

**Tap su CTA `"Aggiungi esercizio / Add exercise"`:**
- Appare un bottom sheet (overlay dal basso) con i seguenti campi:
  - `Esercizio / Exercise` — input testo, obbligatorio
  - `Serie / Sets` — input numerico intero, obbligatorio
  - `Ripetizioni / Reps` — input numerico intero, obbligatorio
  - `Peso (kg) / Weight (kg)` — input numerico decimale, opzionale, placeholder `"Opzionale / Optional"`
  - `Note / Notes` — input testo multi-line, opzionale
- CTA: `"Salva / Save"` (disabilitata se i campi obbligatori sono vuoti)
- CTA: `"Annulla / Cancel"` → chiude il bottom sheet senza salvare

**Al salvataggio:**
1. Crea o recupera la `gym_session` di oggi per quest'area
2. Aggiunge il record in `gym_exercises` con i dati inseriti
3. Il bottom sheet si chiude
4. L'esercizio appare in cima alla lista della sessione di oggi
5. Se è il primo esercizio del giorno → completa automaticamente il check-in del giorno (come se l'utente avesse tappato "Log today")

**Visualizzazione esercizio in lista:**
- Nome esercizio (bold)
- Sotto: `"4 × 80kg"` (serie × peso) oppure `"4 serie × 10 rip"` se nessun peso (non mostrare "0 kg")
- Note in piccolo se presenti

---

## story-11-03 — Modifica ed eliminazione esercizio

Continua Epic 11 di BetonMe. Implementa la modifica e l'eliminazione di un esercizio.

**Tap su un esercizio in lista:**
- Il bottom sheet si apre precompilato con i dati dell'esercizio
- Stessa struttura di story-11-02
- CTA aggiuntiva in fondo: `"Elimina / Delete"` (testo colore errore `#E24A4A`)

**Al salvataggio modifica:**
- Aggiorna il record in `gym_exercises`
- Bottom sheet si chiude
- Lista aggiornata

**Al tap su Elimina:**
- Nessun modale di conferma (azione a basso rischio, dati facilmente ricreabili)
- Il record viene eliminato immediatamente
- La lista si aggiorna
- Se l'esercizio eliminato era l'unico → torna all'empty state (ma il check-in rimane completato)

---

## story-11-04 — Storico sessioni collassabile

Continua Epic 11 di BetonMe. Aggiungi la sezione storico sessioni sotto la sessione di oggi.

**Cosa mostra:**
- Titolo: `"Storico sessioni"` (IT) / `"Session history"` (EN)
- Lista di sessioni precedenti (esclusa quella di oggi), ordinate dalla più recente
- Ogni sessione collassata: data formattata (es. `"Lun 3 Mar"`) + numero esercizi (es. `"4 esercizi / 4 exercises"`)
- Tap sulla sessione → espande la lista degli esercizi (read-only, stessa visualizzazione della sessione di oggi)
- Solo una sessione può essere espansa alla volta

**Empty state storico:**
- Se non ci sono sessioni precedenti → la sezione "Storico sessioni" non viene mostrata

**Edge case:**
- Sessione con 1 esercizio → `"1 esercizio / 1 exercise"` (singolare)
- Sessione con 0 esercizi (anomalia) → non mostrata nello storico
