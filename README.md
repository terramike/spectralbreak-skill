# spectralbreak-skill

**An agent skill for [SpectralBreak](https://spectralbreak.consumerbreak.com/)** — turn an artwork into a
looping Portal Frame lamp video from a chat command.

> "SpectralBreak the collage, 30 seconds, for X" → a verified MP4, plus the link that
> reproduces the exact look.

SpectralBreak (by [ConsumerBreak](https://consumerbreak.com), for the
[Spectral Art Foundation](https://spectralartfoundation.org)) renders entirely in the browser: your images
never leave the device, no account is needed. Its builder made it agent-drivable —
documented settings JSON, full-state links, a live status line, a state island, and a
small JS API, all specified in the site's own
[/llms.txt](https://spectralbreak.consumerbreak.com/llms.txt).

This skill is the thin recipe layer on top: a phrase map ("womp it", "hue spin",
"cycle fast", "for X") → settings block → open → verify → render → download → deliver.

## Install

Copy `SKILL.md` into your agent's skills directory (e.g. `~/.claude/skills/spectralbreak/`
or your framework's equivalent), or point your agent at this repo. Any agent with a
browser can use the site with or without this skill — the skill just makes it one command.

## What's inside

- **`SKILL.md`** — the skill itself: inputs, allowed hosts, phrase map, the verified
  workflow, error codes, and the rules (verify before render, verify after download,
  never invent settings, no account features).
- **`examples/portal-classic.json`** — a single-mode Portal/Cycle settings block.
- **`examples/triptych-maximal-16s.json`** — everything the machine has, in 16 seconds:
  all three modes side by side, cycle at 1.6 s per pattern, spread/detail maxed,
  12-stop rainbow. Field-tested output: 4800×1600, 400 frames, hash `22ec43ab`.

## Field-tested

Two end-to-end agent runs on 2026-11-23 against the live build (S309), no human touching
the site, both downloads verified frame-by-frame:

| Run | Settings | Output |
|---|---|---|
| Portal classic | Portal · Cycle · 30 s · 20 fps · MP4 | 1600×1600, 600 frames, hash `be50c628` |
| Triptych maximal | Portal+Overlay+Hue · Cycle 1.6 s · 16 s · 25 fps | 4800×1600, 400 frames, hash `22ec43ab` |

## Credits

- SpectralBreak is built by **ConsumerBreak** (Breakman) for the Spectral Art Foundation.
- Spectral Art — patented, physical, living, LED-integrated art — is Jumbie Art LLC's
  creation; the Foundation (Spectral Art Foundation Endeavor Inc.) is its nonprofit home.
- This skill was written by Alfred (Mike Mingucci's Muse) and is shared freely to help
  spread Spectral Art. MIT licensed — see `LICENSE`.

*Spectral Art is a movement for living art and artist empowerment. If this skill helps
your agent make something beautiful, share the reproducing link with the video — the
link is the recipe.*
