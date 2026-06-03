# immobile-demo-web — Contesto Progetto

Sito demo scrolling per il cliente **Immobilads** (complesso Lido di Borgio).
Repo GitHub Pages: `github.com/Spinna94/scrolling-primo` (branch `main`, root `/`)

---

## File principali

| File | Descrizione |
|------|-------------|
| `index.html` | Versione "Vivace" — animazioni IntersectionObserver, no dipendenze esterne |
| `luxury.html` | Versione cinematica stile springs.house — GSAP + ScrollTrigger via CDN |
| `immobilads.html` | Versione scroll-video — canvas 139 frame JPG, overlay interattivo piani |
| `piantina.html` | Piantina SVG interattiva Piano 3° — 4 appartamenti cliccabili |
| `assets/frames/` | 139 frame JPG estratti da `/home/Spinna/Video/Immobilads.mp4` |

---

## Stato lavori (aggiornato 2026-06-03)

- ✅ `index.html` — completo (placeholder contenuto)
- ✅ `luxury.html` — completo (placeholder contenuto)
- ✅ `immobilads.html` — completo incl. overlay piani: 8 piani + rooftop, separatori a 3 punti prospettici, label equidistanti
- ✅ `piantina.html` — completo: Piano 3°, appartamenti A/B/C/D cliccabili, pannello info
- ✅ `floor-detect.html` — tool interno: calibrazione overlay piani (3 punti per boundary, 10 boundary totali)
- ⏳ Siti reali Lido di Borgio e Via Nizza 277 — in attesa media dal cliente

---

## immobilads.html — dettaglio tecnico

### Frame extraction
```bash
ffmpeg -i /home/Spinna/Video/Immobilads.mp4 -vf "fps=8,scale=1280:-1" -q:v 3 assets/frames/frame_%04d.jpg
```
Produce 139 frame JPG (1280×720). Usare sempre `.jpg`, non `.webp` (webp produce file singolo animato).

### Canvas cover-fit
```javascript
const scale = Math.max(canvas.width / img.naturalWidth, canvas.height / img.naturalHeight);
const x = (canvas.width  - img.naturalWidth  * scale) / 2;
const y = (canvas.height - img.naturalHeight * scale) / 2;
ctx.drawImage(img, x, y, img.naturalWidth * scale, img.naturalHeight * scale);
```

### Frame freeze (overlay piani)
```javascript
const FREEZE_FRAME = 59;   // frame_0060.jpg — edificio completamente visibile
const FREEZE_START = 0.35; // 35% scroll → inizia freeze
const FREEZE_END   = 0.52; // 52% scroll → riprende animazione
```

### Overlay piani (floors overlay) — coordinate frame_0060.jpg
- Separatori: `<polyline class="f-sep" fill="none">` con 3 punti (sx / centro / dx) — prospettiva reale
- Zone hit: `<polygon class="floor-hit">` con 6 punti (top-sx, top-mid, top-dx, bot-dx, bot-mid, bot-sx)
- **IMPORTANTE**: aggiungere `fill: none` al CSS `.f-sep` — polyline defaulta a fill:black
- Piano 3° → `piantina.html` (classe `has-plan`), gli altri → toast "in elaborazione"
- Calibrato con `floor-detect.html` (10 boundary × 3 punti = 30 click)
- Separatori (sx / centro / dx):
  - Roof/8:  500,147 — 830,66  — 1010,140
  - 8/7:     501,208 — 828,141 — 1010,207
  - 7/6:     500,262 — 829,206 — 1009,257
  - 6/5:     487,327 — 828,289 — 1008,325
  - 5/4:     488,393 — 827,372 — 1009,393
  - 4/3:     489,457 — 828,456 — 1008,462
  - 3/2:     489,526 — 828,536 — 1007,526
  - 2/1:     489,592 — 827,621 — 1008,596
- Label sinistra: equidistanti y=80..640 step 70px (Rooftop=80, Piano 1°=640)
- Per ricalibrazione: aprire `floor-detect.html`, cliccare 3 punti per boundary (10 totali), copiare SVG output

### Overlay testi (4 card durante lo scroll)
```javascript
{ id: 'ov1', start: 0.00, fadeIn: 0.03, peak: 0.12, fadeOut: 0.22 }, // presentazione
{ id: 'ov2', start: 0.22, fadeIn: 0.25, peak: 0.30, fadeOut: 0.34 }, // palazzo svelato
{ id: 'ov3', start: 0.54, fadeIn: 0.57, peak: 0.65, fadeOut: 0.77 }, // rooftop (dopo freeze)
{ id: 'ov4', start: 0.79, fadeIn: 0.83, peak: 0.91, fadeOut: 1.00 }, // CTA finale
```

---

## piantina.html — coordinate SVG (viewBox 0 0 900 580)

| App | x | y | w | h | Stato |
|-----|---|---|---|---|-------|
| A (NW) | 50 | 80 | 320 | 210 | Disponibile — 95m², 3cam, €420k |
| B (NE) | 530 | 80 | 320 | 210 | Venduto |
| C (SE) | 530 | 290 | 320 | 210 | Riservato — 80m², 2cam, €360k |
| D (SW) | 50 | 290 | 320 | 210 | Disponibile — 110m², 3cam, €490k |
| Core | 370 | 80 | 160 | 420 | — |

Logge: hatch pattern, sporgono a nord (y=40–80) e sud (y=500–540).

---

## Deploy

```bash
# Push con deploy key (la chiave è in /tmp — si perde al reboot, va rimessa)
GIT_SSH_COMMAND='ssh -i /tmp/deploy_key_scrolling -o StrictHostKeyChecking=no' git push origin main
```

Se `/tmp/deploy_key_scrolling` non esiste (si perde al reboot): generare nuova chiave, aggiungere la pubblica come deploy key su GitHub repo scrolling-primo con write access:
```bash
ssh-keygen -t ed25519 -f /tmp/deploy_key_scrolling -N ""
cat /tmp/deploy_key_scrolling.pub
# → GitHub → repo → Settings → Deploy keys → Add → incolla pubkey → Allow write access
```

---

## Prossimi step

1. Ricevere media dal cliente (video drone/render) per Lido di Borgio e Via Nizza 277
2. Estrarre frame con ffmpeg, caricare in `assets/frames/` (o cartella dedicata per progetto)
3. Adattare `immobilads.html` con contenuto reale del complesso
4. Replicare struttura per Via Nizza 277 come sito separato

Note Obsidian correlate: `Lido di Borgio — Residenze.md`, `Via Nizza 277 — Residenze.md`
