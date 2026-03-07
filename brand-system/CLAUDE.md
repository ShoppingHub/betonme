# CLAUDE.md — brand-system/

## Scopo di questa cartella

Contiene le **regole visive e di comunicazione** di BetonMe.
Ogni decisione di design — colori, tipografia, tono, componenti — è definita qui.

---

## File presenti

```
brand-system/
├── betonme_brand_system_lovable.md   ← fonte primaria per UI e design
```

---

## Quando consultare il brand system

Consultare `betonme_brand_system_lovable.md` ogni volta che si specifica:

- **Colori** — backgrounds, testo, grafico, accent
- **Tipografia** — dimensioni, pesi, gerarchia
- **Spaziatura** — padding, gap, margini
- **Componenti** — TrajectoryCard, CheckInButton, TimeRangeSelector, AreaTypePill, CalendarHeatmap
- **Grafici** — libreria, tipo di linea, colori, animazioni
- **Motions** — durate, easing, cosa è permesso e cosa no
- **Tono di voce** — vocabulary permesso e vietato, copy UI ufficiali
- **Anti-pattern** — lista di ciò che non va mai implementato

---

## Regole d'uso

- I token del brand system sono **non negoziabili** — non inventare colori o stili alternativi
- Il colore `#E24A4A` (accent rosso) è **solo per errori** — mai per trend negativi o giorni mancati
- Il colore decline è sempre `#BFA37A` (ambra calda) — mai rosso
- Non usare mai vocabulary vietato negli epic, nelle stories o nei prompt Lovable
- Se una specifica UI non è coperta dal brand system → chiedi prima di inventare

---

## Riferimento rapido token

```
Sfondo principale:   #0F2F33
Card / container:    #1F4A50
Testo primario:      #EAEAEA
Testo secondario:    #B9C0C1
Accent (solo errori):#E24A4A
Grafico positivo:    #7DA3A0
Grafico neutro:      #8C9496
Grafico decline:     #BFA37A

Font:       Inter, 600/500/400
Radius:     12px
Motion:     300ms ease-in-out
Tap target: min 44px
```

---

## Mood del prodotto

> "Questo è un sistema di osservazione calmo, non un tracker di abitudini."

- ✅ Calmo · Neutro · Contemplativo
- ❌ Gamificato · Motivazionale · Valutativo
