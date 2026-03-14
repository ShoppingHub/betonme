# Epic 13 — Google Tasks Sync

## Obiettivo

Permettere all'utente di sincronizzare le proprie attività binarie con Google Tasks, così da gestirle anche dall'app Google Tasks sul proprio dispositivo.

opad.me è e rimane la **source of truth**: tracciamento, storico e grafici vivono solo in opad. Google Tasks è un layer operativo esterno — un promemoria su cui spuntare le attività.

---

## Principi

- **Opzionale**: la sync non è attiva per default su nessuna attività
- **Per attività**: ogni area può avere la sync abilitata o disabilitata indipendentemente
- **Solo attività binarie**: le attività `quantity_reduce` non sono sincronizzabili
- **Google non comanda**: le azioni su Google (completamento, eliminazione) impattano solo lo stato `completato/non completato` in opad. Il resto (nome, frequenza, storico) rimane esclusivamente in opad.
- **Sincronizzazione bidirezionale sullo stato**: completare un'attività su opad → task completato su Google; completare un task su Google → check-in registrato su opad

---

## Come funziona

### Collegamento account Google

L'utente collega il proprio account Google dalla schermata Settings. Il collegamento è unico per account opad — non per singola attività.

Flusso:
1. L'utente tappa "Collega Google / Connect Google" in Settings
2. Si apre il flusso OAuth di Google (scope: `tasks`)
3. Dopo l'autorizzazione, l'account risulta collegato
4. In Settings appare l'email Google collegata e il bottone "Scollega"

Lo scollegamento disabilita la sync su tutte le attività e cancella i task futuri da Google.

### Abilitare la sync su un'attività

Il toggle "Sync con Google Tasks / Sync with Google Tasks" appare nella schermata Edit Area, solo se:
- L'account Google è collegato
- L'attività è `tracking_mode = binary`

Quando l'utente attiva il toggle:
1. opad crea una TaskList dedicata in Google Tasks chiamata `"opad.me"` (se non esiste già)
2. Vengono creati i task per la settimana corrente e la settimana successiva
3. Ogni task rappresenta un'occorrenza pianificata dell'attività (vedi sezione Task Generation)
4. Salvataggio silenzioso — nessun feedback visivo oltre al toggle acceso

Quando l'utente disattiva il toggle:
1. Tutti i task futuri **non completati** vengono eliminati da Google Tasks
2. I task già completati rimangono (sono storia) — non vengono toccati
3. Salvataggio silenzioso

### Generazione dei task (Task Generation)

Ogni attività con sync abilitata e `tracking_mode = binary` genera N task per settimana, dove N = `frequency_per_week`.

I task vengono distribuiti nei giorni della settimana (lunedì–domenica) secondo questa logica predefinita:

| frequency_per_week | Giorni |
|---|---|
| 1 | Lunedì |
| 2 | Lunedì, Giovedì |
| 3 | Lunedì, Mercoledì, Venerdì |
| 4 | Lunedì, Martedì, Giovedì, Venerdì |
| 5 | Lunedì → Venerdì |
| 6 | Lunedì → Sabato |
| 7 | Tutti i giorni |

Il **titolo** del task in Google è il nome dell'attività (es. `"Corsa"`).
La **data di scadenza** (`due`) è il giorno pianificato della settimana (senza orario).
Il campo **notes** contiene un identificatore opaco: `[opad:area_id:date]` — usato per evitare loop di sync.

### Orizzonte a 2 settimane

opad mantiene un orizzonte di 2 settimane di task creati in avanti:

- **All'attivazione della sync**: task creati per la settimana corrente + prossima settimana
- **Ogni inizio settimana**: task creati per la settimana successiva (automatico, all'apertura dell'app)
- L'utente vede sempre almeno una settimana piena di task in Google, spesso due

### Sincronizzazione bidirezionale

**opad → Google** (push immediato, ad ogni check-in):

| Evento in opad | Azione su Google |
|---|---|
| Check-in completato | `status: "completed"` sul task con `due = data` |
| Check-in rimosso | `status: "needsAction"` sul task con `due = data` |
| Check-in su giorno non pianificato | Nessuna azione (nessun task da aggiornare) |

**Google → opad** (pull al lancio dell'app):

All'apertura dell'app, se l'utente ha almeno un'attività con sync abilitata, opad interroga Google Tasks per i task modificati dall'ultimo sync (`updatedMin = last_synced_at`). La risposta include task completati e nascosti (`showCompleted=true`, `showHidden=true`, `showDeleted=true`).

| Stato trovato in Google | Azione in opad |
|---|---|
| `status: "completed"` — nessun check-in locale | Crea check-in per `area_id` + `due_date` |
| `status: "needsAction"` — check-in locale presente | Rimuove il check-in per `area_id` + `due_date` |
| `deleted: true` | Nessuna azione — opad ricrea il task al prossimo sync settimanale |
| Task non trovato (scomparso) | Nessuna azione |

**Conflict resolution**: last-write-wins sul campo `status`. opad è source of truth per tutto il resto.

---

## Flussi utente

### Flusso 1 — Collegare l'account Google

```
Settings → tappa "Collega Google"
  → flusso OAuth Google (scope: tasks)
  → autorizzazione concessa
  → Settings mostra: "Connesso come user@gmail.com" + "Scollega"
```

### Flusso 2 — Abilitare sync su un'attività

```
Edit Area → toggle "Sync con Google Tasks" (visibile solo se Google collegato + binary)
  → toggle ON
  → task creati in Google per settimana corrente + prossima
  → (silenzioso)
```

### Flusso 3 — Check-in su opad

```
Home o Area Detail → tappa "Log today"
  → check-in salvato in opad
  → se sync attiva → task Google con due=oggi → PATCH status: "completed"
  → (silenzioso, nessun feedback extra)
```

### Flusso 4 — Completamento da Google Tasks

```
Utente completa task in Google Tasks app
  → apertura opad
  → pull Google Tasks (updatedMin = last_synced_at)
  → task "Corsa" due=2026-03-14 status=completed
  → check-in creato in opad per area "Corsa" date=2026-03-14
  → (silenzioso, solo la UI si aggiorna)
```

### Flusso 5 — Scollegare Google

```
Settings → tappa "Scollega"
  → conferma modale
  → tutti i task futuri non completati eliminati da Google
  → sync disabilitata su tutte le attività
  → Settings torna a mostrare "Collega Google"
```

---

## Stati UI

### Settings — sezione Google Tasks

| Stato | UI |
|---|---|
| Non collegato | Bottone "Collega Google / Connect Google" + descrizione 1 riga |
| Collegamento in corso | Bottone disabilitato + spinner |
| Collegato | Email Google (read-only) + bottone "Scollega / Disconnect" |
| Errore auth (token revocato) | Banner inline: "Connessione Google scaduta. Ricollega il tuo account." + bottone "Ricollega" |

### Edit Area — toggle sync

| Stato | UI |
|---|---|
| Google non collegato | Toggle nascosto |
| Attività `quantity_reduce` | Toggle nascosto |
| Google collegato + binary | Toggle "Sync con Google Tasks / Sync with Google Tasks" |
| Toggle ON | Indicatore attivo + testo sub: "I task vengono creati su Google Tasks / Tasks are created in Google Tasks" |
| Toggle OFF | Indicatore inattivo |

---

## Edge Case

| Scenario | Comportamento |
|---|---|
| Token OAuth scaduto / revocato | La sync si interrompe silenziosamente. In Settings appare il banner "Connessione Google scaduta". Nessun dato viene perso in opad. |
| Task eliminato da Google | Nessuna azione in opad. Il task viene ricreato all'inizio della settimana successiva (se la sync è ancora attiva). |
| Check-in su giorno non pianificato | Non crea task in Google (nessun task corrispondente). |
| Attività modificata (frequency_per_week cambia) | I task per la settimana corrente rimangono invariati. I nuovi task della settimana successiva rispecchiano la nuova frequenza. |
| Attività archiviata con sync attiva | La sync viene disabilitata automaticamente. I task futuri non completati vengono eliminati da Google. |
| Account Google scollegato con sync attiva su più attività | Tutte le sync vengono disabilitate. I task futuri non completati vengono eliminati. |
| Apertura app senza connessione internet | Il pull viene saltato. Nessun errore visibile. Il pull avviene alla prossima apertura con connessione. |
| Quota API Google esaurita | La sync si interrompe silenziosamente. Nessun errore mostrato all'utente. |

---

## Database

Nuove tabelle:

```
google_oauth_tokens
  user_id              FK → users.id  (unique per user)
  google_email         TEXT
  refresh_token        TEXT  (stored encrypted in Supabase Vault)
  access_token         TEXT  (short-lived, cached)
  access_token_expires_at  TIMESTAMPTZ
  connected_at         TIMESTAMPTZ
  status               TEXT  CHECK ('active' | 'auth_error')

google_tasks_sync
  id                   UUID  PK
  area_id              FK → areas.id
  user_id              FK → users.id
  sync_date            DATE                ← data dell'occorrenza pianificata
  google_task_id       TEXT                ← ID opaco di Google
  google_tasklist_id   TEXT
  status               TEXT  CHECK ('pending' | 'completed' | 'deleted')
  last_synced_at       TIMESTAMPTZ
  UNIQUE(area_id, sync_date)
```

Modifica tabella `areas`:
```
google_sync_enabled    BOOLEAN  DEFAULT false
```

---

## Acceptance Criteria

- [ ] Settings mostra sezione "Google Tasks" con stato connessione
- [ ] Il flusso OAuth Google funziona e salva il token
- [ ] L'email Google collegata è visibile in Settings (read-only)
- [ ] Il toggle sync appare in Edit Area solo per attività binary con Google collegato
- [ ] All'attivazione del toggle, i task vengono creati su Google Tasks per le prossime 2 settimane
- [ ] Ogni task ha titolo = nome attività, due = data pianificata, notes con marker opad
- [ ] I task sono creati nella lista "opad.me" (creata automaticamente se non esiste)
- [ ] Check-in su opad → task Google marcato `completed`
- [ ] Rimozione check-in su opad → task Google tornato `needsAction`
- [ ] All'apertura dell'app, pull Google Tasks per task modificati dall'ultimo sync
- [ ] Task completato su Google → check-in creato in opad per la data corrispondente
- [ ] Task riportato a needsAction su Google → check-in rimosso in opad
- [ ] Task eliminato su Google → nessuna azione in opad
- [ ] Disattivazione toggle → task futuri non completati eliminati da Google
- [ ] Scollegamento Google → sync disabilitata su tutte le attività + task futuri eliminati
- [ ] Token revocato → banner in Settings, nessun crash, nessun dato perso
- [ ] Le attività `quantity_reduce` non mostrano il toggle sync
- [ ] L'inizio di ogni nuova settimana genera i task per la settimana successiva

---

## Copy UI

| Elemento | IT | EN |
|---|---|---|
| Label sezione Settings | `"Google Tasks"` | `"Google Tasks"` |
| Bottone collega | `"Collega Google"` | `"Connect Google"` |
| Sub bottone collega | `"Sincronizza le tue attività con Google Tasks"` | `"Sync your activities with Google Tasks"` |
| Stato connesso (email) | `"Connesso come {email}"` | `"Connected as {email}"` |
| Bottone scollega | `"Scollega"` | `"Disconnect"` |
| Banner auth error | `"Connessione Google scaduta. Ricollega il tuo account."` | `"Google connection expired. Reconnect your account."` |
| Bottone ricollega | `"Ricollega"` | `"Reconnect"` |
| Label toggle Edit Area | `"Sync con Google Tasks"` | `"Sync with Google Tasks"` |
| Sub toggle attivo | `"I task vengono creati su Google Tasks"` | `"Tasks are created in Google Tasks"` |
| Titolo modale scollega | `"Scollega Google"` | `"Disconnect Google"` |
| Corpo modale scollega | `"La sincronizzazione verrà disabilitata su tutte le attività. I task futuri verranno rimossi da Google Tasks."` | `"Sync will be disabled for all activities. Future tasks will be removed from Google Tasks."` |
| CTA modale scollega | `"Scollega"` | `"Disconnect"` |
| CTA modale cancel | `"Annulla"` | `"Cancel"` |

---

## Dipendenze

- Epic 05 (Edit Area) — il toggle sync vive in questa schermata
- Epic 07 (Settings) — la sezione Google Tasks è aggiunta alla schermata Settings
- Epic 03 (Check-in) — ogni check-in triggera il push verso Google Tasks

---

## Note tecniche (per Lovable)

- Il **refresh token OAuth** va conservato in modo sicuro (Supabase Vault o colonna encrypted). Non in plaintext.
- Il **pull** (Google → opad) avviene all'apertura dell'app tramite chiamata frontend diretta alle Google Tasks API, non tramite cron job — semplificazione MVP.
- La **distribuzione dei giorni** per `frequency_per_week` è predefinita da opad (vedi tabella sopra) — non richiede input utente.
- Lo scope OAuth richiesto è `https://www.googleapis.com/auth/tasks`.
- La TaskList Google si chiama `"opad.me"` e viene creata al primo utilizzo.
- Il marker `[opad:area_id:date]` nel campo `notes` del task Google serve a riconoscere i task creati dall'app ed evitare loop.
