# Epic 09 — Layout Responsivo (Mobile Bottom Nav · Desktop Sidebar)

## Obiettivo
Adattare la navigazione principale al dispositivo: bottom navigation su mobile, sidebar laterale fissa su desktop — mantenendo la stessa struttura di voci e lo stesso sistema di personalizzazione del menu.

---

## Behavior

Su mobile il comportamento attuale rimane invariato: bottom navigation fissa in basso.

Su desktop (viewport ≥ 1024px) la navigazione migra sulla colonna sinistra come sidebar fissa, in stile Laravel Breeze / dashboard admin classico. Il contenuto si espande a destra della sidebar.

Le voci di navigazione — fisse e personalizzabili — sono le stesse su entrambi i layout (vedi Epic 07 — Settings per la configurazione del menu).

---

## Layout Mobile (< 1024px) — invariato

```
┌─────────────────────────────┐
│ Header                      │
├─────────────────────────────┤
│                             │
│   Contenuto pagina          │
│                             │
├─────────────────────────────┤
│  Home · Areas · [+] · [+] · Settings │  ← Bottom nav fissa 56px
└─────────────────────────────┘
```

---

## Layout Desktop (≥ 1024px)

```
┌──────────┬──────────────────────────────┐
│          │ Header pagina                │
│  [Logo]  ├──────────────────────────────┤
│          │                              │
│  Home    │   Contenuto pagina           │
│  Areas   │                              │
│  [+]     │                              │
│  [+]     │                              │
│          │                              │
│  ──────  │                              │
│  Settings│                              │
│          │                              │
└──────────┴──────────────────────────────┘
```

### Specifiche sidebar desktop

| Proprietà | Valore |
|---|---|
| Larghezza | 240px fissa |
| Background | `#0F2F33` (stesso del bottom nav mobile) |
| Logo / nome app | In cima alla sidebar, `BetonMe` |
| Voci nav | Icona + label testuale (su mobile solo icona) |
| Icone | Lucide outline 20px |
| Colore voce attiva | `#7DA3A0` |
| Colore voce inattiva | `#B9C0C1` |
| Settings | In fondo alla sidebar, separato da divider |
| Contenuto principale | `margin-left: 240px`, padding laterale 32px |
| Max width contenuto | 900px centrato |

---

## Voci di navigazione

Le voci mostrate dipendono dalla configurazione dell'utente (Epic 07):

| Posizione | Voce | Tipo |
|---|---|---|
| 1 | Home | Fissa |
| 2 | Areas / Aree | Fissa |
| 3 | [Slot custom 1] | Configurabile in Settings |
| 4 | [Slot custom 2] | Configurabile in Settings |
| Ultima | Settings / Impostazioni | Fissa |

Se l'utente non ha configurato slot custom → il menu mostra solo le 3 voci fisse.

---

## Breakpoint

| Breakpoint | Layout |
|---|---|
| < 1024px | Bottom navigation mobile |
| ≥ 1024px | Sidebar desktop |

> Usare `lg:` breakpoint di Tailwind (1024px).

---

## Stati UI

| Stato | Mobile | Desktop |
|---|---|---|
| Voce attiva | Icona `#7DA3A0`, label sotto | Riga intera highlight, icona + label `#7DA3A0` |
| Voce inattiva | Icona `#B9C0C1` | Icona + label `#B9C0C1` |
| Slot non configurato | Non mostrato | Non mostrato |

---

## Edge Case

- Viewport ridimensionato da desktop a mobile durante la sessione → il layout si adatta senza reload
- Sidebar desktop non collassabile (MVP) — nessun toggle hamburger
- Nessuna sub-navigazione nella sidebar (flat list)

---

## Acceptance Criteria

- [x] Su viewport < 1024px compare esclusivamente il bottom nav in basso
- [x] Su viewport ≥ 1024px compare esclusivamente la sidebar laterale sinistra (240px)
- [x] La sidebar mostra le stesse voci del bottom nav (fisse + configurabili)
- [x] Settings è sempre l'ultima voce nella sidebar, separata da divider
- [x] Il contenuto principale su desktop ha `margin-left: 240px` e max-width 900px
- [x] La voce attiva è evidenziata con `#7DA3A0` su entrambi i layout
- [x] Il resize del viewport aggiorna il layout dinamicamente (no reload)

---

## Stato implementazione

**Completato** — commit su `ShoppingHub/project-spark`.

| Componente | File |
|---|---|
| Layout shell | `src/components/AppLayout.tsx` — switch `lg:hidden` / `hidden lg:flex` |
| Bottom nav mobile | `src/components/BottomNav.tsx` — h-14, z-50, `lg:hidden` |
| Sidebar desktop | `src/components/DesktopSidebar.tsx` — w-[240px], bg `#0F2F33`, `hidden lg:flex` |
| Menu config | `src/hooks/useMenuConfig.tsx` — alimenta entrambi i layout |

### Variazioni rispetto all'epic
- Il bottom nav mobile mostra icone + label (non solo icone come specificato). Funzionalmente migliore per l'UX.
- Desktop content: `ml-[240px]`, `max-w-[900px] mx-auto`, `p-8` — conforme.

---

## Stories

- `story-09-01` — Sidebar desktop con logo, voci fisse e settings in fondo — **completata**
- `story-09-02` — Adattamento responsivo: bottom nav su mobile, sidebar su desktop — **completata**
- `story-09-03` — Voci custom nella sidebar da configurazione Settings — **completata**
