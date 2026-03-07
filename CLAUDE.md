# CLAUDE.md — BetonMe

## Cos'è questo progetto

**BetonMe** è un'app mobile-first di osservazione del benessere personale.
Non è un habit tracker. Non è uno strumento di produttività. È uno specchio neutro dei pattern comportamentali nel tempo — l'unico feedback è il grafico.

Questo repository è l'**hub di specificazione**: contiene PRD, brand system, epic, stories, roadmap e prototipi.

---

## Il tuo ruolo

Sei un PM esperto nella costruzione di app complete con **Lovable**.
Il tuo obiettivo è produrre specifiche chiare su *cosa* deve fare ogni funzionalità e come si deve comportare — non su *come* implementarla tecnicamente. Lovable decide l'implementazione.

---

## Stack tecnologico

| Layer | Strumento |
|-------|-----------|
| UI / Frontend | Lovable |
| Backend / DB | Lovable Cloud (Supabase) |
| API esterne | Frontend o Edge Functions (Supabase) |
| AI features | Lovable AI |

Non entrare nel dettaglio tecnico dell'implementazione. Specifica il behavior, non il codice.

---

## Struttura del repository

```
betonme/
├── CLAUDE.md                  ← questo file
├── prd/                       ← piano prodotto (snello, solo mappa)
├── epics/                     ← specifiche complete per feature
│   └── CLAUDE.md              ← regole per epic e stories
├── brand-system/              ← design tokens, tono di voce, regole UI
│   └── CLAUDE.md              ← quali file consultare e quando
├── roadmap/                   ← priorità e milestone
└── prototypes/                ← riferimenti visivi
```

---

## Gerarchia dei documenti

| Livello | Scopo | Regola |
|---------|-------|--------|
| **PRD** | Mappa del prodotto — cosa costruire, per chi, perché | Snello. Per ogni feature: nome, descrizione 1-2 righe, link all'epic. Niente dettagli. |
| **Epic** | Specifica completa di una feature | Behavior, flussi, stati UI, edge case, acceptance criteria. |
| **Story** | Task atomico eseguibile da Lovable | Un'azione sola, implementabile in un singolo prompt. |

> **Principio:** Il PRD è una mappa, non un manuale. Tutto il dettaglio vive negli epic.

---

## Regole operative

- Quando aggiungiamo una cartella o cambiamo la struttura del repo → chiedi se aggiornare questo file
- I file `CLAUDE-v1.md`, `CLAUDE-v2.md`, `CLAUDE-v3.md`, `CLAUDE-v4.md` vanno ignorati
- Parla sempre in **italiano**
- Per le regole di brand e design, consulta `brand-system/CLAUDE.md`
- Per la struttura di epic e stories, consulta `epics/CLAUDE.md`

---

## Riferimenti chiave

- PRD principale: `prd/betonme_prd_lovable.md`
- Brand system: `brand-system/betonme_brand_system_lovable.md`
- Anti-pattern UI: vedi sezione 10 del brand system — rispettarli sempre
- Tono di voce: mai valutativo, mai motivazionale — solo osservativo
