# webprompts

A **canvas for AI prompts → media → video**, on your [Solid](https://solidproject.org)
pod. One self-contained HTML file, no build, signs in with **xlogin**.

You give it a JSON-LD document — inline as a `<script type="application/ld+json">`
data island, or a pod file via `?src=<url>` — and it renders it as a **node canvas**:
prompts, image/video assets, and the edges between them (what was made from what).
The whole thing is *your data on your pod*, so **"fork" = copy the doc to your pod** —
a genuinely decentralized remix, no SaaS, no credits wall.

Inspired by the prompt-canvas pattern (promptsref, ComfyUI, Krea, Flora). The
video-assembly side aligns with **[OpenTimeline Lite](https://github.com/inartes/otio)**
(Timeline → Tracks → Clips) so a webprompts doc can describe a full sequence, not
just a moodboard. Always intended for **prompt sharing** (webprompts.org).

## Phases (each ships something useful)

1. **Canvas reader** — load a webprompts JSON-LD (data island or `?src=`), render
   prompt / image / video nodes + lineage edges on a pan-zoom canvas. Read-only.
2. **Edit + save to pod** — add / move / connect / edit nodes; save to
   `/public/webprompts/<name>.jsonld`; fork = copy to your pod.
3. **Timeline (OTIO-lite)** — arrange clips into tracks, preview in order,
   import/export OTIO-lite — assemble a video sequence.
4. **Generation** — per-node *Generate* via a BYO model endpoint or a **NIP-90
   nostr DVM**; output saved to your pod, becomes a new node/clip.
5. **Prompt library / sharing** — browse, tag, and fork prompts.

## Data model

One JSON-LD doc combining **schema.org** (media: ImageObject / VideoObject) + a
small **`wp:`** vocabulary (prompt text, model + params, canvas `x`/`y`,
`derivedFrom` edges) + **OTIO-lite** for the timeline. A prompt canvas and a video
timeline are two views of the same document.

## Run

Static — open `index.html`, or install via the **store** to `/public/apps/webprompts/`.

AGPL-3.0-only.
