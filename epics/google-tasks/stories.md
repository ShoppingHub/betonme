# Stories — Epic 13: Google Tasks Sync

## Sequenza di implementazione

Le stories vanno eseguite in ordine. Ogni story è un prompt atomico per Lovable.

---

## story-13-01 — Collegamento account Google in Settings

**Dipendenze:** Epic 07 (Settings)

Aggiungi una nuova sezione "Google Tasks" nella schermata Settings, dopo la sezione Preferenze e prima di Account.

La sezione ha due stati:

**Stato: non collegato**
- Testo descrittivo: "Sincronizza le tue attività con Google Tasks / Sync your activities with Google Tasks"
- Bottone "Collega Google / Connect Google"
- Al tap, avvia il flusso OAuth di Google con scope `tasks`
- Durante il flusso: bottone disabilitato + spinner
- Dopo l'autorizzazione: salva il token su Supabase, aggiorna la UI allo stato "collegato"

**Stato: collegato**
- Testo "Connesso come {email} / Connected as {email}" (email read-only)
- Bottone "Scollega / Disconnect"
- Al tap su Scollega: modale di conferma con copy specificato nell'epic
- Conferma: disconnette il token, disabilita la sync su tutte le attività, torna allo stato "non collegato"

**Stato: auth error (token scaduto/revocato)**
- Banner inline nella sezione: "Connessione Google scaduta. Ricollega il tuo account. / Google connection expired. Reconnect your account."
- Bottone "Ricollega / Reconnect" che riavvia il flusso OAuth

Nuovo DB: tabella `google_oauth_tokens` (user_id, google_email, refresh_token, access_token, access_token_expires_at, connected_at, status). Refresh token conservato in modo sicuro.

---

## story-13-02 — Toggle sync in Edit Area

**Dipendenze:** story-13-01, Epic 05 (Edit Area)

Aggiungi il toggle "Sync con Google Tasks / Sync with Google Tasks" nella schermata Edit Area.

Regole di visibilità:
- Visibile solo se l'account Google è collegato (status = 'active')
- Visibile solo se l'attività è `tracking_mode = binary`
- Nascosto in tutti gli altri casi (nessun placeholder, nessun messaggio)

Comportamento toggle ON:
- Salvataggio silenzioso del campo `google_sync_enabled = true` su `areas`
- Sotto il toggle appare il testo: "I task vengono creati su Google Tasks / Tasks are created in Google Tasks"
- Triggera la generazione dei task (story-13-03)

Comportamento toggle OFF:
- `google_sync_enabled = false`
- Triggera la cancellazione dei task futuri non completati (story-13-03)
- Salvataggio silenzioso

---

## story-13-03 — Generazione task Google alla settimana

**Dipendenze:** story-13-02

Implementa la logica di creazione e cancellazione dei task su Google Tasks.

**Creazione task (trigger: toggle ON o inizio nuova settimana):**

Per ogni attività con `google_sync_enabled = true` e `tracking_mode = binary`:
1. Crea (se non esiste) la TaskList "opad.me" tramite Google Tasks API
2. Calcola le date di occorrenza per le prossime 2 settimane (settimana corrente + prossima) in base a `frequency_per_week`, usando la distribuzione predefinita:
   - 1x → Lunedì
   - 2x → Lunedì, Giovedì
   - 3x → Lunedì, Mercoledì, Venerdì
   - 4x → Lunedì, Martedì, Giovedì, Venerdì
   - 5x → Lunedì-Venerdì
   - 6x → Lunedì-Sabato
   - 7x → tutti i giorni
3. Per ogni data: crea il task su Google con titolo = nome attività, `due` = data, `notes` = `[opad:{area_id}:{date}]`
4. Salva il mapping in `google_tasks_sync` (area_id, user_id, sync_date, google_task_id, google_tasklist_id, status='pending')
5. Non creare duplicati: controlla che non esista già un record in `google_tasks_sync` per (area_id, sync_date)

**Cancellazione task (trigger: toggle OFF o scollegamento Google):**
1. Recupera tutti i task futuri con `status = 'pending'` da `google_tasks_sync` per quell'area
2. Cancella ogni task su Google Tasks API
3. Elimina i record da `google_tasks_sync`
4. I task con `status = 'completed'` non vengono toccati

**Nuovo inizio settimana:**
All'apertura dell'app, se è lunedì (o se l'ultima generazione è antecedente all'inizio della settimana corrente), genera i task per la settimana successiva per tutte le attività con sync attiva.

---

## story-13-04 — Push opad → Google al check-in

**Dipendenze:** story-13-03, Epic 03 (Check-in)

Ogni volta che viene registrato o rimosso un check-in per un'attività con `google_sync_enabled = true`:

**Check-in completato:**
1. Cerca in `google_tasks_sync` il record con `area_id` + `sync_date = data_checkin`
2. Se trovato: chiama Google Tasks API `PATCH` con `status: "completed"`
3. Aggiorna `google_tasks_sync.status = 'completed'` e `last_synced_at`
4. Se non trovato (check-in su giorno non pianificato): nessuna azione su Google

**Check-in rimosso:**
1. Cerca in `google_tasks_sync` il record con `area_id` + `sync_date = data_checkin`
2. Se trovato: chiama Google Tasks API `PATCH` con `status: "needsAction"`, rimuove il campo `completed`
3. Aggiorna `google_tasks_sync.status = 'pending'` e `last_synced_at`

Errore di rete durante il push: il check-in è già salvato in opad (non rollbackare). L'errore viene ignorato silenziosamente — il pull successivo riallineerà lo stato.

---

## story-13-05 — Pull Google → opad all'apertura app

**Dipendenze:** story-13-04

All'apertura dell'app (mount del componente principale), se l'utente ha almeno un'attività con `google_sync_enabled = true` e lo stato OAuth è 'active':

1. Recupera il `last_synced_at` più recente da `google_tasks_sync` per questo utente
2. Chiama Google Tasks API: `GET /lists/{tasklist_id}/tasks?updatedMin={last_synced_at}&showCompleted=true&showHidden=true&showDeleted=true`
3. Per ogni task nella risposta, estrai `area_id` e `date` dal campo `notes` (pattern `[opad:{area_id}:{date}]`)
4. Se `deleted: true` → nessuna azione
5. Se `status: "completed"` e nessun check-in locale per (area_id, date) → crea il check-in
6. Se `status: "needsAction"` e check-in locale presente per (area_id, date) → rimuovi il check-in
7. Aggiorna `last_synced_at` in tutti i record `google_tasks_sync` processati
8. Se la risposta restituisce `401` o `invalid_grant` → imposta `google_oauth_tokens.status = 'auth_error'`, non mostrare errori all'utente (il banner in Settings apparirà al prossimo accesso)

Il pull avviene in background e non blocca il caricamento dell'app. La UI si aggiorna dopo il completamento del pull (re-fetch delle attività).

Se non c'è connessione internet: skip silenzioso, nessun errore mostrato.

---

## story-13-06 — Gestione attività archiviata con sync attiva

**Dipendenze:** story-13-05, Epic 05 (Edit Area)

Quando un'attività con `google_sync_enabled = true` viene archiviata:
1. Esegui la stessa logica di "toggle OFF" (story-13-03): cancella i task futuri non completati da Google
2. Imposta `google_sync_enabled = false`

Questo garantisce che l'archiviazione non lasci task orfani in Google Tasks.

---

## Note di implementazione

- Il pull (story-13-05) è frontend-only per l'MVP — nessun cron job, nessuna Edge Function schedulata.
- Il refresh del token OAuth avviene client-side: se `access_token_expires_at` è nel passato, chiama il token endpoint di Google prima di ogni API call.
- Tutti i campi `notes` nei task Google devono contenere il marker `[opad:{area_id}:{date}]` per essere riconosciuti durante il pull. Task senza marker vengono ignorati.
- L'ordine delle stories rispecchia le dipendenze tecniche: non saltare story-13-01/02/03 prima di implementare la sync bidirezionale.
