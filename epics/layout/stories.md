# Stories — Epic 09 — Layout Responsivo

## Sequenza di implementazione

```
story-09-01 → Sidebar desktop (≥ 1024px) con logo e voci fisse
story-09-02 → Adattamento responsivo: switch automatico mobile/desktop
story-09-03 → Voci custom nella sidebar da configurazione Settings
```

> **Dipendenze:** Richiede Epic 07 (Settings — sezione Menu) per le voci custom. La sidebar riflette la stessa configurazione del bottom nav.

---

## story-09-01 — Sidebar desktop

BetonMe è un'app di osservazione del benessere. Aggiungi una sidebar di navigazione laterale che appare su viewport ≥ 1024px.

**Struttura sidebar (dall'alto verso il basso):**
- Logo / wordmark `"BetonMe"` in cima (padding top 24px)
- Voci di navigazione fisse: `Home` · `Aree / Areas`
- [Slot per voci custom — story-09-03]
- Divider orizzontale sottile
- `Impostazioni / Settings` in fondo

**Specifiche:**
- Larghezza fissa 240px
- Background `#0F2F33`
- Ogni voce: icona Lucide outline 20px + label testuale, altezza minima 44px, padding laterale 16px
- Voce attiva: testo e icona `#7DA3A0`, background `rgba(125,163,160,0.1)` rounded
- Voce inattiva: testo e icona `#B9C0C1`
- Il contenuto principale ha `margin-left: 240px`, padding 32px, max-width 900px centrato

**Label (seguono lingua utente):**
- Home: `Home`
- Areas: `Aree` (IT) / `Areas` (EN)
- Settings: `Impostazioni` (IT) / `Settings` (EN)

---

## story-09-02 — Layout responsivo mobile/desktop

Continua Epic 09 di BetonMe. Implementa lo switch automatico tra bottom nav (mobile) e sidebar (desktop).

**Behavior:**
- Viewport < 1024px → mostra solo il bottom nav fisso in basso (comportamento attuale), sidebar nascosta
- Viewport ≥ 1024px → mostra solo la sidebar laterale sinistra, bottom nav nascosto
- Il cambio avviene dinamicamente al ridimensionamento della finestra, senza reload
- Su desktop la sidebar non è collassabile (MVP — nessun toggle hamburger)

**Icone bottom nav (mobile):** solo icone (nessuna label)
**Sidebar desktop:** icone + label testuali

---

## story-09-03 — Voci custom nella sidebar

Continua Epic 09 di BetonMe. Propaga le voci custom configurate in Settings alla sidebar desktop e al bottom nav mobile.

**Behavior:**
- Leggi `menu_custom_items` dalla tabella `users` per l'utente corrente
- Inserisci le voci attivate tra `Aree / Areas` e il divider di `Impostazioni / Settings`
- Se nessuna voce custom → sidebar mostra solo: Home · Aree · [divider] · Impostazioni
- Se 1 voce custom (es. Finance) → Home · Aree · Finanze · [divider] · Impostazioni
- Se 2 voci custom → Home · Aree · Finanze · Palestra · [divider] · Impostazioni
- Lo stesso ordine si applica al bottom nav mobile

**Routing voci custom:**
- `Finanze / Finance` → naviga a `/finance`
- `Palestra / Gym` → naviga all'Area Detail dell'area gym dell'utente (`/areas/:gymAreaId`)

**Aggiornamento in tempo reale:**
- Al cambio in Settings, il menu si aggiorna senza reload (ottimistico)
