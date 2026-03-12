# Epic 02 — Home (Hub Giornaliero)

## Obiettivo
Offrire all'utente un accesso immediato alle attività da registrare oggi, con check-in diretto per ciascuna area — in meno di 3 secondi dall'apertura dell'app.

> **Nota architetturale:** Questa è la Home tab (Tab 1). La visualizzazione dei trend storici è stata spostata nella sezione Progress (Epic 12). Home è interazione, Progress è osservazione.

---

## Behavior

All'apertura dell'app l'utente vede la lista delle proprie aree attive con lo stato del check-in di oggi. Ogni area mostra un bottone check-in che cambia stato quando il log viene registrato.

Se l'utente ha un'area Gym con scheda configurata, la Home mostra anche un indicatore del giorno scheda previsto per oggi.

La Home non mostra grafici. I grafici vivono in Progress.

---

## Flussi

### Flusso principale — Log di oggi
1. L'utente apre l'app → Home
2. Vede la lista delle aree di oggi con bottone "Log today" / "Registra" per ciascuna
3. Tappa "Log today" sull'area desiderata
4. Il check-in viene registrato (Epic 03)
5. Il bottone transiziona a "Observed" (300ms)
6. Done — nessun altro feedback

### Check-in già effettuato
- Al caricamento, le aree già loggate oggi mostrano già lo stato "Observed"
- Il bottone "Observed" non è tappabile

### Indicatore Gym Day
- Se esiste un'area Gym con scheda e la sessione di oggi non è ancora stata completata:
  - Una riga nell'area card mostra il nome del giorno scheda (es. "Giorno 2 — Gambe")
  - Tap sull'indicatore → naviga ad Area Detail (Attività > Area Gym)

### Empty state (nessuna area)
- L'utente non ha ancora nessuna area
- Messaggio neutro con CTA per andare ad Attività e creare la prima area

---

## Layout

```
┌─────────────────────────────┐
│ BetonMe                     │  ← Header
├─────────────────────────────┤
│  Oggi, 12 Marzo             │  ← Data corrente
├─────────────────────────────┤
│                             │
│  ┌─────────────────────┐    │
│  │ Palestra            │    │  ← Area card
│  │ Giorno 2 — Gambe →  │    │  ← Indicatore gym day (se presente)
│  │ [Log today]         │    │  ← CheckInButton
│  └─────────────────────┘    │
│                             │
│  ┌─────────────────────┐    │
│  │ Studio              │    │
│  │ [Observed ✓]        │    │  ← Già loggato oggi
│  └─────────────────────┘    │
│                             │
│  ┌─────────────────────┐    │
│  │ Social media        │    │
│  │ [Log today]         │    │
│  └─────────────────────┘    │
│                             │
├─────────────────────────────┤
│  [Nav]                      │
└─────────────────────────────┘
```

---

## Area Card nella Home

| Proprietà | Valore |
|---|---|
| Background | `bg-[#1F4A50] rounded-xl p-4` |
| Nome area | Testo principale, `text-[#EAEAEA]` |
| Indicatore gym day | Testo piccolo `text-[#B9C0C1]`, tap → Area Detail gym |
| CheckInButton | Vedi Epic 03 per gli stati |
| Ordine aree | Ordine di creazione dell'utente |

---

## Stati UI

### Empty State (nessuna area)
```
Icona:    Eye (Lucide), 48px, #7DA3A0
Testo IT: "Cosa vuoi osservare?"
Testo EN: "What do you want to observe?"
Sub IT:   "Aggiungi un'area in Attività per iniziare."
Sub EN:   "Add an area in Activities to start observing."
CTA IT:   "Vai ad Attività"
CTA EN:   "Go to Activities"
→ naviga a /activities
```

### Loading State
```
Skeleton: 2-3 card bg-[#1F4A50] animate-pulse rounded-xl h-24
```

### Nessuna attività pianificata per oggi
Se tutte le aree risultano già loggiate oggi:
```
Messaggio IT: "Tutto registrato per oggi."
Messaggio EN: "All logged for today."
Stile: testo piccolo centrato, text-[#B9C0C1]
```

---

## Edge Case

- Area archiviata → non compare nella lista
- Più di 5 aree → scroll verticale naturale
- Utente ha solo aree di tipo Finance → compaiono normalmente (Finance non è speciale in Home)
- Area Gym senza scheda configurata → nessun indicatore gym day, solo CheckInButton
- Sessione gym già completata oggi → nessun indicatore gym day

---

## Acceptance Criteria

- [ ] La Home mostra la lista di tutte le aree attive con CheckInButton
- [ ] Il CheckInButton segue gli stati definiti in Epic 03 (idle / loading / completed)
- [ ] Al caricamento, le aree già loggate oggi mostrano già lo stato "Observed"
- [ ] L'indicatore gym day appare solo se esiste area gym con scheda e sessione non completata oggi
- [ ] Tap sull'indicatore gym day naviga all'Area Detail dell'area gym
- [ ] L'empty state mostra il copy corretto nella lingua corrente con CTA verso Attività
- [ ] Le aree archiviate non compaiono
- [ ] La Home non contiene grafici (i grafici sono in Progress — Epic 12)

---

## Note UI

- Nessun grafico in Home — la Home è azione, non osservazione
- Nessun feedback celebrativo al check-in — coerente con tono osservativo del prodotto
- Data mostrata nell'header: formato localizzato (IT: "12 Marzo 2026" / EN: "March 12, 2026")
- Brand tokens: `brand-system/brand_system.md`

---

## Dipendenze

- Epic 03 (Check-in) — CheckInButton e logica score
- Epic 08 (i18n) — label IT/EN
- Epic 11 (Gym) — indicatore gym day condizionale
- Epic 12 (Progress) — la visualizzazione dei trend è stata spostata lì

---

## Stato implementazione

**Da refactorare** — l'attuale `src/pages/Index.tsx` contiene il grafico aggregato (comportamento del vecchio spec). Va completamente riscritto come hub giornaliero con lista attività.

| Componente da creare | Descrizione |
|---|---|
| `src/components/TodayActivityList.tsx` | Lista aree con CheckInButton per ciascuna |
| `src/components/GymDayIndicator.tsx` | Indicatore giorno scheda palestra |

| Dead code da rimuovere | Motivo |
|---|---|
| `src/components/TrajectoryCard.tsx` | Non più usato |
| `src/components/TrajectoryCardSkeleton.tsx` | Non più usato |

Il grafico aggregato e il MacroAreaSelector migrano in `src/pages/Progress.tsx` (Epic 12).

---

## Stories

- `story-02-01` — Layout Home: header data, lista aree di oggi con CheckInButton — **da fare**
- `story-02-02` — Stato check-in per ciascuna area: caricamento e aggiornamento ottimistico — **da fare**
- `story-02-03` — Indicatore gym day nella Home — **da fare**
- `story-02-04` — Empty state Home (nessuna area → CTA Attività) — **da fare**

> Le story 02-01..02-05 precedenti (grafico aggregato) sono **deprecate**. Il comportamento è stato spostato in Epic 12 (Progress).
