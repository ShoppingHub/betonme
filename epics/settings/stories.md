# Stories — Epic 07 — Settings

## Sequenza di implementazione

```
story-07-01 → Layout Settings: toggle + sezione account
story-07-02 → Toggle score: aggiornamento settings_score_visible su Supabase
story-07-03 → Sign out con redirect
story-07-04 → Delete account con modale e cascade
```

> **Dipendenze:** Richiede Epic 00 (auth) completato. Logout e delete account si appoggiano a Supabase Auth e all'Edge Function creata in story-00-06.

---

## story-07-01 — Layout Settings

BetonMe è un'app di osservazione del benessere. Costruisci la schermata Settings (route `/settings`).

**Cosa mostra, dall'alto verso il basso:**
1. Header: `"Settings"`
2. Toggle: `"Show trajectory score"` — default OFF
3. Toggle: `"Notifications"` — default ON
4. Label sezione: `"Account"`
5. Email dell'utente (read-only, testo colore secondario)
6. Bottone: `"Sign out"`
7. Bottone: `"Delete account"` (testo in colore errore)

**Behavior toggle:**
- I toggle usano il componente shadcn `<Switch>` con override del colore attivo (`#7DA3A0`)
- Il salvataggio avviene in modo silenzioso — nessun toast, nessun messaggio di conferma
- Toggle Notifications: in MVP salva la preferenza ma non attiva push notification reali

**Note:**
- Esattamente 5 elementi (2 toggle, email, sign out, delete) — nessun altro elemento
- Email molto lunga → troncata con ellipsis

---

## story-07-02 — Toggle score

Continua Epic 07 di BetonMe. Implementa la logica del toggle "Show trajectory score".

**Behavior:**
- Al cambio del toggle → aggiornamento silenzioso di `settings_score_visible` (boolean) nella tabella `users` per l'utente corrente
- Se `true` → il punteggio numerico diventa visibile nell'Area Detail (story-04-03)
- Se `false` → il punteggio non appare da nessuna parte nell'app
- Salvataggio ottimistico: il toggle cambia stato subito, l'aggiornamento Supabase avviene in background
- In caso di errore → il toggle torna al valore precedente, nessun messaggio visibile

---

## story-07-03 — Sign out

Continua Epic 07 di BetonMe. Implementa il logout.

**Behavior:**
- Tap su `"Sign out"` → `supabase.auth.signOut()`
- Il bottone mostra opacity ridotta durante la chiamata
- Sessione invalidata → redirect a `/login`
- Nessun modale di conferma
- Nessun messaggio di addio

---

## story-07-04 — Delete account

Continua Epic 07 di BetonMe. Implementa la cancellazione account.

**Behavior:**
- Tap su `"Delete account"` → si apre un modale di conferma

**Modale:**
- Titolo: `"Delete account"`
- Corpo: `"This will permanently delete all your observation history. This cannot be undone."`
- CTA conferma: `"Delete permanently"` (testo in colore errore `#E24A4A`)
- CTA annulla: `"Cancel"` → chiude il modale, nessuna azione

**Al tap su `"Delete permanently"`:**
1. CTA mostra spinner + opacity ridotta
2. Chiamata all'Edge Function Supabase (già creata in story-00-06) che elimina l'account con cascade su `areas`, `checkins`, `score_daily`, `users`
3. Sessione invalidata
4. Redirect a `/login`
5. Nessun messaggio di addio

**Edge case:**
- Connessione assente durante il delete → errore inline sotto la CTA, nessuna azione eseguita, modale rimane aperto
