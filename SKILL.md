---
name: spectralbreak
description: Create a SpectralBreak portal / hue-cycle video from a chat command.
  Use when the user says "SpectralBreak this", "portal video", "make it womp",
  or asks to run an image through the Portal Frame simulator at
  spectralbreak.consumerbreak.com. Builds a settings block (per the site's
  /llms.txt), opens the full-state URL or uploads the art, waits for the
  render, downloads the file, verifies it, and delivers it with the
  reproducing link.
license: MIT
---

# SpectralBreak — drive the Portal Frame simulator from chat

[SpectralBreak](https://spectralbreak.consumerbreak.com/) (by ConsumerBreak, for the
[Spectral Art Foundation](https://spectralartfoundation.org/)) turns an artwork into a looping
Portal Frame lamp preview and exports it as MP4, WebM or GIF. It runs entirely in the
visitor's browser — images are never uploaded to a server, no account is needed.

The site is agent-drivable by design (build S309+). **The canonical reference for every
field, range and default is the site's own https://spectralbreak.consumerbreak.com/llms.txt —
read it before your first run and whenever a field name is in doubt.** This skill is the
recipe layer on top: natural words in, a verified video out.

## Inputs

- **Art, two ways:**
  - A URL on an **allowed host**: `spectralartfoundation.org`, `www.spectralartfoundation.org`,
    or any `consumerbreak.com` host; https; ending `.png .jpg .jpeg .webp .gif`; ≤ 40 MB.
    Other hosts are refused **by design** (`error: art-host`) — a crafted link must not be able
    to make the page fetch from anywhere. If the art lives elsewhere, get it hosted on an
    allowed host first, or use the upload path.
  - A local file, uploaded through the page's drop zone / artwork input.
- **Words:** any mix of the phrase map below. Anything unmentioned falls back to defaults.

## Suggested defaults

| Field | Default | Notes |
|---|---|---|
| `modes` | `["portal"]` | Add `"overlay"` / `"hue"` when asked; multiple modes render side by side |
| `format` | `webm` | Smallest. `mp4` for sharing / X / phones, `gif` when asked |
| `loop` | 60 | Portal loop seconds (32 if pattern ≠ cycle); or set `durations` per effect |
| `fps` | 25 | Allowed: 10 / 20 / 25 |
| `pattern` | `cycle` | The site's signature look |
| `stops` | Site default 7-stop spectrum | 2–16 hex colours, walked in order |

## Phrase map (natural words → settings fields)

| The user says | Set |
|---|---|
| "portal it" / "womp it" | `modes` includes `"portal"` |
| "hue spin / hue cycle it" | `modes` includes `"hue"` |
| "overlay" / "backlit" | `modes` includes `"overlay"` (`backlit` accepted as alias) |
| "just hue, no frame" | `modes = ["hue"]` |
| "all three" / "show me everything" | `modes = ["portal","overlay","hue"]` |
| "solid / sweep / ripple / pulse / spiral / marble / plasma / turbulence / texture" | `pattern` = `solid` / `sweep-x` (or `-y` / `-d`) / `ripple` / `pulse` / `spiral` / `marble` / `plasma` / `turbulence` / `texture` |
| "run the cycle" | `pattern = "cycle"` (members `cycleMembers`, hold `cyclePer` s, default 6) |
| "cycle fast" | `cyclePer` ≈ 1.5–2 |
| "custom colours #… #…" | `stops = ["#rrggbb", …]` (2–16) |
| "30 seconds" / "a minute" | `loop` = 30 / 60 (or `durations` per effect) |
| "quick / draft" | `fps = 10`, `loop` 30 unless told otherwise |
| "smooth" | `fps = 20` |
| "max quality" | `fps = 25` |
| "more pure / softer" | `purity` up / down (0–100, default 100) |
| "pixelly / frame pitch" | `pitch` up (0–300; 0 = smooth). Pixelblaze layout = `frame` {strips, perStrip, vertical, startRight, startTop} |
| "for X / for Instagram / for my phone" | `format = "mp4"` |
| "small file" | `format = "webm"` |
| "gif" | `format = "gif"` |
| "reverse it" | `reverse = true` |
| "spin it" | `rotate` (0–360) |

Anything the phrase map can't parse: ask one short question. Never guess a pattern or a
colour set silently. Named colour presets ("fire", "ocean", "sunset") are **captured from
the site once** (set the preset in the UI, Copy current, store the `stops` array here) —
never invented.

## Workflow

1. **Build the settings block** from the art + phrase map + defaults. Include
   `"spectralbreak": 1`.
2. **Open the full-state URL** (URL-art path):
   `https://spectralbreak.consumerbreak.com/?art=<encoded>&s=<base64url block, no padding>&autocreate=<format>`
   Scalar fields can also ride as plain params (`&pattern=plasma&loop=30`) and override `s`.
   For a local file, open the plain site and upload via the drop zone, then load the block
   through Advanced → Settings → "Load settings" (or `window.SpectralBreak.load(...)`).
3. **Verify before render.** Read `#spectralbreak-state` (JSON island, rewritten on every
   change) and confirm the settings match what you sent; note the `settingsHash`. Mismatch =
   stop and report — never render the wrong look. If your tooling can't read the island,
   verify in the visible UI controls + the settings box instead, and say so.
   ⚠️ The page loads with sample art already `ready:` — always confirm the status line
   names *your* art before creating.
4. **Render.** `autocreate` starts the export when the art is ready, or click Create
   (`sb-create`) / call `window.SpectralBreak.create(format)`. Wait for `#sb-status` =
   `done: <filename>` or the `spectralbreak:done` event `{filename, format, frames, bytes,
   settingsHash}`. A 60 s / 25 fps render is 1,500 frames; expect up to a few minutes at
   large sizes (600 frames at 1600×1600 took under a minute in testing).
5. **Download.** The download is a link you click (`sb-download`) — nothing downloads on
   its own. Verify the file exists, is non-empty, the extension matches, and the filename's
   hash matches the state hash. Exports carry **no embedded metadata** — the 8-hex settings
   hash in the filename (`spectralbreak-<art>-<effects>[-<pattern>]-<seconds>s-<hash>.<ext>`)
   IS the look's identity. Same settings → same frames; bytes may differ by encoder
   timestamps, so compare hashes, not files.
6. **Deliver** the file with one line — art, pattern, colours, loop, format — **and the
   full-state URL** (URL-art path), so anyone can reproduce or tweak the exact look. For
   uploaded art, deliver the settings block JSON instead (there is no art URL).

**Errors** arrive as one line in `#sb-status`: `error: <code> - <why>` — codes include
`art-host`, `art-fetch`, `art-http-<status>`, `art-type`, `art-size`, `settings-invalid`,
`encode-failed`. On error the page keeps its previous settings. Report the code verbatim;
don't retry blindly.

**Script API** (for agents with a JS surface): `window.SpectralBreak` —
`getSettings()`, `getState()`, `load(blockOrJson)` → `{ok, errors, warnings}`,
`create(format?)` → Promise of the done detail. Events on `window`:
`spectralbreak:ready / progress / done / error`. Test ids: `sb-art-input`, `sb-create`,
`sb-download`, `sb-format`, `sb-pattern`, `sb-settings-box`, `sb-load-settings`,
`sb-copy-settings`, `sb-status`, `sb-preview-canvas`.

## Rules

- **Verify before render, verify after download.** State or hash mismatch = stop.
- **Never invent settings.** Unparsed words → one question, or a stated default — and say
  which default was used in the delivery line.
- **No account features.** "Save Look" and "Send to PortalBreak" are never used (they are
  also the only two things on the site that store data). Everything runs signed-out,
  client-side, matching the site's privacy design.
- **Allowed hosts only** for `?art=`. A refusal (`art-host`) is deliberate anti-abuse
  design, not a bug to work around.
- **One render at a time** per browser window; queue further requests.

## Examples

- "SpectralBreak the collage, 30 seconds, for X"
  → `?art=…collage.webp` · `{"spectralbreak":1,"modes":["portal"],"pattern":"cycle","loop":30,"fps":25,"format":"mp4"}`
- "Womp this photo, plasma, quick draft" → upload · `pattern:"plasma"` · `loop:30` · `fps:10` · `format:"webm"`
- "Everything it's got, 16 seconds" → the triptych in `examples/triptych-maximal-16s.json`
  (all three modes, cycle at 1.6 s, spread/detail maxed, 12-stop rainbow — renders
  4800×1600 side by side)

## What this skill is not

- Not an interface the site depends on — any agent can drive SpectralBreak without it.
  This skill is the convenience layer: phrasing, defaults, and verification discipline.
- Not a renderer — all rendering stays in SpectralBreak, in the browser, private by design.

---

*Field-tested 2026-11-23 against build S309: two end-to-end agent runs (single-mode 30 s
Portal/Cycle, hash `be50c628`; maximal 16 s triptych, hash `22ec43ab`), both verified
frame-by-frame after download. Site by ConsumerBreak; this skill is a community companion,
shared to spread Spectral Art.*
