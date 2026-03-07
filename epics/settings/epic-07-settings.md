# Epic 07 — Settings

## Obiettivo
Offrire all'utente un'unica schermata minimale per gestire le preferenze dell'app e il proprio account, senza clutter e senza opzioni superflue.

---

## Behavior

La schermata Settings è intenzionalmente semplice. Contiene solo ciò che è strettamente necessario per l'MVP. Nessuna sezione espandibile, nessun accordion, nessuna lista di opzioni avanzate.

I salvataggi sono silenziosi — nessun toast, nessun messaggio di conferma per le azioni non distruttive.

---

## Impostazioni Disponibili

| Setting | Tipo | Default | Note |
|---|---|---|---|
| Show trajectory score | Toggle | OFF | Mostra il punteggio numerico nell'Area Detail |
| Notifications | Toggle | ON | Logica push rinviata al post-MVP |
| Account email | Testo (read-only) | — | Non modificabile |
| Sign out | Bottone | — | |
| Delete account | Bottone distruttivo | — | Richiede conferma |

---

## Flussi

### Toggle "Show trajectory score"
1. L'utente attiva il toggle
2. La preferenza `settings_score_visible` viene aggiornata su Supabase
3. Salvataggio silenzioso — nessun feedback visivo
4. Il punteggio numerico diventa visibile nell'Area Detail

### Toggle "Notifications"
1. L'utente attiva/disattiva il toggle
2. La preferenza `settings_notifications` viene aggiornata su Supabase
3. Salvataggio silenzioso
4. La logica di push notification è fuori scope MVP

### Sign Out
1. L'utente tappa "Sign out"
2. La sessione Supabase viene terminata
3. Redirect alla schermata di onboarding (Step 1 — magic link)

### Delete Account
1. L'utente tappa "Delete account"
2. Appare un modale di conferma con il copy specifico
3. L'utente tappa "Delete permanently"
4. Account e dati vengono eliminati (cascade su tutte le tabelle)
5. Redirect alla schermata iniziale non autenticata
6. Nessun messaggio di addio

---

## Layout

```
┌─────────────────────────────┐
│ Settings                    │  ← Header
├─────────────────────────────┤
│                             │
│  Show trajectory score  [○] │  ← Toggle (default OFF)
│                             │
│  Notifications          [●] │  ← Toggle (default ON)
│                             │
│  Account                    │  ← Label sezione
│  user@email.com             │  ← Email read-only, text-[#B9C0C1]
│                             │
│  [Sign out]                 │  ← Bottone standard
│                             │
│  [Delete account]           │  ← Bottone distruttivo
│                             │
├─────────────────────────────┤
│  Home · Areas · Finance · ⚙ │
└─────────────────────────────┘
```

---

## Modale Delete Account

```
Titolo:  "Delete account"
Corpo:   "This will permanently delete all your observation history.
          This cannot be undone."
CTA:     "Delete permanently"  ← text-[#E24A4A]
Cancel:  "Cancel"              ← testo neutro, chiude il modale
```

> ⚠️ Non usare "Are you sure you want to leave?" o altre frasi generiche.

---

## Stati UI

| Stato | Comportamento |
|---|---|
| Toggle OFF | Stile inattivo standard shadcn |
| Toggle ON | Stile attivo con colore `#7DA3A0` |
| Salvataggio toggle | Silenzioso, nessun feedback |
| Sign out loading | Bottone opacity-50 durante la chiamata |
| Modale delete | Overlay semi-trasparente su background `#0F2F33` |
| Delete in corso | CTA "Delete permanently" opacity-50 + spinner |

---

## Edge Case

- Utente tappa "Delete permanently" e la connessione cade → errore non bloccante, nessuna azione eseguita
- Toggle notifications attivato → nessuna push notification reale (fuori scope MVP)
- Email molto lunga → troncata con `text-ellipsis overflow-hidden`

---

## Acceptance Criteria

- [ ] La schermata mostra esattamente i 5 elementi specificati (2 toggle, email, sign out, delete)
- [ ] Il toggle "Show trajectory score" aggiorna `settings_score_visible` su Supabase
- [ ] Il salvataggio dei toggle è silenzioso (nessun toast)
- [ ] "Sign out" termina la sessione e redirige all'onboarding
- [ ] "Delete account" apre il modale con il copy esatto specificato
- [ ] "Delete permanently" elimina account e dati con cascade
- [ ] Il modale ha CTA distruttiva in `#E24A4A`
- [ ] Dopo il delete, redirect alla schermata non autenticata senza messaggi

---

## Copy UI

| Elemento | Stringa |
|---|---|
| Header | `"Settings"` |
| Label toggle score | `"Show trajectory score"` |
| Label toggle notifiche | `"Notifications"` |
| Label sezione account | `"Account"` |
| Bottone sign out | `"Sign out"` |
| Bottone delete | `"Delete account"` |
| Titolo modale | `"Delete account"` |
| Corpo modale | `"This will permanently delete all your observation history. This cannot be undone."` |
| CTA modale confirm | `"Delete permanently"` |
| CTA modale cancel | `"Cancel"` |

---

## Note UI

- Il bottone "Delete account" ha colore testo `#E24A4A` (accent — uso consentito per azioni distruttive)
- Brand tokens: `brand-system/betonme_brand_system_lovable.md`
- Toggle usa shadcn/ui `<Switch>` con override colore active: `#7DA3A0`

---

## Stories

- `story-07-01` — Layout Settings con toggle e sezione account
- `story-07-02` — Logica toggle score e salvataggio su Supabase
- `story-07-03` — Sign out con redirect
- `story-07-04` — Delete account con modale di conferma e cascade
