# gemini-interactions — Project Memory

> Agent-facing durable memory for this repo. Auto-loaded via `@MEMORY.md` in CLAUDE.md.
> Human narrative / status: none (no matching Obsidian project note found).
> Related global memory: [[reference_gemini_interactions_api]].
> Update this after significant work.

## What this is
Open-source, MIT-licensed CLI wrapping Google's Gemini Interactions API (GA June 2026) — one self-contained `uv`-run script (`gemini.py`) with subcommands for image gen (Nano Banana 2/Pro), music (Lyria 3), TTS, and grounded/Deep Research (Maps + Search). Built and published by PixelCove 2026-06-22/23; public repo, not client-specific. Status: shipped/live, folded into single CLI at commit `a886d17`.

## Stack & key commands
- Python, single-file script (`gemini.py`), PEP 723 inline deps (`google-genai`), run via `uv run --script` — no install step.
- Run: `uv run gemini.py <image|music|tts|research> --help` for flags per subcommand.
- Examples: `uv run gemini.py image --prompt "..." --aspect-ratio 1:1 --image-size 512 --out leaf.png`; `uv run gemini.py tts --text "..." --voice Kore --out vo.wav`; `uv run gemini.py music --prompt "..." --run --out beat.mp3`; `uv run gemini.py research --deep --query "..." --out report.md`.
- No dev server / build / test scripts in this repo — it's a CLI tool, not a service.

## Architecture / where things live
- `gemini.py` — the entire CLI (434 lines), argparse-based, one subcommand per Gemini surface (image/music/tts/research).
- `SKILL.md` — Claude Code skill wrapper (frontmatter + quick-reference table) so the CLI is invokable as a skill.
- `examples/` — sample outputs (e.g. `voiceover.wav`).
- `README.md` — full usage docs, setup, HyperFrames TTS hookup notes.
- `LICENSE` — MIT.

## Durable decisions & gotchas
- Key resolution order: `GEMINI_API_KEY` / `GOOGLE_API_KEY` / `GOOGLE_GENAI_API_KEY` env → optional `op read $GEMINI_OP_REF` (+ `$GEMINI_OP_REF_FALLBACK`) if `op` present. Genericized — no PixelCove-specific op refs baked in (this is the public repo).
- Image/music require a **billing-enabled** Gemini API key; Deep Research/grounding may need those features enabled on the key's Cloud project.
- Model IDs: image = `gemini-3.1-flash-lite-image` (NB2 Lite, ~4s, $0.034/1K image) / `gemini-3.1-flash-image` (NB2, default) / `gemini-3-pro-image` (NB Pro, high-fidelity text-in-image). `--model` accepts the aliases `lite` / `nb2` / `pro` or a full ID (`IMAGE_MODEL_ALIASES`). **Verified live 2026-08-10: BOTH Flash tiers accept 512px — the old "512 is NB2 only" guard was wrong and is fixed** (`IMAGE_MODELS_WITH_512`). Music = `lyria-3-pro-preview` / `lyria-3-clip-preview`; TTS = `gemini-3.1-flash-tts-preview`; research = `gemini-3.5-flash` + `deep-research-*` agents.
- `research --mode both` runs Maps and Search grounding as **two separate passes and merges sources** — Google's API cannot combine `google_maps` + `google_search` in one request (400 `invalid_request`).
- TTS output is raw 24kHz 16-bit mono PCM base64 — the CLI wraps it in a WAV header itself.
- Deep Research: `background=True`, poll `interactions.get(id)` until `status=="completed"`; response `steps[]` order is not guaranteed to put report text first (a chart/image block can be `steps[-1]`) — must scan blocks, not assume position.
- All image outputs carry Google's SynthID watermark.
- This repo is the canonical, decoupled home for Gemini Interactions logic; PixelCove-internal consumers (`/seo-image-gen`, `/seo-local-research` skills, `ai-music-label-lab`) each have their own copies/adaptations rather than importing this directly — keep behavior changes in sync manually if fixing bugs found here.
- Published under the PixelCove GitHub org per house rule (never personal `jg-cc`).

## Current focus / open threads
_unknown_ — no open TODOs surfaced in README/SKILL.md; last known state is "shipped" (single-CLI fold, commit `a886d17`).

## Links
- Obsidian: none
- Repo remote: https://github.com/PixelCove/gemini-interactions.git
- Related memory: [[reference_gemini_interactions_api]]
