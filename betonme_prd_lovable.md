# BetonMe — Product Requirements Document
### MVP · Personal Wellbeing Trajectory App · v1.0
### Target: **Lovable** · March 2026 · `CONFIDENTIAL`

---

## ⚡ TL;DR for Lovable

Build a **mobile-first React web app** called **BetonMe**.  
It is a personal wellbeing tracker that shows **smooth trajectory graphs** — not scores, not streaks, not badges.  
Users log one daily action per life area. The graph is the product.  
Tone: calm, neutral, reflective. Think personal journal, not fitness app.

---

## 0. Tech Stack (non-negotiable)

| Layer | Technology |
|---|---|
| Framework | React 18 + TypeScript |
| Styling | Tailwind CSS only — no custom CSS files |
| Components | shadcn/ui — override with BetonMe tokens |
| Charts | Recharts — `LineChart` with `type="monotone"` |
| Animations | Framer Motion |
| Backend | Supabase (auth + database + edge functions) |
| Auth | Supabase magic link (email only, no password) |
| State | Zustand or React Context |
| Mobile breakpoint | 375px primary |
| Max lines/component | 150 lines |

### Lovable Booster — include this in every generation:
```
Technical requirements:
- Use shadcn/ui components where possible
- Tailwind CSS for all styling, no custom CSS
- Framer Motion for all animations and transitions
- Clean component structure, max 150 lines per component
- Mobile-first responsive design, primary breakpoint 375px
- No unnecessary wrapper divs
```

---

## 1. Product Overview

BetonMe is a mobile-first web application that makes the **trajectory of personal wellbeing visible over time** through smooth longitudinal graphs.

It is **not** a habit tracker. It is **not** a goal-setting tool. It is an **observation system** — a neutral mirror of behavioral patterns.

### Core Mechanics
- User defines life areas (Health, Study, Reduce, Finance)
- User logs daily: completed ✓ or not
- System calculates a directional score internally
- A smooth line graph visualizes the resulting trajectory
- The score is **never shown** to the user by default

### Product Philosophy (Lovable must respect this)
| ✅ This product IS | ❌ This product is NOT |
|---|---|
| A visual journal | A fitness app |
| A trajectory viewer | A gamified habit tracker |
| A neutral mirror | A productivity dashboard |
| Calm and reflective | Motivational or pressuring |

---

## 2. Target User

- **Age:** 25–45
- **Profile:** Interested in personal growth, already tries wellbeing habits
- **Pain:** Exhausted by productivity culture and performance pressure
- **Need:** See the direction of behavior over time — without judgment
- **JTBD:** *"When I try to improve aspects of my life, I want to observe the direction of my actions over time, so that I can understand my trajectory without feeling evaluated."*

---

## 3. Design System

> Apply every token below **globally**. Do not override with generic Tailwind defaults.

### 3.1 Color Palette

```css
/* Backgrounds */
--bg-primary:    #0F2F33   /* Main app background — dominant */
--bg-secondary:  #1F4A50   /* Cards, graph containers */
--bg-neutral:    #F4F4F2   /* Modals, light surfaces */

/* Text */
--text-primary:   #EAEAEA  /* Primary text on dark */
--text-secondary: #B9C0C1  /* Secondary, captions */

/* Accent */
--accent:         #E24A4A  /* Sparingly — errors, highlights only */

/* Graph colors — all desaturated */
--graph-positive: #7DA3A0  /* Upward trajectory */
--graph-neutral:  #8C9496  /* Flat / stable */
--graph-decline:  #BFA37A  /* Declining — never aggressive red */
```

> ⚠️ **Never use bright red for negative feedback.** The app must never feel punishing.

### 3.2 Typography

```
Font family: Inter (primary), fallback: SF Pro, Source Sans Pro

Title:     28px / weight 600   → page titles, area names
Section:   18px / weight 500   → section labels
Body:      16px / weight 400   → default text
Secondary: 14px / weight 400   → supporting info
Caption:   12px / weight 400   → labels, axis ticks, helper text
```

### 3.3 Spacing Scale

```
xs:  4px
sm:  8px
md:  16px
lg:  24px
xl:  32px
xxl: 48px
```

> Maintain **generous negative space**. Max **4 components per screen**.

### 3.4 Layout Tokens

```
Graph height:         50–60vh  (graph is dominant)
Container padding:    16px
Card border-radius:   12px
Tap target minimum:   44px × 44px
```

### 3.5 Graph Tokens

```
Line width:          2px
Curve type:          monotone (smooth)
Animation duration:  300ms ease-in-out
Grid opacity:        0.1 (horizontal lines only)
Y-axis:              hidden by default
```

### 3.6 Motion Tokens

```
motion.fast:   200ms ease-in-out
motion.normal: 300ms ease-in-out
motion.slow:   400ms ease-in-out
```

### 3.7 UI Anti-Patterns — NEVER implement

- ❌ Confetti animations
- ❌ Badges or achievement popups
- ❌ Streak counters as primary UI element
- ❌ "Great job!" / "Keep it up!" messaging
- ❌ Red color for missed days
- ❌ Dense information layouts
- ❌ More than 4 components visible per screen
- ❌ Stock photo imagery
- ❌ Generic SaaS card layouts

---

## 4. Information Architecture

### Navigation Structure
```
App
├── Onboarding (unauthenticated)
│   ├── Step 1: Account (magic link)
│   ├── Step 2: Choose areas to observe
│   ├── Step 3: Set frequency
│   └── → Dashboard
│
└── Main App (authenticated)
    ├── [Tab 1] Dashboard (Home)
    ├── [Tab 2] Areas
    ├── [Tab 3] Finance
    └── [Tab 4] Settings
```

### Bottom Navigation Bar
- Fixed at bottom, height 56px, background `#0F2F33`
- 4 tabs: **Home · Areas · Finance · Settings**
- Icons: Lucide icon set, outline style, 24px
- Active tab: `#7DA3A0`, inactive: `#B9C0C1`
- All touch targets ≥ 44px

---

## 5. Database Schema (Supabase)

Enable **Row Level Security on all tables**.

### `users`
```sql
id                    uuid PRIMARY KEY  -- from auth.users
created_at            timestamp
settings_score_visible boolean DEFAULT false
settings_notifications boolean DEFAULT true
```

### `areas`
```sql
id                  uuid PRIMARY KEY DEFAULT gen_random_uuid()
user_id             uuid REFERENCES users(id) ON DELETE CASCADE
name                text NOT NULL
type                text CHECK (type IN ('health','study','reduce','finance'))
frequency_per_week  int DEFAULT 7 CHECK (frequency_per_week BETWEEN 1 AND 7)
created_at          timestamp DEFAULT now()
archived_at         timestamp  -- NULL = active
```

### `checkins`
```sql
id          uuid PRIMARY KEY DEFAULT gen_random_uuid()
area_id     uuid REFERENCES areas(id) ON DELETE CASCADE
user_id     uuid REFERENCES users(id)
date        date NOT NULL
completed   boolean NOT NULL
created_at  timestamp DEFAULT now()
UNIQUE (area_id, date)  -- one check-in per area per day
```

### `score_daily`
```sql
id                 uuid PRIMARY KEY DEFAULT gen_random_uuid()
area_id            uuid REFERENCES areas(id) ON DELETE CASCADE
date               date NOT NULL
daily_score        numeric
cumulative_score   numeric
consecutive_missed int DEFAULT 0
```

### Score Calculation Logic (Edge Function or trigger)
```
completed        → daily_score = +1.0
first missed     → daily_score =  0.0
second missed    → daily_score = -0.5
third+ missed    → daily_score = -1.0

cumulative_score = previous_cumulative + daily_score
```

> ⚠️ The `cumulative_score` feeds the graph line. It is **never displayed as a number** unless `settings_score_visible = true`.

---

## 6. Screen Specifications

---

### Screen 1: Dashboard (Home)

**Purpose:** Primary screen. Opened 95% of the time. The graph IS the screen.

**Layout (top → bottom):**
```
┌─────────────────────────────┐
│ BetonMe          [settings] │  ← Header bar, 56px
├─────────────────────────────┤
│  30d  │  90d  │  365d       │  ← Time range pill selector
├─────────────────────────────┤
│                             │
│   [Area Card: Health]       │  ← 55vh height card
│   [smooth trajectory graph] │
│   [Log today]               │
│                             │
├─────────────────────────────┤
│                             │
│   [Area Card: Study]        │  ← scrollable
│                             │
├─────────────────────────────┤
│  Home · Areas · Finance · ⚙ │  ← Bottom nav
└─────────────────────────────┘
```

**Area Graph Card spec:**

| Property | Value |
|---|---|
| Background | `bg-[#1F4A50] rounded-xl` |
| Card padding | 16px |
| Graph height | 55vh |
| Chart component | Recharts `<LineChart>` |
| Line type | `type="monotone"` |
| Line color logic | Calculate 7-day slope → `#7DA3A0` (up) / `#8C9496` (flat) / `#BFA37A` (down) |
| Grid | Horizontal only, `opacity-10` |
| Y-axis | Hidden |
| X-axis | Date ticks, 12px, `#B9C0C1` |
| Area label | Top-left, 18px semibold, `#EAEAEA` |
| Type badge | Top-right, pill component `<AreaTypePill>` |
| Check-in button | Bottom, full-width, min-h 44px |

**Check-in Button States:**

| State | Style | Text |
|---|---|---|
| Idle (not logged today) | `border border-[#7DA3A0] text-[#EAEAEA] bg-transparent` | "Log today" |
| Completed | `bg-[#7DA3A0]/20 text-[#7DA3A0] border border-[#7DA3A0]` | "Observed" |
| Loading | `opacity-50 cursor-not-allowed` | spinner |

> ✅ After check-in: graph animates (300ms Framer Motion). **No toast. No popup. No confetti. The graph update is the only feedback.**

**Empty State (no areas):**
```
Centered vertically:
  Icon: eye outline (Lucide), 48px, #7DA3A0
  Text: "What do you want to observe?"
  Subtext: "Add a life area to start seeing your trajectory."
  Button: "Add your first area" → navigate to Add Area screen
```

---

### Screen 2: Area Detail

**Purpose:** Deep-dive into a single area's trajectory.

**Layout:**
```
┌─────────────────────────────┐
│ ← [Area Name]    [edit]     │  ← Header with back navigation
├─────────────────────────────┤
│  30d  │  90d  │  365d       │  ← Time range selector
├─────────────────────────────┤
│                             │
│   [Large graph — 60vh]      │
│                             │
├─────────────────────────────┤
│   Last 30 days heatmap      │  ← Calendar row
│   □□■□■■□■□■□□■□■□         │
├─────────────────────────────┤
│   [Score: —]  (if visible)  │  ← Only if settings_score_visible=true
├─────────────────────────────┤
│  Home · Areas · Finance · ⚙ │
└─────────────────────────────┘
```

**Calendar Heatmap spec:**

| Day type | Style |
|---|---|
| Completed | `bg-[#7DA3A0]/60 rounded-sm w-7 h-7` |
| Missed | `bg-[#BFA37A]/40 rounded-sm w-7 h-7` |
| Future | `bg-[#1F4A50]/30 rounded-sm w-7 h-7` |
| Gap | 2px |
| Layout | Single horizontal row, scroll if needed |

> ⚠️ No streak counter. No "best streak" label. No fire emoji. No consecutive-day celebration.

**Edit link:** Small text link at bottom — `"Edit area"` in `#B9C0C1` — NOT a button.

---

### Screen 3: Add / Edit Area

**Purpose:** Create or modify a tracked life area. Must complete in < 10 seconds.

**Form fields (exactly 3):**

| Field | Type | Placeholder / Options |
|---|---|---|
| Area name | Text input | `"e.g. Morning walk"` |
| Type | 4 pill selector | Health · Study · Reduce · Finance |
| Frequency / week | Stepper 1–7 | Default: 7 |

**CTA:** `"Start observing"` — full-width button, `bg-[#7DA3A0] text-[#0F2F33]`

> ⚠️ Never use: "Create goal", "Set target", "Begin challenge", "Track habit".

**Type pill styles:**

| Type | Style |
|---|---|
| Health | `bg-[#7DA3A0]/20 text-[#7DA3A0] border border-[#7DA3A0]/40` |
| Study | `bg-[#8C9496]/20 text-[#B9C0C1] border border-[#8C9496]/40` |
| Reduce | `bg-[#BFA37A]/20 text-[#BFA37A] border border-[#BFA37A]/40` |
| Finance | `bg-[#1F4A50] text-[#EAEAEA] border border-[#7DA3A0]/30` |

---

### Screen 4: Finance Projection

**Purpose:** Show the finance area with a simple forward projection line.

**Layout:**
```
┌─────────────────────────────┐
│ ← Finance                   │
├─────────────────────────────┤
│   Historical trajectory     │
│   [graph — solid line]      │
│   + Projection [dashed]     │
├─────────────────────────────┤
│   "30-day estimate based    │
│    on your current trend."  │
├─────────────────────────────┤
│  Home · Areas · Finance · ⚙ │
└─────────────────────────────┘
```

**Projection line:** Same color as graph line, `strokeDasharray="4 4"`, calculated as simple linear regression of last 30 days.

> ⚠️ No savings goal input. No target amount. No "you need to save X". Pure trajectory projection only.

---

### Screen 5: Settings

**Purpose:** Minimal preference screen. No clutter.

| Setting | Type | Default |
|---|---|---|
| Show trajectory score | Toggle | OFF |
| Notifications | Toggle | ON |
| Account email | Display only | — |
| Sign out | Button | — |
| Delete account | Destructive button | — (requires confirmation) |

**Delete confirmation copy:** `"This will permanently delete all your observation history. This cannot be undone."` — NOT "Are you sure you want to leave?"

---

## 7. Onboarding Flow

4 steps. No tutorial overlays. No skip buttons in steps 1–3.

### Step 1 — Account
- Single email input
- CTA: `"Send me a link"`
- Below input: `"We'll send you a magic link. No password needed."`
- No social auth in MVP

### Step 2 — What do you want to observe?
- Header: `"What do you want to observe?"`
- 4 pre-built suggestion chips (tap to add):
  - 🚶 Morning movement
  - 📖 Daily reading
  - 📵 Less screen time
  - 💰 Monthly saving
- `+ Add custom` text link for freeform input
- CTA: `"Continue"` (disabled until ≥1 area selected)

### Step 3 — How often?
- Per-area frequency stepper (1–7 days/week)
- Label: `"How many days per week?"` — NOT "Set your goal"
- CTA: `"Start observing"`

### Step 4 — Dashboard
- Immediate redirect to Dashboard
- ❌ No "You're all set!" screen
- ❌ No congratulations animation
- ❌ No confetti

> Copy rule: avoid "Great!", "You're all set!", "Let's go!", "Start your journey!"

---

## 8. Core User Flows

### Flow A — Daily Check-in (< 10 seconds)
```
1. Open app
2. See Dashboard with trajectory graphs
3. Tap "Log today" on an area card
4. Button state → "Observed"
5. Graph animates update (300ms Framer Motion)
6. Done — no popup, no toast, no reward
```

### Flow B — Trend Analysis
```
1. Tap on area card (body, not button)
2. Navigate to Area Detail
3. Switch time range: 30d → 90d → 365d
4. Observe trajectory at different scales
5. Optional: view calendar heatmap
```

### Flow C — Add Area
```
1. Tap "Add area" from Dashboard empty state
2. Fill 3-field form (< 10 seconds)
3. Tap "Start observing"
4. Return to Dashboard — new card visible
5. No celebration screen
```

---

## 9. Component Library

### `<TrajectoryCard>`
```typescript
interface TrajectoryCardProps {
  areaId: string
  areaName: string
  areaType: 'health' | 'study' | 'reduce' | 'finance'
  data: Array<{ date: string; score: number }>
  timeRange: 30 | 90 | 365
  isLoggedToday: boolean
  onCheckIn: () => void
}
```
- Recharts `<LineChart>` with `type="monotone"`
- Line color: computed from 7-day slope of `data`
- Framer Motion: `animate` on mount + on `data` change
- Height: `h-[55vh]`
- Background: `bg-[#1F4A50] rounded-xl p-4`

### `<CheckInButton>`
```typescript
interface CheckInButtonProps {
  isCompleted: boolean
  isLoading: boolean
  onPress: () => void
}
// idle:      border border-[#7DA3A0] text-[#EAEAEA] bg-transparent
// completed: bg-[#7DA3A0]/20 text-[#7DA3A0] border border-[#7DA3A0]
// min-h-[44px] w-full rounded-lg font-medium text-[16px]
```

### `<TimeRangeSelector>`
```typescript
// Options: [30, 90, 365] labeled as "30d" | "90d" | "365d"
// Active:   bg-[#1F4A50] text-[#EAEAEA] rounded-full px-4 py-1
// Inactive: text-[#B9C0C1]
// Transition: 200ms ease-in-out via Framer Motion layoutId
```

### `<AreaTypePill>`
```typescript
// health:  bg-[#7DA3A0]/20  text-[#7DA3A0]  border-[#7DA3A0]/40
// study:   bg-[#8C9496]/20  text-[#B9C0C1]  border-[#8C9496]/40
// reduce:  bg-[#BFA37A]/20  text-[#BFA37A]  border-[#BFA37A]/40
// finance: bg-[#1F4A50]     text-[#EAEAEA]  border-[#7DA3A0]/30
// All: rounded-full px-3 py-0.5 text-[12px] font-medium border
```

### `<CalendarHeatmap>`
```typescript
// 30 cells, single row, horizontal scroll
// Cell: w-7 h-7 rounded-sm
// completed: bg-[#7DA3A0]/60
// missed:    bg-[#BFA37A]/40
// future:    bg-[#1F4A50]/30
// gap:       gap-0.5
```

---

## 10. UI States

### Empty States

| Context | Visual | Copy | CTA |
|---|---|---|---|
| Dashboard — no areas | Eye icon 48px `#7DA3A0` | "What do you want to observe?" | "Add your first area" |
| Area Detail — no data | Graph icon, muted | "Keep observing. Your trajectory takes shape over days." | — |
| Finance — no data | Chart icon | "Log your first finance check-in to see your trend." | — |

### Loading States

| Context | Pattern |
|---|---|
| Dashboard initial load | Skeleton cards `bg-[#1F4A50] animate-pulse rounded-xl h-[55vh]` |
| Graph update after check-in | Framer Motion 300ms transition on data |
| Area creation | Inline spinner in button — no full-screen loader |
| Auth link sent | Static confirmation screen — no loader needed |

### Error States

| Context | Pattern | Copy |
|---|---|---|
| Network error | Bottom toast, non-blocking | "Couldn't sync. Will retry automatically." |
| Auth error | Redirect to login | "Session expired. Sign in again." |
| Form validation | Inline below field, `text-[#E24A4A] text-sm` | Field-specific message |

### Success States

| Action | Feedback |
|---|---|
| Check-in logged | Button state change + graph animation only |
| Area created | Navigate to Dashboard, new card visible |
| Settings saved | Silent save — no toast |
| Account deleted | Redirect to unauthenticated home, no message |

> ⚠️ **No confetti. No "Great job!". The graph update is always the only positive feedback.**

---

## 11. Copy & Tone of Voice

### Brand Voice
**Calm · Neutral · Observational**  
The app never evaluates the user. It only describes what it sees.

### Allowed Vocabulary
```
trend · trajectory · direction · pattern · change over time
observe · notice · see
your data · your trajectory · over the past N days
```

### Forbidden Vocabulary
```
success · achievement · failure · win
streak · chain · combo
great job · well done · keep it up · you're crushing it
goal · target · challenge · level · reward
don't break the chain · you missed · you failed
```

### Key UI String Reference

| Location | String |
|---|---|
| App tagline | "Observe your direction." |
| Onboarding step 2 header | "What do you want to observe?" |
| Add area CTA | "Start observing" |
| Check-in button (idle) | "Log today" |
| Check-in button (done) | "Observed" |
| Dashboard empty state | "What do you want to observe?" |
| Finance projection label | "30-day estimate based on your current trend." |
| Score toggle label | "Show trajectory score" |
| Delete account confirm | "This will permanently delete all your observation history." |
| Onboarding area step label | "How many days per week?" |

---

## 12. Accessibility

| Rule | Value |
|---|---|
| Minimum contrast ratio | 4.5:1 |
| All tap targets | ≥ 44 × 44px |
| Font scaling | enabled (respect system settings) |
| Trend communication | slope + position + color (never color alone) |
| Focus indicators | visible on all interactive elements |

---

## 13. Scope Boundaries

### ✅ In MVP
- Magic link authentication
- 4 life area types (Health, Study, Reduce, Finance)
- Binary daily check-in (done / not done)
- Smooth trajectory graph per area
- Time ranges: 30d / 90d / 365d
- 30-day calendar heatmap
- Finance linear projection
- Score visibility toggle (default off)
- Notification toggle

### ❌ Explicitly Out of MVP
- Social features or sharing
- Badges, points, levels, leaderboards
- Streak gamification
- Custom graph color picker
- Push notifications (toggle exists, logic deferred)
- Desktop-optimized layout
- Export / data download
- Multiple users / households
- AI suggestions or coaching

---

## 14. Success Metrics

| Metric | Target |
|---|---|
| Activation | User creates ≥1 area and logs for 7 consecutive days |
| Engagement | Weekly logging rate ≥ 60% of user-defined frequency |
| Retention | User still active at day 30 |
| Core value | ≥3 graph views per week per active user |

---

## 15. Pre-Generation Checklist for Lovable

Before generating, confirm:

- [x] Target audience defined
- [x] Validation hypothesis clear
- [x] Page architecture confirmed (5 screens + onboarding)
- [x] Copy strings written (no placeholders)
- [x] Design tokens specified (colors, spacing, typography)
- [x] Competitive differentiation clear (anti-fitness app, anti-gamification)
- [x] All UI states specified (empty, loading, error, success)
- [x] Lovable technical booster included
- [x] Anti-patterns explicitly listed

---

*BetonMe PRD v1.0 — Ready for Lovable generation*
