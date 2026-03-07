# Stories — Epic 04 — Area Detail

## Sequenza di implementazione

```
story-04-01 → Layout Area Detail: header, back nav, time range, grafico 60vh
story-04-02 → CalendarHeatmap 30 giorni
story-04-03 → Score condizionale (visibile solo se settings_score_visible = true)
story-04-04 → Transizione Framer Motion dalla Dashboard
```

> **Dipendenze:** Richiede Epic 02 (Dashboard), Epic 03 (Check-in), Epic 05 (Add/Edit Area) completati.

---

## story-04-01 — Layout Area Detail

BetonMe è un'app di osservazione del benessere. Costruisci la schermata Area Detail (route `/areas/:id`).

**Cosa mostra:**
- Header: back navigation (`←`) + nome dell'area + link testuale `"Edit area"` a destra (navigazione a `/areas/:id/edit`)
- Selettore time range: stessa logica della Dashboard (30d / 90d / 365d)
- Grafico Recharts `<LineChart>` con `type="monotone"`, altezza 60vh — stesso stile della Dashboard ma più grande
- `<CheckInButton>` (già costruito nella story-03-01)
- Link testuale `"Edit area"` in fondo alla schermata (colore secondario, non un bottone)

**Behavior:**
- Tap su `←` → back navigation verso `/`
- Time range selector aggiorna il grafico solo di questa area
- Il check-in funziona esattamente come sulla Dashboard

**Stati:**
- Loading: skeleton grafico (h-60vh, animate-pulse)
- Empty state (nessun dato): icona TrendingUp (Lucide) muted + testo `"Keep observing. Your trajectory takes shape over days."` — nessuna CTA

---

## story-04-02 — CalendarHeatmap 30 giorni

Continua Epic 04 di BetonMe. Aggiungi il `<CalendarHeatmap>` sotto il grafico.

**Cosa mostra:**
- Riga orizzontale di celle quadrate per gli ultimi 30 giorni (oggi incluso)
- Ogni cella rappresenta un giorno

**3 stili di cella:**
- Giorno **completato** (check-in presente): cella evidenziata teal
- Giorno **mancato** (nessun check-in, data passata): cella evidenziata warm/amber
- Giorno **futuro**: cella muted

**Behavior:**
- Scroll orizzontale se le celle non entrano in una riga
- Nessuna label sui giorni
- Nessun tooltip
- Nessun contatore di streak o "best streak"
- Nessuna fiamma o icona celebrativa per giorni consecutivi
- Loading: riga di celle skeleton animate-pulse

---

## story-04-03 — Score condizionale

Continua Epic 04 di BetonMe. Aggiungi la visualizzazione condizionale dello score.

**Behavior:**
- Il punteggio numerico (`cumulative_score`) è visibile SOLO se `settings_score_visible = true` nella tabella `users` per l'utente corrente
- Se visibile: label `"Trajectory score"` + valore numerico, posizionato sotto il CalendarHeatmap
- Se nascosto (default): questa sezione non occupa spazio nella schermata

**Note:**
- Questo è il SOLO punto dell'app dove un numero appare come score
- Il numero non ha unità di misura, colori, frecce o giudizi associati

---

## story-04-04 — Transizione Framer Motion dalla Dashboard

Continua Epic 04 di BetonMe. Aggiungi la transizione animata dalla Dashboard all'Area Detail.

**Behavior:**
- Tap su una `<TrajectoryCard>` in Dashboard → navigazione all'Area Detail con transizione fade + slide (300ms, ease-in-out) usando Framer Motion
- Back navigation: transizione inversa (slide out)
