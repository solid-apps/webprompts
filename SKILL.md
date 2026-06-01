---
name: webprompts
description: How to author and render a webprompts canvas — a JSON-LD node graph (character sheet → storyboard → clip → stitch → score → export) for AI prompt→image→video pipelines on a Solid pod. Use when generating or editing a webprompts document, planning an AI video pipeline, or producing a prompt graph the webprompts app can render.
---

# webprompts

**webprompts** is a build-free [solid-app](https://github.com/solid-apps) that renders
a **node canvas** for an AI **prompt → image → clip → stitched video** pipeline. The
canvas is one **JSON-LD document** (on a pod, a `?src=` URL, or an inline
`<script type="application/ld+json">` data island). "Fork" = copy the document to your
own pod. App: https://solid-apps.github.io/webprompts/

Your job as an LLM is usually one of:
1. **Author** a webprompts JSON-LD doc from a story/brief (the common case).
2. **Edit/extend** an existing doc (add a shot, a stitch, a score…).

Get the **document shape** right (so it renders) AND the **prompt craft** right (so the
generated media is good and passes content filters). Both are below.

## Document shape

```json
{
  "@context": { "schema": "https://schema.org/", "wp": "https://webprompts.org/ns#" },
  "@id": "#this",
  "@type": "wp:Canvas",
  "schema:name": "Project title",
  "wp:bible": "Global style + canon + character descriptions, reused VERBATIM downstream.",
  "wp:node": [ /* nodes, see below */ ]
}
```

The reader is tolerant: `wp:`/`schema:` prefixed keys, full IRIs, or bare keys all work;
nodes may live under `wp:node` or a top-level `@graph`.

### Node

```json
{
  "@id": "#shot1",
  "@type": "wp:Clip",
  "schema:name": "Shot 1 — the crest",
  "wp:x": 380, "wp:y": 300,              // canvas position (px); lay the pipeline left→right
  "wp:prompt": "…",                       // the generation prompt (Clip/Storyboard/Score)
  "wp:description": "…",                   // identity text (CharacterSheet)
  "wp:model": "Seedance",                 // which model/tool for this node
  "wp:duration": 15,                       // seconds (Clip)
  "wp:transition": "match-frame",          // Stitch: match-frame | crossfade | whip | dissolve
  "wp:format": "1920×1080 · yuv420p · +faststart",  // Export
  "wp:status": "success",                 // success | pending | error (status dot)
  "schema:image": "https://…",            // image asset URL (thumbnail)
  "schema:video": "https://…",            // video asset URL (plays on click)
  "wp:derivedFrom": [ { "@id": "#hero" }, { "@id": "#board" } ]   // LINEAGE — draws the edges
}
```

**`wp:derivedFrom`** is what wires the graph — list every node this one was made from.
Edges render left→right, so position upstream nodes to the left (smaller `wp:x`).

### Node types (the pipeline)

`CharacterSheet → Storyboard → Clip(s) → Stitch → Score → Export` (also `Image`,
`Prompt`, `Video`). Standard shape:

- **CharacterSheet** — one hero; a locked `wp:description` reused **verbatim** in every
  downstream node (paraphrasing causes identity drift). Optional `schema:image` sheet.
- **Storyboard** — beats of the sequence (`wp:prompt`, ≤2000 chars).
- **Clip** — one ~15s shot: `wp:prompt` (≤1800 chars), `wp:model`, `wp:duration`,
  `wp:derivedFrom` the character sheet + prior clip. **A 30s piece = two stitched 15s clips.**
- **Stitch** — joins clips; `wp:transition`.
- **Score** — music/ambient bed (`wp:prompt`); mix notes in the prompt.
- **Export** — final render settings (`wp:format`).

## Authoring guidance (distilled from real production)

### Pipeline
- **Plan in 15s units** (most video models cap ~15s). 30s = two stitched clips.
- **One hero per shot** generates cleanest. Reuse the character-sheet text verbatim.
- **Match-frame handoff:** clip N's last beat == clip N+1's first beat, described
  identically — that shared frame makes the stitch seamless. (Time-jump → use a dissolve.)
- State **"one unbroken continuous take"** or a multi-beat clip will hard-cut.
- **Screen-direction lock** (180° rule + a fixed landmark on one side), restated in
  every clip, stops subjects flipping sides.
- **Pace the payoff:** front-load the build; give the climax the final 5–6s. Say so.

### Prompt craft
- **Lean, evocative beats**, screenwriter-direct — not novelistic.
- **Lead with the subject grammatically** and repeat it in every beat; remove competing
  cues (bury "a dragon" under siege detail and you get no dragon).
- **Stage geometry explicitly** — the model won't infer spatial tension.
- Char limits: clip ≤1800, storyboard ≤2000. Move global style to `wp:bible`; keep
  per-shot beats tight.

### Content filters (the #1 failure cause — author defensively)
- **No named IP / characters** (film *titles* pass, character *names* don't).
- **No real-person likeness** terms ("A-list star", "supermodel"); photoreal beautiful
  (esp. blonde) faces can likeness-flag. Fix: characterful specifics (freckles, a scar,
  an unusual hair shade), or go **stylized**, or **text-only** (drop the reference image).
- **No proper names that partial-match IP** ("Mara" → "Mara Jade"); prefer descriptors.
- **No in-frame text** — trips the text filter and mangles spelling. Add titles in post.
- **Wholesome wording for peril** ("scoops up / holds safe / caught in ropes" — NOT
  "traps / jaws / decapitate"). Creature combat reads safer than human violence.
- **Sensual-but-safe = perfume-ad register** (gaze, silk, light; elegant *covered*
  wardrobe). "sexy / lingerie / exposed" → refused.

### Play to strengths
- **Great at:** a single near-still hero, atmosphere, light, scale, slow camera moves,
  particle/glow **reveals**, establishing shots, expressive faces.
- **Weak at:** complex choreography, morphs, fast multi-hit action, lip-sync, dense
  multi-character scenes, in-frame text.
- Make the money shot a **reveal** (a glow, a bloom, a tail unfurl); let the **camera**
  carry dynamism while the subject stays simple. Use **action-tuned models** (Hailuo/
  MiniMax) for kinetic shots, baseline (Seedance) for atmosphere — set `wp:model` per node.

### Audio / export (for Score / Export nodes)
- Clips default to **no music/dialogue** — specify an ambient bed; score separately.
- Mix note: clip audio is quiet (~−32 dB); duck the bed (~0.3×), boost clip (~+12 dB),
  `amix=…:normalize=0`, `loudnorm I=−16`.
- Export: **1920×1080, `yuv420p`** (never yuv444p — X rejects it), SAR 1:1, AAC,
  `+faststart`. Title/end cards in post.

## Minimal example

```json
{
  "@context": { "schema": "https://schema.org/", "wp": "https://webprompts.org/ns#" },
  "@id": "#this", "@type": "wp:Canvas", "schema:name": "First Light",
  "wp:bible": "Style: warm dawn, soft haze, slow push-ins. Canon: a lone lighthouse keeper.",
  "wp:node": [
    { "@id": "#hero", "@type": "wp:CharacterSheet", "schema:name": "Keeper",
      "wp:description": "An older keeper in an oilskin coat, white stubble, a brass key on a cord.",
      "wp:x": 60, "wp:y": 160 },
    { "@id": "#c1", "@type": "wp:Clip", "schema:name": "Shot 1 — the climb",
      "wp:prompt": "The Keeper climbs the spiral stair at dawn, warm haze, slow push-in. One unbroken continuous take. Window kept on frame-right throughout.",
      "wp:model": "Seedance", "wp:duration": 15, "wp:x": 360, "wp:y": 160,
      "wp:derivedFrom": [ { "@id": "#hero" } ] },
    { "@id": "#c2", "@type": "wp:Clip", "schema:name": "Shot 2 — first light",
      "wp:prompt": "Match-frame from Shot 1's last beat: same stair top, window on frame-right. The lamp blooms to life; warm light floods the lens. Climax in the final 5 seconds.",
      "wp:model": "Hailuo", "wp:duration": 15, "wp:x": 660, "wp:y": 160,
      "wp:derivedFrom": [ { "@id": "#c1" } ] },
    { "@id": "#stitch", "@type": "wp:Stitch", "schema:name": "Stitch 30s",
      "wp:transition": "match-frame", "wp:x": 960, "wp:y": 160,
      "wp:derivedFrom": [ { "@id": "#c1" }, { "@id": "#c2" } ] }
  ]
}
```

## Storage & conventions

- Saved on a pod at `/public/webprompts/<name>.jsonld` (content type `application/ld+json`).
- The app loads it via xlogin's authenticated fetch (works signed-out for public docs).
- Lay the graph **left→right** along the pipeline; give every non-root node a
  `wp:derivedFrom` so the lineage edges render.
- Status: set `wp:status` to `success` for assets that exist, `pending` for to-be-generated.

(Generation itself is a later app phase — a node's prompt can be sent to a model endpoint
or a NIP-90 nostr DVM, with the output written back as a new node.)
