# Image Downsampling & Vision Budgeting

**Addresses:** Cause 4.1 in [`../CAUSE.md`](../CAUSE.md)

**Idea:** Size every image to the smallest resolution the task actually
needs before it enters the request — vision token cost scales with pixel
area/tiles on every major provider, so resolution is a direct cost dial.

---

## The cost model (know your provider's formula)

| Provider | Cost model (approximate) | Practical lever |
| --- | --- | --- |
| Anthropic | ~`(width × height) / 750` tokens; high-res models accept up to 2576px long edge (up to ~4,784 tokens/image) | Downscale long edge; 1568px matches the older cap (~1,600 tokens) |
| OpenAI | Tile-based: base cost + per-512px-tile cost at `detail: high`; flat ~85 tokens at `detail: low` | `detail: low` for classification/presence checks; resize to reduce tile count |
| Gemini | Tiled: ~258 tokens per 768×768 tile | Fit within fewer tiles |

Two consequences:

1. A screenshot sent at 4K vs 1080p can differ by **3–5×** in tokens for the
   same *legible* content.
2. Images persist in history (cause 2.1) — an oversized screenshot is
   re-billed on every subsequent turn until pruned.

## How to apply

1. **Downscale at the harness boundary, always.** Route every image through
   a resize step before request assembly:
   - General screenshots / computer-use loops: **1080p** is the
     performance/cost sweet spot on current models; **720p / 1366×768** for
     cost-sensitive loops.
   - Text-heavy documents: keep enough DPI for OCR-legibility (~1,500px long
     edge is usually sufficient); test on your worst fonts.
   - Classification / "is there a dialog on screen": provider low-detail
     modes (OpenAI `detail: low` ≈ 85 tokens) or aggressive downscale.
2. **Crop before you send.** If the region of interest is known (an element
   bounding box, a diff region), crop to it — a 300×200 crop is ~80 tokens
   vs thousands for the full screen. Agentic vision models can also be given
   a crop/zoom tool and a small full-frame overview instead of one huge
   image.
3. **Prune stale screenshots from history.** In computer-use loops, replace
   screenshots older than 2–3 steps with a stub ("[screenshot step 12 —
   superseded]") — the current screen state is what matters. This is the
   vision case of `context-editing.md`.
4. **Don't re-send unchanged images.** Hash frames; if the screen didn't
   change, say so in text instead of attaching an identical screenshot
   (`context-hygiene.md`).
5. **Prefer text when text exists.** DOM/accessibility-tree extraction for
   browser agents, PDF text layers for documents — structured text is
   usually 5–20× cheaper than pixels of the same content, and more reliably
   parsed. Use vision when layout/appearance is the point.

## SOTA tools

### Native — coding agents & provider APIs

| Provider / agent | Feature | Notes |
| --- | --- | --- |
| OpenAI API | `detail: low\|high\|auto` per-image dial | `low` is a flat ~85 tokens — the cheapest presence-check mode anywhere |
| All providers | `count_tokens` on representative images | Re-baseline after model upgrades — image token formulas change between generations |
| Anthropic / OpenAI computer-use tools | Documented resolution guidance | Follow the vendor's recommended capture resolution rather than native screen size |

### Third-party — agent-agnostic (open source preferred)

| Tool | License | Notes |
| --- | --- | --- |
| sharp (Node) / Pillow-SIMD (Python) / libvips | Apache-2.0 / PIL / LGPL | Fast resize/crop at the harness boundary, before any provider sees the image |
| Playwright accessibility tree / `browser-use` DOM extraction | Apache-2.0 / MIT | Text-first page representation for browser agents; screenshots only as fallback |
| pixelmatch / perceptual hashing | ISC / MIT | Skip unchanged frames in screenshot loops |

## Trade-offs

- Over-downscaling breaks OCR and small-UI-element grounding; coordinate
  precision for computer-use clicks degrades below ~720p. Calibrate on your
  actual UI.
- A resize step adds a few ms per image and a processing dependency.
- Crop-based flows need correct region tracking; a wrong crop forces a
  re-shot (extra round trip).

## Expected impact

- Downscaling 4K/native screenshots to 1080p typically cuts vision input
  **3–5×** with negligible task-accuracy loss on standard UIs.
- In screenshot-per-step computer-use loops, combining 1080p capture +
  stale-screenshot pruning routinely reduces total session input by
  **50–80%**, since screenshots dominate those transcripts.
- Switching browser agents from screenshot-primary to DOM-primary
  representations cuts per-step cost by **an order of magnitude** on
  text-dense pages.
