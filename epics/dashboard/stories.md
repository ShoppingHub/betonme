# Stories — Epic 02 — Dashboard

## Sequenza di implementazione

```
story-02-01 → Layout Dashboard: header + time range selector
story-02-02 → TrajectoryCard con grafico Recharts e colore da slope
story-02-03 → TimeRangeSelector con animazione Framer Motion
story-02-04 → Empty state
story-02-05 → Loading skeleton
```

> **Dipendenze:** Richiede Epic 00 (auth) e Epic 01 (onboarding) completati. I dati arrivano dalla tabella `areas` e `score_daily` di Supabase.

---

## story-02-01 — Layout Dashboard

BetonMe è un'app mobile-first di osservazione del benessere personale. Costruisci il layout base della Dashboard (route `/`).

**Cosa mostra:**
- Header fisso (56px): wordmark `"BetonMe"` a sinistra, icona Settings (Lucide) a destra che naviga a `/settings`
- Selettore time range sotto l'header: 3 pill — `"30d"` · `"90d"` · `"365d"` — `"30d"` attivo di default
- Area scrollabile verticale per le card delle aree
- Bottom navigation bar (già presente dallo scaffold)

**Behavior:**
- L'header è fisso, il contenuto scrolla sotto di esso
- Il time range selector rimane visibile durante lo scroll (sticky sotto l'header)
- Il tab `"Home"` nel bottom nav è attivo su questa route

---

## story-02-02 — TrajectoryCard

Continua la Dashboard di BetonMe. Costruisci il componente `<TrajectoryCard>` che rappresenta una singola area di vita.

**Cosa mostra ogni card:**
- Nome area (top-left, testo grande semibold)
- Badge tipo area (top-right): `<AreaTypePill>` con il tipo dell'area (health / study / reduce / finance)
- Grafico Recharts `<LineChart>` con `type="monotone"`, griglia solo orizzontale con opacity bassa, asse Y nascosto, asse X con date in formato breve
- Bottone `"Log today"` full-width in fondo alla card (implementato nella story-03-01)

**Logica colore della linea (calcolata sugli ultimi 7 giorni di `cumulative_score`):**
- Slope positivo (> 0.1) → colore `#7DA3A0`
- Slope negativo (< -0.1) → colore `#BFA37A` (mai rosso per trend negativi)
- Slope neutro → colore `#8C9496`

**Behavior:**
- Tap sul corpo della card (non sul bottone) → naviga all'Area Detail (`/areas/:id`)
- L'altezza della card è circa 55vh per dare prominenza al grafico
- I dati del grafico sono filtrati in base al time range selezionato nel selettore

**Edge case:**
- Nessun dato per il range selezionato → il grafico mostra una linea piatta a zero senza messaggi di errore
- Check-in già effettuato oggi → il bottone mostra già `"Observed"` al caricamento

---

## story-02-03 — TimeRangeSelector animato

Continua la Dashboard di BetonMe. Implementa l'animazione del `<TimeRangeSelector>`.

**Behavior:**
- Tap su una pill (30d / 90d / 365d) → la pill attiva si sposta con animazione Framer Motion usando `layoutId`
- Tutte le `<TrajectoryCard>` aggiornano simultaneamente i dati del grafico al cambio di range
- Transizione 300ms ease-in-out

**Note:**
- La pill attiva ha sfondo e testo evidenziati secondo il brand system
- Le pill inattive hanno stile muted

---

## story-02-04 — Empty state

Continua la Dashboard di BetonMe. Aggiungi l'empty state per utenti senza aree attive.

**Quando appare:** nessuna area nella tabella `areas` per l'utente corrente (o tutte archiviate).

**Cosa mostra:**
- Icona Eye (Lucide), 48px, colore accent teal
- Testo: `"What do you want to observe?"`
- Sottotesto: `"Add a life area to start seeing your trajectory."`
- CTA: `"Add your first area"` → naviga al form Add Area (`/areas/new`)

---

## story-02-05 — Loading skeleton

Continua la Dashboard di BetonMe. Aggiungi lo stato di loading con skeleton.

**Quando appare:** durante il caricamento iniziale dei dati da Supabase.

**Cosa mostra:**
- Una skeleton card per ogni area dell'utente (se il numero è noto) o 2 skeleton card di default
- Le skeleton card replicano la forma della `<TrajectoryCard>` (stessa altezza ~55vh)
- Animazione `animate-pulse`
- Il selettore time range rimane visibile ma non interattivo durante il loading
