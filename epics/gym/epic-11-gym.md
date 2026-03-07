# Epic 11 — Gym Card (Scheda Palestra)

## Obiettivo
Offrire una sezione dedicata al log degli allenamenti all'interno dell'area "Gym" (EN) / "Palestra" (IT), consentendo all'utente di registrare gli esercizi della sessione e osservare i progressi nel tempo.

---

## Behavior

Quando un utente crea un'area di tipo `health` con nome `Gym` (EN) o `Palestra` (IT) — case insensitive — l'Area Detail di quell'area mostra una sezione aggiuntiva: la **Gym Card / Scheda Palestra**.

La Gym Card è una sezione di log strutturato che si aggiunge sotto il grafico traiettoria e l'heatmap standard dell'Area Detail. Non sostituisce il check-in normale — il check-in binario rimane il meccanismo principale di tracciamento per il grafico.

---

## Rilevamento automatico

| Condizione | Comportamento |
|---|---|
| `area.type == "health"` AND `area.name` == `"gym"` o `"palestra"` (case insensitive) | Mostra la sezione Gym Card |
| Qualsiasi altra area | Sezione Gym Card non mostrata |

> Non richiedere all'utente di "attivare" la Gym Card — appare automaticamente.

---

## Layout Area Detail con Gym Card

```
┌─────────────────────────────┐
│ ← Palestra / Gym   [edit]   │
├─────────────────────────────┤
│  30d │ 90d │ 365d            │
├─────────────────────────────┤
│   [Grafico — 60vh]          │
├─────────────────────────────┤
│   Ultimi 30 giorni          │
│   □□■□■■□■□                 │  ← CalendarHeatmap
├─────────────────────────────┤
│   [Log today / Registra oggi] │
├─────────────────────────────┤
│                             │
│   ── Scheda / Gym Card ──   │  ← Sezione aggiuntiva
│                             │
│   Sessione di oggi          │
│   ┌─────────────────────┐   │
│   │ Esercizio     [+]   │   │
│   │ Squat               │   │
│   │  4 × 80kg           │   │
│   │ Bench Press         │   │
│   │  3 × 70kg           │   │
│   └─────────────────────┘   │
│   [Aggiungi esercizio]      │
│                             │
│   Storico sessioni          │
│   [lista collassabile]      │
│                             │
└─────────────────────────────┘
```

---

## Struttura della sessione

Una sessione è legata al check-in del giorno (`checkin.date`). Per ogni sessione l'utente aggiunge uno o più esercizi.

### Esercizio

| Campo | Tipo | Obbligatorio |
|---|---|---|
| Nome esercizio | Testo libero | Sì |
| Serie | Numero intero | Sì |
| Ripetizioni | Numero intero | Sì |
| Peso (kg) | Numero decimale | No |
| Note | Testo libero | No |

> Il peso è opzionale per chi fa esercizi a corpo libero.

---

## Flussi

### Aggiunta esercizio
1. L'utente tappa `[Aggiungi esercizio / Add exercise]`
2. Appare un bottom sheet con i campi: nome, serie, rip, peso (opzionale), note (opzionale)
3. L'utente compila e tappa `Salva / Save`
4. L'esercizio appare nella lista sessione di oggi
5. Il check-in del giorno viene automaticamente marcato come completato (se non già fatto)

### Modifica esercizio
1. L'utente tappa su un esercizio già inserito
2. Il bottom sheet si apre precompilato
3. L'utente modifica e salva — oppure tappa `Elimina / Delete` per rimuovere

### Storico sessioni
1. Sotto la sessione di oggi, una sezione `Storico / History` mostra le sessioni precedenti
2. Ogni sessione è un item collassato: data + numero esercizi
3. Tap → espande la lista degli esercizi di quella sessione (read-only)

---

## Database

Nuove tabelle:

```
gym_sessions  → id, area_id, user_id, date (FK a checkin.date)
gym_exercises → id, session_id, name, sets, reps, weight_kg (nullable), notes (nullable), order
```

---

## Copy UI

| Elemento | IT | EN |
|---|---|---|
| Titolo sezione | `Scheda Palestra` | `Gym Card` |
| Sottotitolo sessione | `Sessione di oggi` | `Today's session` |
| CTA aggiungi | `Aggiungi esercizio` | `Add exercise` |
| Label nome | `Esercizio` | `Exercise` |
| Label serie | `Serie` | `Sets` |
| Label rip | `Ripetizioni` | `Reps` |
| Label peso | `Peso (kg)` | `Weight (kg)` |
| Label note | `Note` | `Notes` |
| Placeholder peso | `Opzionale` | `Optional` |
| Salva | `Salva` | `Save` |
| Elimina esercizio | `Elimina` | `Delete` |
| Storico | `Storico sessioni` | `Session history` |
| Empty state (nessun esercizio oggi) | `Nessun esercizio ancora. Inizia ad aggiungere.` | `No exercises yet. Start adding.` |

---

## Stati UI

| Stato | Comportamento |
|---|---|
| Nessuna sessione oggi | Empty state con CTA |
| Sessione in corso | Lista esercizi + CTA aggiungi |
| Bottom sheet aperto | Overlay, campi di input, Salva/Cancel |
| Storico vuoto | Sezione nascosta |
| Peso non inserito | Non mostrato nella card esercizio |

---

## Edge Case

- L'utente rinomina l'area da "Palestra" a "Corsa" → la Gym Card scompare (ma i dati rimangono nel DB)
- L'utente riporta il nome a "Palestra" → la Gym Card riappare con lo storico intatto
- Check-in già fatto manualmente → aggiunta esercizio non cambia lo stato del check-in
- Esercizio con peso 0 → trattato come "a corpo libero", non mostrare "0 kg"

---

## Acceptance Criteria

- [ ] La sezione Gym Card appare solo per aree con nome `gym` o `palestra` (case insensitive) e tipo `health`
- [ ] L'utente può aggiungere un esercizio con nome, serie, rip (peso opzionale)
- [ ] L'esercizio appare nella lista della sessione di oggi dopo il salvataggio
- [ ] L'aggiunta del primo esercizio completa automaticamente il check-in del giorno
- [ ] Lo storico mostra le sessioni precedenti collassate con espansione al tap
- [ ] Il bottom sheet ha i campi specificati e si chiude correttamente dopo il salvataggio
- [ ] Il peso "0" non viene mostrato — viene mostrato solo se > 0
- [ ] Le label seguono la lingua selezionata (Epic 08)

---

## Dipendenze

- Epic 03 (Check-in) — il check-in si completa automaticamente al primo esercizio
- Epic 04 (Area Detail) — la Gym Card è una sezione aggiuntiva nell'Area Detail
- Epic 08 (i18n) — tutte le label in IT/EN

---

## Stories

- `story-11-01` — Rilevamento automatico area Gym/Palestra e rendering sezione Gym Card
- `story-11-02` — Aggiunta esercizio via bottom sheet e lista sessione di oggi
- `story-11-03` — Modifica ed eliminazione esercizio
- `story-11-04` — Storico sessioni collassabile
