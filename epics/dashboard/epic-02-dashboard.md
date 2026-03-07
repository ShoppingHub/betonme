# Epic 02 — Dashboard (Home)

## Obiettivo
Offrire all'utente una visione immediata della traiettoria di tutte le proprie aree di vita, con la possibilità di eseguire il check-in del giorno direttamente dalla schermata principale.

---

## Behavior

La Dashboard è la schermata aperta nel 95% dei casi. Il grafico **è** la schermata — non un componente accessorio. Ogni area di vita è rappresentata da una `<TrajectoryCard>` che occupa ~55vh.

L'utente scorre verticalmente tra le card. Il time range selezionato si applica a tutte le card simultaneamente.

---

## Flussi

### Visualizzazione Dashboard
1. L'utente apre l'app
2. Vede le card delle proprie aree in ordine di creazione
3. Ogni card mostra: nome area, tipo, grafico traiettoria, bottone check-in
4. Il selettore time range è fisso in cima (30d / 90d / 365d)

### Cambio Time Range
1. L'utente tappa uno dei tre pill (30d, 90d, 365d)
2. Tutte le card aggiornano il grafico simultaneamente
3. La pill attiva scorre con animazione Framer Motion (`layoutId`)

### Tap su Area Card (navigazione a Detail)
1. L'utente tappa il corpo della card (non il bottone check-in)
2. Naviga alla schermata Area Detail

---

## Layout

```
┌─────────────────────────────┐
│ BetonMe          [settings] │  ← Header 56px
├─────────────────────────────┤
│  30d  │  90d  │  365d       │  ← TimeRangeSelector
├─────────────────────────────┤
│                             │
│   [TrajectoryCard: Health]  │  ← 55vh
│   [grafico traiettoria]     │
│   [Log today]               │
│                             │
├─────────────────────────────┤
│   [TrajectoryCard: Study]   │  ← scrollabile
│   ...                       │
├─────────────────────────────┤
│  Home · Areas · Finance · ⚙ │  ← Bottom nav
└─────────────────────────────┘
```

---

## Specifiche TrajectoryCard

| Proprietà | Valore |
|---|---|
| Background | `bg-[#1F4A50] rounded-xl p-4` |
| Altezza card | `h-[55vh]` |
| Componente grafico | Recharts `<LineChart>` |
| Tipo linea | `type="monotone"` |
| Colore linea | Calcolato da slope 7 giorni → `#7DA3A0` / `#8C9496` / `#BFA37A` |
| Griglia | Solo orizzontale, `opacity-10` |
| Asse Y | Nascosto |
| Asse X | Tick date, 12px, `#B9C0C1` |
| Label area | Top-left, 18px semibold, `#EAEAEA` |
| Badge tipo | Top-right, `<AreaTypePill>` |
| Bottone check-in | Bottom, full-width, min-h 44px |

### Logica colore linea (slope 7 giorni)
```typescript
function getLineColor(slope: number): string {
  if (slope > 0.1)  return '#7DA3A0'; // positivo
  if (slope < -0.1) return '#BFA37A'; // declino — mai rosso
  return '#8C9496';                   // neutro
}
```

---

## Stati UI

### Empty State (nessuna area)
```
Icona:    Eye (Lucide), 48px, #7DA3A0
Testo:    "What do you want to observe?"
Subtesto: "Add a life area to start seeing your trajectory."
CTA:      "Add your first area" → naviga ad Add Area
```

### Loading State (caricamento iniziale)
```
Skeleton: bg-[#1F4A50] animate-pulse rounded-xl h-[55vh]
Una skeleton card per area
```

### Dopo check-in
```
Il bottone cambia stato → "Observed"
Il grafico anima (300ms Framer Motion)
Nessun toast. Nessun popup. Nessun confetti.
```

---

## Edge Case

- Utente con 1 sola area → una card, schermata non scrollabile
- Utente con molte aree (6+) → scroll verticale fluido, tutte le card al 55vh
- Time range 365d senza dati sufficienti → il grafico mostra i dati disponibili senza errori
- Check-in già effettuato oggi → bottone mostra "Observed" al caricamento
- Nessuna connessione → dati cached mostrati, errore non bloccante in basso

---

## Acceptance Criteria

- [ ] Le TrajectoryCard sono visibili con grafico al caricamento della Dashboard
- [ ] Il time range selector aggiorna tutti i grafici simultaneamente
- [ ] La pill attiva ha animazione Framer Motion layoutId
- [ ] Il tap sul corpo della card naviga all'Area Detail
- [ ] Lo stato empty mostra l'icona Eye con il copy corretto
- [ ] Lo stato loading mostra skeleton animate-pulse
- [ ] Dopo il check-in: solo il bottone e il grafico cambiano — nessun feedback aggiuntivo
- [ ] Il bottone "Log today" è min-h 44px e full-width

---

## Note UI

- Componenti di riferimento: `<TrajectoryCard>`, `<CheckInButton>`, `<TimeRangeSelector>`, `<AreaTypePill>`
- Brand tokens: `brand-system/betonme_brand_system_lovable.md`
- Anti-pattern da rispettare: sezione 10 del brand system

---

## Stories

- `story-02-01` — Struttura layout Dashboard con header e bottom nav
- `story-02-02` — TrajectoryCard con grafico Recharts e colore da slope
- `story-02-03` — TimeRangeSelector con animazione Framer Motion
- `story-02-04` — Empty state Dashboard
- `story-02-05` — Loading skeleton state
