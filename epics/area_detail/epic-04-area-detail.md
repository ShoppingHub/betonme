# Epic 04 — Area Detail

## Obiettivo
Offrire all'utente una visione approfondita della traiettoria di una singola area, con la possibilità di osservare il pattern a diverse scale temporali e visualizzare il calendario storico.

---

## Behavior

L'Area Detail è una schermata di analisi. Il grafico è più grande rispetto alla Dashboard (60vh) e occupa la parte dominante dello schermo. In basso è presente un heatmap a riga singola degli ultimi 30 giorni.

Il check-in del giorno può essere eseguito anche da questa schermata (stesso comportamento dell'Epic 03).

---

## Flussi

### Visualizzazione Area Detail
1. L'utente tappa il corpo di una TrajectoryCard in Dashboard
2. Naviga all'Area Detail con transizione Framer Motion (fade + slide 300ms)
3. Vede: header con nome area + back navigation, time range selector, grafico 60vh, heatmap 30 giorni

### Cambio Time Range
1. L'utente tappa un pill (30d / 90d / 365d)
2. Il grafico si aggiorna con i dati del periodo selezionato

### Modifica Area
1. L'utente tappa il link testuale "Edit area" in fondo alla schermata
2. Naviga ad Add/Edit Area (Epic 05) in modalità edit

---

## Layout

```
┌─────────────────────────────┐
│ ← [Nome Area]    [edit]     │  ← Header con back navigation
├─────────────────────────────┤
│  30d  │  90d  │  365d       │  ← TimeRangeSelector
├─────────────────────────────┤
│                             │
│   [Grafico — 60vh]          │
│                             │
├─────────────────────────────┤
│   Ultimi 30 giorni          │
│   □□■□■■□■□■□□■□■□         │  ← CalendarHeatmap
├─────────────────────────────┤
│   [Score: —]                │  ← Solo se settings_score_visible=true
├─────────────────────────────┤
│  Home · Areas · Finance · ⚙ │
└─────────────────────────────┘
```

---

## Specifiche CalendarHeatmap

| Tipo giorno | Stile |
|---|---|
| Completato | `bg-[#7DA3A0]/60 rounded-sm w-7 h-7` |
| Mancato | `bg-[#BFA37A]/40 rounded-sm w-7 h-7` |
| Futuro | `bg-[#1F4A50]/30 rounded-sm w-7 h-7` |
| Gap tra celle | `gap-0.5` |
| Layout | Riga singola orizzontale, scroll se necessario |

> ⚠️ Nessun contatore streak. Nessun "best streak". Nessuna fiamma. Nessuna celebrazione per giorni consecutivi.

---

## Visualizzazione Score

- Default: nascosto
- Visibile solo se `settings_score_visible = true` nelle impostazioni utente
- Se visibile: mostrato come numero sotto l'heatmap, label `"Trajectory score"`
- Se nascosto: l'area non compare

---

## Edit Link

- Posizionato in fondo alla schermata, sopra il bottom nav
- Stile: testo small, `text-[#B9C0C1]`, non un bottone
- Label: `"Edit area"`

---

## Stati UI

### Empty State (nessun dato per l'area)
```
Icona: TrendingUp (Lucide), muted
Testo: "Keep observing. Your trajectory takes shape over days."
Nessuna CTA
```

### Loading State
```
Skeleton grafico: bg-[#1F4A50] animate-pulse rounded-xl h-[60vh]
Skeleton heatmap: riga di celle bg-[#1F4A50] animate-pulse
```

---

## Edge Case

- Area senza dati (appena creata) → empty state corretto
- Time range 365d con meno di 365 giorni di dati → mostra i dati disponibili
- Score nascosto ma richiesto via URL diretto → rispettare la preferenza utente
- Back navigation con modifiche non salvate (da edit) → nessun modale di conferma in MVP

---

## Acceptance Criteria

- [ ] Il grafico occupa 60vh e usa `type="monotone"` Recharts
- [ ] Il time range selector funziona e aggiorna il grafico
- [ ] Il CalendarHeatmap mostra 30 giorni con i tre stili corretti (completato / mancato / futuro)
- [ ] Nessun streak counter o label "best streak" presente
- [ ] Lo score numerico appare solo se `settings_score_visible = true`
- [ ] Il link "Edit area" è testo small in `#B9C0C1`, non un bottone
- [ ] La back navigation riporta alla Dashboard
- [ ] L'empty state mostra il copy corretto senza CTA

---

## Note UI

- Componenti: `<CalendarHeatmap>`, `<TimeRangeSelector>`, `<CheckInButton>`
- Brand tokens: `brand-system/betonme_brand_system_lovable.md`

---

## Stories

- `story-04-01` — Layout Area Detail con header e back navigation
- `story-04-02` — Grafico 60vh con TimeRangeSelector
- `story-04-03` — CalendarHeatmap 30 giorni
- `story-04-04` — Visibilità score condizionale
