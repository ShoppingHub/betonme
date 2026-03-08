# Stories — Epic 10 — Areas (Sezione Aree)

## Sequenza di implementazione — ✅ tutte completate

```
story-10-01 → Layout sezione Aree con 4 macro-categorie e liste area         ✅
story-10-02 → CTA aggiungi per tipo + CTA globale header                     ✅
story-10-03 → Stati empty, loading e aree archiviate                          ✅
```

> **Dipendenze:** Richiede Epic 04 (Area Detail) e Epic 05 (Add/Edit Area) per la navigazione. Epic 08 (i18n) per i label.

---

## story-10-01 — Layout sezione Aree con 4 macro-categorie

BetonMe è un'app di osservazione del benessere. Sostituisci il placeholder della pagina Areas (route `/areas`) con il layout completo.

**Struttura:**
- Header: `"Aree" / "Areas"` + icona `+` in alto a destra (CTA globale)
- 4 sezioni verticali — una per macro-area — sempre visibili anche se vuote:

**Sezione Salute / Health** (icona `Heart`)
- Lista delle aree dell'utente con `type = "health"` e `archived_at IS NULL`
- Ogni area: riga con nome a sinistra + chevron `>` a destra, background `#1F4A50`, min-height 48px, rounded
- Tap su riga → naviga ad Area Detail (`/areas/:id`)

**Sezione Studio / Study** (icona `BookOpen`)
- Stessa struttura, filtra `type = "study"`

**Sezione Riduci / Reduce** (icona `TrendingDown`)
- Stessa struttura, filtra `type = "reduce"`

**Sezione Finanze / Finance** (icona `Wallet`)
- Stessa struttura, filtra `type = "finance"`

**Label macro-aree (seguono lingua utente):**
- IT: `Salute · Studio · Riduci · Finanze`
- EN: `Health · Study · Reduce · Finance`

---

## story-10-02 — CTA aggiungi

Continua Epic 10 di BetonMe. Aggiungi le CTA per creare nuove aree.

**CTA per tipo (in fondo a ogni sezione):**
- Label IT: `+ Aggiungi` / EN: `+ Add`
- Stile: link testuale piccolo, colore `#7DA3A0`
- Tap → naviga a `/areas/new` con il tipo della sezione pre-selezionato come query param (es. `?type=health`)

**CTA globale (icona `+` nell'header):**
- Tap → naviga a `/areas/new` senza pre-selezione del tipo

---

## story-10-03 — Stati empty, loading e aree archiviate

Continua Epic 10 di BetonMe. Gestisci gli stati speciali della sezione Aree.

**Macro-area senza aree (empty per quel tipo):**
- Non mostrare card — mostrare solo la CTA `+ Aggiungi / + Add`
- Nessun testo aggiuntivo, nessuna icona

**Loading state:**
- Skeleton `animate-pulse` per ogni sezione: 1-2 righe placeholder per macro-area
- Il layout a 4 sezioni rimane visibile con i titoli

**Aree archiviate:**
- Non mostrate nella lista (filtrate dalla query: `archived_at IS NULL`)
- Le aree archiviate rimangono accessibili solo tramite link diretto (se necessario)

**Nome area lungo:**
- Troncato con `text-ellipsis overflow-hidden` — nessun wrapping
