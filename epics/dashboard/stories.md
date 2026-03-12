# Stories — Epic 02 — Home (Hub Giornaliero)

## Sequenza di implementazione

```
story-02-01 → Layout Home: header data + lista aree di oggi con CheckInButton   ⏳ da fare
story-02-02 → Stato check-in: caricamento e aggiornamento per ciascuna area     ⏳ da fare
story-02-03 → Indicatore gym day nella Home                                     ⏳ da fare
story-02-04 → Empty state Home (nessuna area → CTA Attività)                    ⏳ da fare
```

> **Prerequisiti:** Richiede story-09-04 (refactor nav, route `/activities`) e Epic 03 (CheckInButton). La Home non richiede Epic 12 come prerequisito.

> **Story precedenti 02-01..02-05 (grafico aggregato) sono deprecate.** Il comportamento è stato spostato in Epic 12 (Progress). L'attuale `src/pages/Index.tsx` va riscritto.

---

## story-02-01 — Layout Home con lista attività di oggi

BetonMe è un'app mobile-first di osservazione del benessere. Riscrivi la Home (route `/`) come hub giornaliero: una lista semplice delle aree attive dell'utente con il bottone check-in per ciascuna.

**Rimuovi** il grafico aggregato attuale (`src/pages/Index.tsx`) e i componenti `TrajectoryCard.tsx` e `TrajectoryCardSkeleton.tsx` che sono dead code.

**Cosa mostra la nuova Home:**

**Header:**
- Wordmark `"BetonMe"` a sinistra
- Data di oggi a destra: formato localizzato — IT: `"12 marzo"` / EN: `"March 12"`

**Lista aree:**
- Una card per ciascuna area attiva dell'utente (non archiviate)
- Ogni card contiene:
  - Nome area (testo principale)
  - CheckInButton (vedi Epic 03 per gli stati: idle / loading / completed)
- Background card: `bg-[#1F4A50] rounded-xl p-4`
- Ordine: ordine di creazione area

**Ordine check-in:**
- Al caricamento, query `checkins` per `user_id` e `date = oggi`
- Se esiste un checkin `completed = true` per un'area → bottone in stato "Observed"
- Se non esiste → bottone in stato "Log today"

**Navigazione:**
- Tap sull'area card (fuori dal bottone) → naviga ad `/activities/:id` (Area Detail)

---

## story-02-02 — Stato check-in per ciascuna area

Continua la Home di BetonMe. Implementa il caricamento e l'aggiornamento dello stato check-in per ogni area nella lista.

**Al caricamento:**
- Query a Supabase: `checkins` dove `user_id = currentUser` AND `date = today`
- Per ogni area, determina se il check-in di oggi è completato
- Il bottone mostra lo stato corretto senza attendere l'interazione utente

**Loading state:**
- Durante la query iniziale: skeleton `animate-pulse` per ogni card (`bg-[#1F4A50] rounded-xl h-20`)

**Dopo tap su "Log today":**
- Segui la logica di Epic 03 per la transizione idle → loading → completed
- L'aggiornamento è ottimistico: il bottone passa a "Observed" immediatamente, la scrittura avviene in background

**Tutto loggato oggi:**
- Se tutte le aree sono in stato "Observed":
  - IT: `"Tutto registrato per oggi."` — testo piccolo centrato sotto la lista, `text-[#B9C0C1]`
  - EN: `"All logged for today."`

---

## story-02-03 — Indicatore gym day nella Home

Continua la Home di BetonMe. Aggiungi l'indicatore del giorno scheda palestra nella card dell'area gym, quando applicabile.

**Condizione di visibilità:**
- L'utente ha un'area `type = health` con nome `gym` o `palestra` (case insensitive)
- L'area ha una scheda configurata (`gym_programs` esiste per quest'area)
- La sessione di oggi non è ancora stata completata (`gym_sessions` non esiste per `area_id + date = oggi`)

**Quando visibile — dentro la card dell'area Gym:**
- Sotto il nome area, riga aggiuntiva: `"Giorno 2 — Gambe →"` (IT) / `"Day 2 — Legs →"` (EN)
- Stile: `text-sm text-[#B9C0C1]`
- Tap sulla riga → naviga a `/activities/:gymAreaId` (Area Detail gym)

**Logica "giorno successivo":**
- Stesso algoritmo dell'Epic 11: giorno successivo all'ultima sessione, rotazione ciclica
- Se nessuna sessione precedente → Giorno 1

**Quando non visibile:**
- Area gym non esiste
- Scheda non ancora configurata
- Sessione di oggi già completata

---

## story-02-04 — Empty state Home

Continua la Home di BetonMe. Implementa l'empty state per utenti senza aree attive.

**Quando appare:** nessuna area nella tabella `areas` per l'utente (o tutte archiviate).

**Cosa mostra (segue la lingua utente):**
- Icona `Eye` (Lucide), 48px, `#7DA3A0`
- IT: `"Cosa vuoi osservare?"` / EN: `"What do you want to observe?"`
- IT: `"Aggiungi un'area in Attività per iniziare."` / EN: `"Add an area in Activities to start observing."`
- CTA IT: `"Vai ad Attività"` / EN: `"Go to Activities"` → naviga a `/activities`
