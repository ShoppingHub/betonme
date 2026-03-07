# CLAUDE.md — epics/

## Scopo di questa cartella

Contiene le **specifiche complete** di ogni feature di BetonMe.
Ogni epic descrive cosa fa una funzionalità, come si comporta, e quando è completa.

---

## Struttura di un epic

Ogni epic è un file markdown nella forma `epic-[XX]-[nome].md`.

```
epics/
├── epic-01-onboarding.md
├── epic-02-dashboard.md
├── epic-03-checkin.md
├── epic-04-area-detail.md
├── epic-05-add-edit-area.md
├── epic-06-finance.md
├── epic-07-settings.md
└── ...
```

---

## Cosa contiene un epic

Ogni file epic deve avere queste sezioni:

```
# Epic [XX] — [Nome]

## Obiettivo
Una frase: cosa risolve questa feature per l'utente.

## Behavior
Descrizione funzionale di come si comporta la feature.
Flussi principali, interazioni, stati UI.

## Flussi
Step-by-step dei percorsi utente principali.

## Stati UI
Lista degli stati visivi: vuoto, caricamento, errore, successo.

## Edge case
Situazioni limite da gestire esplicitamente.

## Acceptance criteria
Lista di condizioni verificabili. La feature è completa quando tutti sono soddisfatti.

## Stories
Lista delle stories collegate (link o riferimento).
```

---

## Struttura di una story

Le stories sono task atomici — una sola azione, implementabile in un singolo prompt Lovable.

Formato file: `story-[epic-XX]-[numero]-[nome].md`

```
# Story [epic-XX]-[N] — [Titolo]

## Contesto
A quale epic appartiene e perché esiste questa story.

## Azione
Cosa deve fare Lovable in questa story. Una sola cosa.

## Acceptance criteria
2-4 condizioni verificabili e specifiche.

## Note UI
Riferimenti a token di brand, componenti, copy esatti se rilevanti.
```

---

## Regole

- Un epic = una feature o un blocco funzionale coeso
- Una story = un task atomico eseguibile in un prompt
- Non scrivere codice negli epic — descrivi il behavior
- Non duplicare informazioni già nel brand system — fai riferimento a `brand-system/`
- Se una story diventa troppo grande → spezzala in due
- Numerazione epic: sequenziale, con zero padding (`01`, `02`...)
- Numerazione stories: per epic (`epic-01` → `story-01-01`, `story-01-02`...)
