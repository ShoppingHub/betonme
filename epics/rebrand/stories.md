# Stories — Rebrand: BetonMe → opad.me

## Sequenza di implementazione

```
story-rebrand-01 → Rename app + aggiorna wordmark con nuovi colori logo   ⏳ da fare
```

---

## story-rebrand-01 — Rename app e wordmark opad.me

L'app cambia nome da **BetonMe** a **opad.me**. Aggiorna tutto il codebase con il nuovo nome e il nuovo stile del wordmark.

---

### 1. Rename globale

Sostituisci tutte le occorrenze di `"BetonMe"` con `"opad.me"` in:
- `<title>` del documento HTML
- Meta tags (`og:title`, `og:site_name`, ecc.)
- Header della Dashboard e di ogni pagina che mostra il nome app
- Qualsiasi stringa hardcoded nel codice

---

### 2. Wordmark nella sidebar desktop e nell'header mobile

Il wordmark `"opad.me"` è composto da due parti cromatiche distinte:

```
pad  .me
───  ───
 ↑    ↑
 │    └── colore #B5453A (terracotta)
 └─────── colore #FFFFFF (bianco)
```

**Implementazione:**
- Renderizza il wordmark come due `<span>` inline:
  - `<span style="color: #FFFFFF">pad</span><span style="color: #B5453A">.me</span>`
- Font: Inter, font-weight 600
- Non usare `#B5453A` in nessun altro elemento dell'UI — è esclusivo del wordmark

**Posizioni dove appare il wordmark:**
- Sidebar desktop: in cima, dove ora c'è `"BetonMe"` (testo)
- Header mobile della Home: a sinistra, dove ora c'è `"BetonMe"` (testo)

---

### 3. Icona / logo (opzionale in questa story)

Il logo completo ha un'icona a sinistra del wordmark (figura Uomo Vitruviano in cerchio con ingranaggio alla base). Se l'asset SVG dell'icona è già disponibile nel progetto, affiancalo al wordmark nella sidebar e nell'header. Se non è disponibile, usare solo il wordmark testuale `"pad.me"` per ora.

---

### 4. Favicon e app icon

Aggiorna il `favicon.ico` e l'`apple-touch-icon` con il nuovo nome/identità se gli asset sono disponibili. Se non lo sono, lascia invariato per ora — non bloccare questa story per la mancanza di asset grafici.

---

### Note

- Il colore `#B5453A` non è nel sistema dei colori dell'app (non si usa su pulsanti, grafici, stati UI) — è **solo per il wordmark**
- Il tono visivo dell'app rimane invariato — stessi background, stessi grafici, stessa tipografia
