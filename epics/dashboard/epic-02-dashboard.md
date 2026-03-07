# Epic 02 — Dashboard (Home)

## Obiettivo
Offrire all'utente una visione immediata dell'andamento complessivo del proprio benessere, con la possibilità di approfondire la traiettoria per singola macro-area tramite un selettore.

---

## Behavior

La Dashboard mostra **un unico grafico aggregato** che rappresenta l'andamento totale dell'utente — media ponderata dei `cumulative_score` di tutte le aree attive. È il "termometro" globale del benessere.

Sotto il grafico totale, un selettore di macro-area permette di filtrare il grafico per visualizzare la traiettoria di una sola macro-categoria (Salute / Studio / Riduci / Finanze). Quando una macro-area è selezionata, il grafico mostra la media dei punteggi delle aree di quel tipo.

Il check-in del giorno per le singole aree rimane accessibile dalla schermata Areas (Epic 10) e dall'Area Detail (Epic 04).

---

## Flussi

### Visualizzazione Dashboard — Vista Totale
1. L'utente apre l'app
2. Vede il grafico totale aggregato (default: 30d)
3. Sotto il grafico: selettore macro-area con 5 opzioni — `Tutto · Salute · Studio · Riduci · Finanze` (IT) / `All · Health · Study · Reduce · Finance` (EN)
4. Il selettore è in stato `Tutto / All` di default

### Filtro per Macro-area
1. L'utente tappa una macro-area nel selettore
2. Il grafico si aggiorna mostrando solo i dati di quella macro-area
3. L'opzione selezionata è evidenziata
4. Tap su `Tutto / All` riporta alla vista aggregata

### Cambio Time Range
1. L'utente tappa uno dei tre pill (30d, 90d, 365d)
2. Il grafico si aggiorna con il range selezionato
3. La selezione di macro-area rimane attiva

---

## Layout

```
┌─────────────────────────────┐
│ BetonMe          [settings] │  ← Header 56px
├─────────────────────────────┤
│  30d  │  90d  │  365d       │  ← TimeRangeSelector
├─────────────────────────────┤
│                             │
│   [Grafico totale — 60vh]   │  ← Aggregato o filtrato per macro-area
│                             │
├─────────────────────────────┤
│ Tutto│Salute│Studio│Riduci│€ │  ← MacroAreaSelector (pill row scrollabile)
├─────────────────────────────┤
│  [Nav]                      │
└─────────────────────────────┘
```

### MacroAreaSelector

| Proprietà | Valore |
|---|---|
| Tipo | Pill row scorrevole orizzontalmente |
| Opzioni | Tutto · Salute · Studio · Riduci · Finanze (IT) / All · Health · Study · Reduce · Finance (EN) |
| Default | `Tutto / All` attivo |
| Icone | Lucide outline mini accanto al label (opzionale) |
| Pill attiva | Background `#7DA3A0`, testo dark |
| Pill inattiva | Background trasparente, bordo sottile, testo `#B9C0C1` |

---

## Specifiche Grafico Totale

| Proprietà | Valore |
|---|---|
| Componente | Recharts `<LineChart>` |
| Tipo linea | `type="monotone"` |
| Dati | Media mobile dei `cumulative_score` di tutte le aree attive (o filtrate per macro-area) |
| Colore linea | Calcolato da slope 7 giorni → `#7DA3A0` / `#8C9496` / `#BFA37A` |
| Griglia | Solo orizzontale, `opacity-10` |
| Asse Y | Nascosto |
| Asse X | Tick date, 12px, `#B9C0C1` |
| Altezza | 60vh |
| Background | `bg-[#1F4A50] rounded-xl p-4` |

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
Testo IT: "Cosa vuoi osservare?"
Testo EN: "What do you want to observe?"
Sub IT:   "Aggiungi un'area di vita per iniziare."
Sub EN:   "Add a life area to start seeing your trajectory."
CTA IT:   "Aggiungi la tua prima area"
CTA EN:   "Add your first area"
→ naviga ad Add Area
```

### Loading State (caricamento iniziale)
```
Skeleton: bg-[#1F4A50] animate-pulse rounded-xl h-[60vh]
Skeleton MacroAreaSelector: 5 pill muted
```

### Macro-area senza dati
```
Il grafico mostra una linea piatta a zero — nessun messaggio di errore
```

---

## Edge Case

- Utente con 1 sola area → il grafico totale e quello filtrato coincidono
- Utente con aree solo in 2 macro-categorie → le altre 2 pill del selettore mostrano una linea piatta
- Time range 365d senza dati sufficienti → grafico con i dati disponibili, senza errori
- Nessuna connessione → dati cached mostrati, errore non bloccante

---

## Acceptance Criteria

- [ ] La Dashboard mostra un unico grafico aggregato (non le singole TrajectoryCard)
- [ ] Il grafico aggregato usa la media dei `cumulative_score` di tutte le aree attive
- [ ] Il MacroAreaSelector ha 5 opzioni: Tutto · Salute · Studio · Riduci · Finanze (IT) / All · Health · Study · Reduce · Finance (EN)
- [ ] La selezione di una macro-area filtra il grafico per quel tipo di aree
- [ ] `Tutto / All` è l'opzione di default
- [ ] Il TimeRangeSelector aggiorna il grafico mantenendo la selezione macro-area
- [ ] L'empty state mostra il copy nella lingua corrente (Epic 08)
- [ ] Il loading state mostra skeleton animate-pulse

---

## Note UI

- Componenti: `<MacroAreaSelector>`, `<TimeRangeSelector>`
- Brand tokens: `brand-system/betonme_brand_system_lovable.md`
- Anti-pattern da rispettare: sezione 10 del brand system

---

## Dipendenze

- Epic 08 (i18n) — per le label IT/EN del MacroAreaSelector e dell'empty state

---

## Stories

- `story-02-01` — Layout Dashboard: header, grafico aggregato totale, TimeRangeSelector
- `story-02-02` — MacroAreaSelector: selezione e filtro grafico per macro-area
- `story-02-03` — Animazione Framer Motion sul TimeRangeSelector e transizione grafico
- `story-02-04` — Empty state Dashboard
- `story-02-05` — Loading skeleton state
