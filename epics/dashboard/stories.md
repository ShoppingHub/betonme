# Stories — Epic 02 — Dashboard

## Sequenza di implementazione — ✅ tutte completate

```
story-02-01 → Layout Dashboard: header + grafico aggregato totale             ✅
story-02-02 → MacroAreaSelector: filtro per macro-area                        ✅
story-02-03 → TimeRangeSelector con animazione Framer Motion                  ✅
story-02-04 → Empty state                                                     ✅
story-02-05 → Loading skeleton                                                ✅
```

> **Dipendenze:** Richiede Epic 00 (auth) e Epic 01 (onboarding) completati. I dati arrivano dalla tabella `areas` e `score_daily` di Supabase.

---

## story-02-01 — Layout Dashboard con grafico aggregato

BetonMe è un'app mobile-first di osservazione del benessere personale. Aggiorna la Dashboard (route `/`) per mostrare un unico grafico aggregato invece delle singole TrajectoryCard.

**Cosa mostra:**
- Header fisso (56px): wordmark `"BetonMe"` a sinistra, icona Settings a destra → naviga a `/settings`
- Selettore time range sticky sotto l'header: pill `"30d"` · `"90d"` · `"365d"` — default `"30d"`
- Grafico unico Recharts `<LineChart>` di altezza 60vh che rappresenta l'andamento totale dell'utente: media mobile dei `cumulative_score` di tutte le aree attive
- Sotto il grafico: `<MacroAreaSelector>` (implementato in story-02-02)

**Logica dati grafico:**
- Raggruppa i `score_daily.cumulative_score` per data
- Per ogni data, calcola la media dei punteggi di tutte le aree attive dell'utente
- La linea rappresenta questa media nel tempo

**Colore linea (slope ultimi 7 giorni):**
- Slope > 0.1 → `#7DA3A0`
- Slope < -0.1 → `#BFA37A`
- Neutro → `#8C9496`

**Stile grafico:**
- Griglia solo orizzontale, `opacity-10`
- Asse Y nascosto
- Asse X: tick date formato breve, 12px, `#B9C0C1`
- Background card: `bg-[#1F4A50] rounded-xl p-4`

---

## story-02-02 — MacroAreaSelector

Continua la Dashboard di BetonMe. Aggiungi il selettore di macro-area sotto il grafico.

**Componente `<MacroAreaSelector>`:**
- Row di pill scorrevole orizzontalmente sotto il grafico
- 5 opzioni: `Tutto` · `Salute` · `Studio` · `Riduci` · `Finanze` (IT) / `All` · `Health` · `Study` · `Reduce` · `Finance` (EN)
- Label rispetta la lingua dell'utente (preferenza `language` da tabella `users`)
- Default: `Tutto / All` attivo
- Pill attiva: background `#7DA3A0`, testo scuro
- Pill inattiva: background trasparente, bordo sottile, testo `#B9C0C1`

**Behavior filtro:**
- Tap su una macro-area → il grafico si aggiorna mostrando la media dei `cumulative_score` delle sole aree di quel tipo (`health` / `study` / `reduce` / `finance`)
- Tap su `Tutto / All` → torna alla media di tutte le aree
- Se l'utente non ha aree per una macro-categoria → il grafico mostra una linea piatta a zero, nessun errore
- La selezione macro-area rimane quando l'utente cambia il time range

---

## story-02-03 — TimeRangeSelector animato

Continua la Dashboard di BetonMe. Implementa l'animazione del `<TimeRangeSelector>`.

**Behavior:**
- Tap su una pill (30d / 90d / 365d) → la pill attiva si sposta con animazione Framer Motion `layoutId`
- Il grafico si aggiorna con i dati del range selezionato (300ms ease-in-out)
- La selezione macro-area rimane attiva

---

## story-02-04 — Empty state

Continua la Dashboard di BetonMe. Aggiungi l'empty state per utenti senza aree attive.

**Quando appare:** nessuna area nella tabella `areas` per l'utente (o tutte archiviate).

**Cosa mostra (segue la lingua utente):**
- Icona `Eye` (Lucide), 48px, `#7DA3A0`
- IT: `"Cosa vuoi osservare?"` / EN: `"What do you want to observe?"`
- IT: `"Aggiungi un'area di vita per iniziare."` / EN: `"Add a life area to start seeing your trajectory."`
- CTA IT: `"Aggiungi la tua prima area"` / EN: `"Add your first area"` → naviga a `/areas/new`

---

## story-02-05 — Loading skeleton

Continua la Dashboard di BetonMe. Aggiungi lo stato di loading.

**Quando appare:** durante il caricamento iniziale da Supabase.

**Cosa mostra:**
- Skeleton `animate-pulse` `bg-[#1F4A50] rounded-xl h-[60vh]` al posto del grafico
- Skeleton row di 5 pill muted al posto del MacroAreaSelector
- Il TimeRangeSelector rimane visibile ma non interattivo
