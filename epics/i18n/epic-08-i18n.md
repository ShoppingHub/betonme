# Epic 08 — Lingua (IT / EN)

## Obiettivo
Permettere all'utente di scegliere la lingua dell'interfaccia tra italiano e inglese, con persistenza della preferenza e applicazione immediata a tutti i testi dell'app.

---

## Behavior

La lingua è una preferenza utente, salvata su Supabase nella tabella `users`. Al primo accesso viene rilevata automaticamente dalla lingua del browser; l'utente può cambiarla in qualsiasi momento dalle Impostazioni.

Tutti i testi dell'app (label, placeholder, messaggi di errore, copy modale, empty state) seguono la lingua selezionata. Non esistono testi misti.

---

## Lingue supportate

| Codice | Nome | Default |
|---|---|---|
| `it` | Italiano | Se browser `it-*` |
| `en` | English | Tutti gli altri browser |

---

## Flussi

### Primo accesso
1. L'app rileva la lingua del browser
2. Imposta `language = "it"` o `"en"` di conseguenza
3. Salva la preferenza nella tabella `users` al termine dell'onboarding

### Cambio lingua da Settings
1. L'utente apre Impostazioni / Settings
2. Vede il selettore lingua: `Italiano · English`
3. Tappa l'opzione desiderata
4. La UI si aggiorna immediatamente — nessun reload di pagina
5. La preferenza `language` viene salvata su Supabase (silenzioso)

---

## Selettore lingua

- Tipo: segmented control a 2 opzioni (`Italiano · English`)
- Posizione: sezione "Preferenze" in Settings, prima degli altri toggle
- Stile: stesso componente `<TimeRangeSelector>` con layout orizzontale

---

## Copy chiave (IT / EN)

| Elemento | IT | EN |
|---|---|---|
| Label selettore | `Lingua` | `Language` |
| Opzione 1 | `Italiano` | `Italiano` |
| Opzione 2 | `English` | `English` |
| Header Settings | `Impostazioni` | `Settings` |
| Toggle score | `Mostra punteggio` | `Show trajectory score` |
| Toggle notifiche | `Notifiche` | `Notifications` |
| Label sezione account | `Account` | `Account` |
| Sign out | `Esci` | `Sign out` |
| Delete account | `Elimina account` | `Delete account` |
| Check-in button | `Registra oggi` | `Log today` |
| Check-in completato | `Osservato` | `Observed` |
| Home — empty state titolo | `Cosa vuoi osservare?` | `What do you want to observe?` |
| Home — empty state sub | `Aggiungi un'area di vita per iniziare.` | `Add a life area to start seeing your trajectory.` |
| Add area CTA | `Aggiungi la tua prima area` | `Add your first area` |
| Edit area | `Modifica area` | `Edit area` |

> La lista completa del copy va definita come file separato `epics/i18n/copy.md` con tutte le stringhe nelle due lingue.

---

## Database

Aggiungere colonna alla tabella `users`:
```
language  TEXT  DEFAULT 'en'  CHECK (language IN ('it', 'en'))
```

---

## Stati UI

| Stato | Comportamento |
|---|---|
| Lingua rilevata dal browser | Applicata senza intervento utente |
| Cambio lingua | UI aggiornata istantaneamente, salvataggio silenzioso |
| Utente non autenticato (login/onboarding) | Lingua da browser, non salvata fino al login |

---

## Edge Case

- Utente con browser in lingua non supportata (es. francese) → default `en`
- Utente cambia lingua in onboarding → preferenza applicata subito, salvata al termine
- Stringa non tradotta → fallback a `en` silenzioso (non mostrare chiave raw)

---

## Acceptance Criteria

- [ ] Il selettore lingua appare nelle Impostazioni come primo elemento della sezione Preferenze
- [ ] Al cambio lingua la UI si aggiorna senza reload
- [ ] La preferenza `language` viene salvata su Supabase
- [ ] Al riavvio dell'app la lingua salvata è applicata correttamente
- [ ] Il browser in `it-*` usa IT come default; tutti gli altri usano EN
- [ ] Nessun testo rimane nella lingua precedente dopo il cambio

---

## Stories

- `story-08-01` — Selettore lingua in Settings + salvataggio su Supabase
- `story-08-02` — Rilevamento lingua dal browser al primo accesso
- `story-08-03` — Traduzione completa IT/EN di tutti i testi dell'app
