# Epic 03 — Check-in Giornaliero

## Obiettivo
Permettere all'utente di registrare l'osservazione del giorno per un'area in meno di 3 secondi, senza interruzioni e senza feedback valutativo.

---

## Behavior

Il check-in è un'azione binaria: fatto o non fatto. Si esegue dalla Dashboard o dall'Area Detail. L'unico feedback è il cambio di stato del bottone e l'animazione del grafico. Nessun popup, nessun toast, nessuna notifica positiva.

Un solo check-in è possibile per area per giorno (constraint: `UNIQUE(area_id, date)`).

---

## Flussi

### Flow principale — Check-in dalla Dashboard (< 3 secondi)
1. L'utente apre l'app
2. Vede la Dashboard con i grafici delle proprie aree
3. Tappa "Log today" sulla card dell'area desiderata
4. Il bottone mostra uno spinner (stato loading)
5. Il check-in viene registrato su Supabase (`completed = true`)
6. Il bottone transiziona a "Observed" (300ms Framer Motion)
7. Il grafico della card anima l'aggiornamento (300ms)
8. Fatto — nessun altro feedback

### Check-in già effettuato
- Al caricamento della Dashboard, il bottone mostra già "Observed"
- Il bottone "Observed" non è tappabile (nessuna azione al tap)

---

## Specifiche CheckInButton

| Stato | Stile | Testo |
|---|---|---|
| Idle | `border border-[#7DA3A0] text-[#EAEAEA] bg-transparent` | `"Log today"` |
| Loading | `opacity-50 cursor-not-allowed` + spinner inline | — |
| Completed | `bg-[#7DA3A0]/20 text-[#7DA3A0] border border-[#7DA3A0]` | `"Observed"` |

**Tutti gli stati:** `min-h-[44px] w-full rounded-lg text-[16px] font-medium`

---

## Logica Score (backend)

Al momento del check-in, viene calcolato e salvato il `score_daily`:

```
completed = true  → daily_score = +1.0
primo giorno mancato  → daily_score = 0.0
secondo giorno mancato → daily_score = -0.5
terzo+ giorno mancato  → daily_score = -1.0

cumulative_score = cumulative_score(ieri) + daily_score
```

Il `cumulative_score` alimenta il grafico. Non viene mai mostrato come numero (default `settings_score_visible = false`).

---

## Stati UI

| Stato | Comportamento |
|---|---|
| Non loggato oggi | Bottone "Log today", stile idle |
| In caricamento | Bottone con spinner, opacity-50 |
| Loggato oggi | Bottone "Observed", stile completed, non tappabile |
| Errore di rete | Bottone torna idle, errore non bloccante in basso |

---

## Edge Case

- Tap multipli rapidi → solo il primo check-in viene registrato (stato loading blocca tap aggiuntivi)
- Check-in offline → operazione in coda, sincronizzata al rientro online
- Check-in a mezzanotte passata → si riferisce sempre alla data locale dell'utente
- Area archiviata → il bottone check-in non è visibile
- `cumulative_score` visualizzato come numero solo se `settings_score_visible = true`

---

## Acceptance Criteria

- [ ] Tap su "Log today" registra `completed = true` per l'area e la data corrente
- [ ] Il bottone transiziona da idle → loading → completed senza refresh della pagina
- [ ] Il grafico anima l'aggiornamento entro 300ms dal check-in
- [ ] Nessun toast, popup o messaggio positivo al completamento
- [ ] Al reload della Dashboard, il bottone mostra già "Observed" se check-in già effettuato
- [ ] Il bottone "Observed" non risponde al tap
- [ ] Il constraint UNIQUE(area_id, date) impedisce duplicati a livello database

---

## Note UI

- Il feedback è esclusivamente visivo: cambio stato bottone + animazione grafico
- Framer Motion: `animate={{ opacity: isCompleted ? 0.8 : 1 }}` sul bottone
- Framer Motion: transizione 300ms `ease-in-out` sul grafico dopo aggiornamento dati
- Brand tokens: `brand-system/betonme_brand_system_lovable.md`

---

## Stories

- `story-03-01` — CheckInButton con tre stati (idle, loading, completed)
- `story-03-02` — Integrazione Supabase: salvataggio check-in e calcolo score
- `story-03-03` — Animazione grafico post check-in (Framer Motion 300ms)
