# Stories — Epic 03 — Check-in Giornaliero

## Sequenza di implementazione

```
story-03-01 → CheckInButton: tre stati (idle / loading / completed)
story-03-02 → Integrazione Supabase: salvataggio check-in e calcolo score
story-03-03 → Animazione grafico post check-in (Framer Motion 300ms)
```

> **Dipendenze:** Richiede Epic 02 (Dashboard + TrajectoryCard) completato. Il bottone vive all'interno della `<TrajectoryCard>`.

---

## story-03-01 — CheckInButton: tre stati

BetonMe è un'app di osservazione del benessere. Costruisci il componente `<CheckInButton>` che appare in fondo a ogni `<TrajectoryCard>` nella Dashboard.

**Tre stati del bottone:**

| Stato | Label | Stile |
|---|---|---|
| Idle (non loggato oggi) | `"Log today"` | Bordo sottile, testo bianco, sfondo trasparente |
| Loading (invio in corso) | — | Spinner inline, opacity ridotta, non tappabile |
| Completed (già loggato oggi) | `"Observed"` | Sfondo teal semi-trasparente, testo teal, non tappabile |

**Behavior:**
- Il bottone ha `min-height: 44px` e occupa tutta la larghezza della card (full-width)
- Stato loading: blocca tap aggiuntivi (impedisce doppio invio)
- Stato completed: nessuna azione al tap
- Al caricamento della Dashboard, il bottone mostra già `"Observed"` se il check-in per oggi è già presente in `checkins`

**Cosa NON fare dopo il check-in:**
- Nessun toast
- Nessun popup
- Nessun messaggio di congratulazioni
- Nessuna animazione celebrativa

---

## story-03-02 — Integrazione Supabase: salvataggio check-in e score

Continua l'Epic 03 di BetonMe. Implementa la logica di backend per il check-in.

**Al tap su `"Log today"`:**
1. Transizione a stato loading
2. Inserimento record in `checkins`: `{ area_id, user_id, date: oggi (data locale), completed: true }` — constraint UNIQUE(area_id, date) impedisce duplicati
3. Calcolo e inserimento/aggiornamento in `score_daily`:

```
completed = true          → daily_score = +1.0
primo giorno mancato      → daily_score = 0.0
secondo giorno mancato    → daily_score = -0.5
terzo+ giorno mancato     → daily_score = -1.0

cumulative_score = cumulative_score(ieri) + daily_score
```

4. Transizione a stato completed

**Edge case:**
- Errore di rete → il bottone torna a stato idle, errore inline non bloccante a fondo schermata (non un popup)
- La `date` usata è sempre la data locale del dispositivo dell'utente (non UTC)

**Note:**
- Il `cumulative_score` alimenta il grafico. Non viene mai mostrato come numero (default `settings_score_visible = false`)

---

## story-03-03 — Animazione grafico post check-in

Continua l'Epic 03 di BetonMe. Aggiungi l'animazione del grafico dopo il check-in.

**Behavior:**
- Dopo il salvataggio riuscito del check-in, i dati del grafico nella `<TrajectoryCard>` si aggiornano
- Il grafico anima l'aggiornamento con Framer Motion: transizione 300ms ease-in-out
- Se il colore della linea cambia (per cambio di slope) → il cambio colore avviene con la stessa transizione

**Cosa NON fare:**
- Nessuna animazione che celebri il completamento (nessun pulse, glow, confetti)
- L'animazione è solo il naturale aggiornamento del dato nel grafico
