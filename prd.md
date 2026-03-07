# BetonMe — Product Requirements Document
### MVP · Personal Wellbeing Trajectory App · v2.0
### Target: **Lovable** · March 2026 · `CONFIDENTIAL`

---

## ⚡ TL;DR

BetonMe è una **mobile-first React web app** di osservazione del benessere personale.
Gli utenti registrano un'azione giornaliera per area di vita. Il grafico è il prodotto.
Tono: calmo, neutro, riflessivo. Un journal personale, non una fitness app.

---

## 1. Prodotto

BetonMe rende **visibile la traiettoria del benessere personale nel tempo** tramite grafici longitudinali.

Non è un habit tracker. Non è un goal-setting tool. È un **sistema di osservazione** — uno specchio neutro dei pattern comportamentali.

---

## 2. Target User

- **Età:** 25–55
- **Profilo:** Interessati alla crescita personale, già sperimentano abitudini di benessere
- **Pain:** Esauriti dalla cultura della produttività e dalla pressione delle performance
- **JTBD:** *"Voglio osservare la direzione delle mie azioni nel tempo, senza sentirmi valutato."*

---

## 3. Tech Stack

| Layer | Tecnologia |
|---|---|
| Framework | React 18 + TypeScript |
| Styling | Tailwind CSS |
| Components | shadcn/ui |
| Charts | Recharts |
| Animations | Framer Motion |
| Backend | Supabase (auth + database + edge functions) |
| Auth | Magic link (email only) |
| State | Zustand o React Context |
| Mobile breakpoint | 375px primary |

---

## 4. Architettura

```
App
├── Onboarding (unauthenticated)          → epic-01-onboarding.md
│   ├── Step 1: Account (magic link)
│   ├── Step 2: Scegli le aree
│   ├── Step 3: Imposta frequenza
│   └── → Dashboard
│
└── Main App (authenticated)
    ├── [Tab 1] Dashboard (Home)           → epic-02-dashboard.md
    ├── [Tab 2] Aree                       → epic-04-area-detail.md / epic-05-add-edit-area.md
    ├── [Tab 3] Finance                    → epic-06-finance.md
    └── [Tab 4] Settings                  → epic-07-settings.md
```

### Bottom Navigation Bar
- Fixed bottom, 56px, background `#0F2F33`
- 4 tab: **Home · Areas · Finance · Settings**
- Icone Lucide outline 24px — active `#7DA3A0`, inactive `#B9C0C1`

---

## 5. Feature Map

| # | Feature | Epic | Priorità |
|---|---|---|---|
| 01 | Onboarding (magic link + setup aree) | `epic-01-onboarding.md` | P0 |
| 02 | Dashboard — grafico traiettoria | `epic-02-dashboard.md` | P0 |
| 03 | Check-in giornaliero | `epic-03-checkin.md` | P0 |
| 04 | Area Detail — analisi traiettoria | `epic-04-area-detail.md` | P1 |
| 05 | Add / Edit Area | `epic-05-add-edit-area.md` | P0 |
| 06 | Finance Projection | `epic-06-finance.md` | P1 |
| 07 | Settings | `epic-07-settings.md` | P1 |

---

## 6. Database (Supabase — schema di riferimento)

RLS abilitato su tutte le tabelle.

```
users        → id, settings_score_visible (default false), settings_notifications
areas        → id, user_id, name, type, frequency_per_week, archived_at
checkins     → id, area_id, user_id, date, completed — UNIQUE(area_id, date)
score_daily  → id, area_id, date, daily_score, cumulative_score, consecutive_missed
```

**Score logic (edge function o trigger):**
```
completed       → +1.0
primo mancato   →  0.0
secondo mancato → -0.5
terzo+ mancato  → -1.0
cumulative_score = previous + daily_score
```

> Il `cumulative_score` alimenta il grafico. Non viene mai mostrato come numero (default off).

---

## 7. Scope MVP

### ✅ Incluso
- Magic link authentication
- 4 tipi area (Health, Study, Reduce, Finance)
- Check-in binario giornaliero
- Grafico traiettoria per area
- Time range: 30d / 90d / 365d
- Calendar heatmap 30 giorni
- Proiezione lineare Finance
- Toggle visibilità score (default off)
- Toggle notifiche

### ❌ Escluso
- Feature social o condivisione
- Badge, punti, livelli, classifiche
- Gamificazione streak
- Push notifications (toggle presente, logica rinviata)
- Layout desktop ottimizzato
- Export / download dati
- Suggerimenti AI

---

## 8. Riferimenti

- Brand system completo: `brand-system/betonme_brand_system_lovable.md`
- Regole design e anti-pattern: sezione 10 del brand system
- Tono di voce: mai valutativo, mai motivazionale — solo osservativo

---

*BetonMe PRD v2.0 — Mappa prodotto. Il dettaglio vive negli epic.*
