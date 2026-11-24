---
name: spectralbreak-xrpl-mint
description: Companion skill — mint a SpectralBreak render as an NFT on the XRP
  Ledger, from chat. Use when the user has made a SpectralBreak video (see the
  spectralbreak skill) and says "mint this", "make it an NFT", "mint my
  SpectralBreak", or asks to put a render on-chain. Requires BOTH the
  spectralbreak skill and the xrpl-muse-skill; if the XRPL skill is missing,
  offer its easy install and stop. The companion adds NO commands to either
  skill — it composes them, and every signing step stays inside the XRPL
  skill's propose → approve → sign ceremony.
license: MIT
---

# SpectralBreak × XRPL — mint the womp

A SpectralBreak render is a living piece of art. This companion turns one into an
NFT on the XRP Ledger (XRPL) — a one-of-one token whose record anyone can verify,
carrying the exact recipe that made it.

It is a **bridge, not a wallet**. It never holds keys, never signs, never edits
policy. Rendering belongs to SpectralBreak; money and minting belong to the
[xrpl-muse-skill](https://github.com/terramike/xrpl-muse-skill) and its ceremony.
This skill only: prepares the artwork file, fills in the story, and walks the
user through the XRPL skill's own steps.

## Plain-English rule (muggle explainers)

**Whenever a technical term appears in anything the user reads, the very next
line explains it in plain English**, starting with "In plain English:". No
exceptions for "everyone knows that one." The glossary at the bottom of this
file is the approved wording — reuse it verbatim.

## Step 0 — Check both skills are installed

1. **SpectralBreak skill** — needed to make the render. If missing, point to
   https://github.com/terramike/spectralbreak-skill and stop.
2. **XRPL skill** — check for `xrpl-trade` on the PATH (or the skill's install
   directory). If absent, **do not fake or improvise a mint.** Say:

   > To mint, you'll need the free XRPL Muse skill installed — it's the part
   > that talks to the XRP Ledger safely. In plain English: SpectralBreak makes
   > the art; the XRPL skill is the cashier and the vault. Install it from
   > https://github.com/terramike/xrpl-muse-skill (one script: `install.sh`),
   > then say "mint my SpectralBreak" again and we'll pick up right here.

   Then stop. On the next run, the XRPL skill's own onboarding takes over
   (it starts read-only; going live is the user's deliberate ceremony there,
   never something this companion requests or rushes).

## Step 1 — Make the render (SpectralBreak skill)

Run a normal SpectralBreak render. Keep **two outputs**:

- the **MP4** — the watch/share copy (plays everywhere; this is the file you
  post and send to people);
- a **GIF** — the mint copy (see Step 2 for why, and the size rules).

Record from the render: the **settings hash**, pattern, modes, loop seconds,
and the **full-state reproducing link** (or the settings block, for uploaded art).

> In plain English: the settings hash is a short fingerprint of the exact
> recipe. Anyone with the link can re-make the identical look on SpectralBreak.

## Step 2 — Prepare the mint file (GIF, small)

Why GIF for the mint: the XRPL skill's artwork checker accepts **images only**
(PNG/JPEG/GIF/WebP, checked by the file's actual bytes, max **10 MB**) — video
files are refused today. A GIF *is* the moving piece, needs no player, and
displays on every phone and marketplace page, Apple or not. (MP4 minting is a
feature proposal for the XRPL skill, not something to work around.)

Make the GIF from the MP4 with the video tool (ffmpeg; if the user's skill
library registers a newer house video tool, use that instead):

- Two-pass palette GIF (best colour for hue-cycling art):
  `ffmpeg -i in.mp4 -vf "fps=12,scale=800:-1:flags=lanczos,palettegen" pal.png`
  then `ffmpeg -i in.mp4 -i pal.png -filter_complex "fps=12,scale=800:-1:flags=lanczos[x];[x][1:v]paletteuse" out.gif`
- **Size ladder** — target ≤ 8 MB (headroom under the 10 MB cap). If over:
  drop to 640 px wide → fps 10 → trim the loop. Re-weigh after each step and
  **tell the user the final size and what was traded for it**. Never stage a
  file over the cap and never silently shrink quality.
- Copy the final GIF into the XRPL skill's media folder (`~/.xrpl/media` by
  default) — the only place its checker will accept artwork from.

## Step 3 — Stage it (XRPL skill, local only)

```
xrpl-trade nft-stage --file <mint.gif> \
  --name "<Art title> — SpectralBreak <Portal/Hue/Overlay>" \
  --description "<one or two sentences, plain words>" \
  --royalty-bps 1000 \
  --meta settings_hash=<hash> --meta pattern=<pattern> \
  --meta modes=<modes> --meta reproduce=<full-state URL>
```

- Staging is **local only** — it checks the file and writes a review page;
  nothing touches the internet or the ledger yet.
- If the user has a registered **collection** (e.g. a "Spectral" collection),
  add `--collection <name>`; collections/templates are the XRPL skill's own
  registries — creating one is shown to the user as an exact change and needs
  their yes, per that skill's rules. Never create one silently.
- Royalty default 1000 = 10% (see glossary) — always state it in plain numbers
  and let the user change it **before** staging; it can never change after.

## Step 4 — Review, pin, propose (XRPL skill ceremony)

Show the user the stage record in plain language: the picture, the name, the
story, the recipe fingerprint, the royalty. Then, **only on their go**:

```
xrpl-trade nft-pin-and-propose --stage <stage-id>
```

> In plain English: this uploads ("pins") the art and its story card to IPFS —
> a public file shelf for the internet — using *your own* free Pinata account,
> and prepares the official mint request. Nothing is minted yet; the request
> still needs your signature in the next step.

The XRPL skill will refuse if its policy doesn't allow minting yet — that's
the user's own safety switch in that skill (its onboarding explains it).
**This companion never edits that policy, never asks for any key or seed, and
never signs.** If a step is refused, read the refusal to the user plainly and
stop.

## Step 5 — The user signs; verify on the ledger

The XRPL skill prints a proposal and a short code (its hash). **The user**
approves that exact code with `xrpl-sign --profile … --hash … --approve` —
in plain English: signing is the user personally stamping the request; the
agent cannot do it for them, by design.

After signing, verify and report:

- the **token ID** and a marketplace link (e.g. `https://xrp.cafe/nft/<id>`),
- that the token's record shows the staged name and the settings hash,
- one line: title, recipe fingerprint, royalty, network (testnet/mainnet).

First runs belong on **testnet** (practice money) until the user says otherwise.

## Rules

- **Compose, never extend.** No new commands in either skill; no policy edits;
  no seed handling — the XRPL skill's "never ask for a seed" rule binds here too.
- **Every gate stays a gate.** Stage → review → pin/propose → user signs. Any
  refusal from the XRPL skill is reported, not routed around.
- **Size honesty.** Report the mint file's real bytes and any quality trade made
  to fit the cap.
- **Provenance is the point.** The settings hash and reproducing link always go
  in the metadata — an NFT whose recipe can't be re-run is just a picture.
- **Plain English, every time** (see the rule at the top).

## Glossary (approved plain-English wording)

| Term | In plain English |
|---|---|
| NFT | A one-of-one digital certificate that says "this is the original, and here's who owns it." |
| Mint | Making that certificate on the ledger — like striking a coin, once. |
| XRPL / XRP Ledger | A public record book nobody owns, where the certificate lives forever. |
| IPFS / pinning | A public file shelf for the internet; pinning = putting the art on the shelf so it can't quietly vanish. |
| Pinata | The free service that keeps your art pinned on that shelf, under your own account. |
| Royalty (bps) | The artist's cut of every future resale. 1000 bps = 10%. Set at mint, never changeable after. |
| Taxon | A family number that groups your pieces into one collection. |
| Proposal / signing | The agent writes the request; only you can stamp it. No stamp, no mint. |
| Testnet | The practice ledger with play money — same moves, nothing real at stake. |
| Settings hash | A short fingerprint of the exact SpectralBreak recipe that made this piece. |

---

*Companion to the spectralbreak skill; minting by the xrpl-muse-skill's NFT flow
(nft-stage → nft-pin-and-propose → xrpl-sign). Verified against xrpl-muse-skill
behaviour on 2026-11-24: image-only artwork validation (PNG/JPEG/GIF/WebP),
10 MB cap, media-folder rule, attribution and policy gates as documented there.
MP4 minting awaits video support in the XRPL skill — proposed, not assumed.*
