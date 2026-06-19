# Menuva AR — marker-based dish viewer (sandbox)

A polishing sandbox for the AR approach before transferring it into Menuva.
Single file: [`index.html`](index.html) + the model [`burger.glb`](burger.glb).

## Why marker-based (AR.js) instead of `model-viewer`

`model-viewer`'s "view on your table" AR is **markerless** — it relies on the
phone's SLAM plane tracking, which drifts and rescales (the floating / size
problems). Here the model is **locked to a physical marker**: as long as the
marker is in frame, position and size are nailed to it. The tradeoff is that
the marker must stay visible — move it out of frame and the dish disappears.

Stack: **A-Frame 1.6.0 + AR.js 3.4.7** (pinned, marker = Hiro preset).

## The marker

Print or show this on a second screen, then point the camera at it:
https://raw.githack.com/AR-js-org/AR.js/3.4.7/data/images/hiro.png

> Hiro is a single demo marker — it can't tell dishes apart. For a real
> multi-dish menu, switch to **barcode/matrix markers** (one code per dish) or
> **NFT image tracking** (track a branded coaster/logo). See "Next steps".

## Running it

### Desktop (quick validation — webcam works over `http://localhost`)
```powershell
python -m http.server 8000
# open http://localhost:8000  and allow the camera, then aim it at the Hiro marker
```
(Or via Claude Code's preview: the `ar-static` config in `.claude/launch.json`.)

### Phone (needs HTTPS — `getUserMedia` is blocked on plain http / file://)
Pick one:
- **ngrok:** `python -m http.server 8000` then `ngrok http 8000` → open the `https://…` URL on the phone.
- **GitHub Pages / any static https host:** push these two files and open the https URL.

> ⚠️ Opening `index.html` directly (`file://`) or over plain `http` on a phone
> gives a blank/black camera — this is the #1 "it doesn't work" cause.

## Model transform — how the numbers were derived

`burger.glb` is glTF-standard **Y-up**, real size **0.31 × 0.10 × 0.22 m**
(bounding box: minY = 0.0191, centre XZ = −0.0048, −0.0283).

The dish entity sits inside a spinner pivot so it rotates **in place**:
```
scale       = 4                              # ~1.25× marker width — clearly visible
position.y  = -(scale × bbox.minY)  = -0.076 # bottom rests ON the marker
position.xz = -(scale × bbox.centreXZ) = 0.019 / 0.113  # centred on the spin axis
rotation    = 0 0 0                          # NO -90° tilt (model is already upright)
```
To swap in a different model: read its bounding box (e.g. the snippet used in
this repo's commit, or any glTF inspector) and recompute those three lines.
**Don't** copy `rotation="-90 0 0"` from old AR.js examples — that assumes a
Z-up model and tips a standard Y-up glTF onto its back.

## Transferring into Menuva

- This is plain A-Frame web components, so it drops into any page. In
  Next.js/React, render it in a route that loads the two `<script>` tags (or
  via `next/script`), and gate it to the screen that needs AR.
- Keep the libraries **pinned** (`@3.4.7`, A-Frame `1.6.0`) — they must match.
- Serve over **HTTPS** in production (Menuva already will be).

## Next steps (decided later)

- **Barcode per dish** — `detectionMode: mono_and_matrix; matrixCodeType: 3x3`
  (up to 64 codes), one `<a-marker type="barcode" value="N">` per dish.
- **NFT image tracking** — most premium look, but the least stable AR.js mode;
  needs pre-generated descriptor files per image.
