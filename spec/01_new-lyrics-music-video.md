# 01 — New-lyrics music video

Status: draft · Branch: `new-lyrics` · Date: 2026-09-30

## 1. Context

This repo renders a code-generated music video for "I'm Upping My P(doom)". It uses three.js and
TypeScript, and runs on Bun. Every frame is a deterministic function of song time. Four inputs drive it:

| Input | File | Produced by |
|---|---|---|
| Song audio | `audio/pdoom.mp3` (Suno "Claude-Pop" version, by deckard) | Suno |
| Rough lyric lines | `lyrics/lyrics.src.js` (`[start, end, "line"]`, 46 lines, ~140 s) | hand-written |
| Word/syllable timings | `data/lyrics.json` | `analysis/align.py` |
| Beats/onsets/sections | `data/audio.json` | `analysis/analyze.py` |

The edit lives in `app/src/timeline.ts`. Scene cuts are anchored to lyric text, for example
`cut('There was a sudden drop')`, and snapped to the beat grid. The scenes in `app/src/scenes/*` are
tightly bound to the original lyric imagery (shoggoth, Chinese room, paperclips, Ilya…).

We have new lyrics with the **same structure and rhyme scheme** as the original, and no audio yet.

## 2. Goal

Make a new video with the same voice, sound and visual style as the original, but with the new lyrics.

### Goals
- A new sung track: same singer voice, arrangement, tempo and mix as `audio/pdoom.mp3`, with the new lyrics.
- Word-synced karaoke typography that is accurate to the new vocal.
- New scenes whose imagery literalizes the new lyrics, in the same visual language.
- Final render: 1080p60, with 4K60 optional.

### Non-goals
- No changes to the renderer engine, post-processing, motion-blur sampler or render CLI.
- No new palette, fonts or typographic system.
- No new song structure. The section layout mirrors the original.

## 3. Song structure (to mirror)

| Section | Original lines (first words) | Original time | Scene slot(s) |
|---|---|---|---|
| Intro/Verse 1a | "I see sparks of AGI…" → "that's no surprise" | 1.5–9.0 | `open` |
| Verse 1b | "There was a sudden drop…" → "…you're my boss" | 9.0–17.9 | `loss` |
| Pre 1 | "ChatGPT, please don't eat me alive" | 17.9–23.0 | `prompt` (variant 1) |
| Hook 1 | "I'm upping my P(doom)" ×4 lines | 23.0–29.4 | `hook` n=1, `room` |
| Post-hook 1 | "See through the shoggoth's lies…" | 29.5–35.5 | `shoggoth` |
| Verse 2 | "We had a stable training run…" → "…atoms rearranging" | 38.5–51.9 | `spacetime` |
| Pre 2 | "Sydney, please let me free" | 53.4–58.4 | `prompt` (variant 2) |
| Hook 2 | "I'm upping…" → "…Omega Point's coming soon" | 59.0–65.9 | `hook` n=2, `ascent` |
| Verse 3 | "One E thirty FLOPs…" → "Without a single CDR" | 66.0–88.0 | `ascent`, `bureau`, `leftturn` |
| Pre 3 | "Gato, please don't let me go" | 89.4–95.0 | `prompt` (variant 3) |
| Hook 3 | "I'm upping…" → "…nowhere left to go" | 95.4–102.4 | `hook` n=3, `paperclips` |
| Bridge | "Too late now, we lit the fuse" → "RLHF goes askew" | 102.5–123.4 | `fuse`, `stack`, `dense` |
| Hook 4 | "I'm upping…" → "To recursive self-upgrade" | 123.5–131.9 | `hook` n=4, `loom` |
| Outro | "What did Ilya see?…" → "Was it all for show?" | 132.0–140.5+ | `ilya`, `outro` |

New lyrics must keep roughly the same syllable count per line. The Suno cover follows the original
melody, so extra or missing syllables get rushed or dropped.

## 4. Workstreams

### 4.1 Audio (manual, in Suno)
1. Upload `audio/pdoom.mp3` to Suno (Create → Upload audio). Cover of uploads needs a paid plan.
2. Run **Cover** on the uploaded clip. Paste the new lyrics with section tags
   (`[Verse 1]`, `[Pre-Chorus]`, `[Chorus]`, `[Bridge]`, `[Outro]`).
3. Style prompt: reuse the original's style tags if they are visible on deckard's Suno post. Otherwise
   describe the genre, vocal and mood by ear. Set audio influence high so the voice is kept.
4. Generate several takes. Pick the one with clean diction, the same tempo, and a length within ±3 s of the original.
5. Optional: create a **Persona** from the chosen take. Use it with Replace Section to fix bad lines
   without changing the voice.
6. Export the highest quality available (WAV if the plan allows, else mp3).
7. Fallback if the voice drifts: split the original instrumental with Demucs, then use Suno
   "Add Vocals" over it.

Rights: the original track is deckard's work. Fan and personal use is fine. Get permission before a public or commercial release.

### 4.2 Song inputs
- Replace the audio. Keep the file name `audio/pdoom.mp3` so the paths in `analysis/common.py` (`AUDIO`)
  and `app/src/main.ts:98` do not change. If you rename it, update both.
- Rewrite `lyrics/lyrics.src.js` with the new lines. Start from the original timings, which fit because
  the structure is the same, then adjust by ear.
- In `analysis/pron.py`, replace `PRON`/`ALT` entries with pronunciations for the acronyms and unusual
  words in the new lyrics.
- In `analysis/whisper_run.py:30`, replace the Whisper `initial_prompt` vocabulary with the key terms
  from the new lyrics.

### 4.3 Timing data regeneration
```bash
cd analysis
uv run python -m demucs -n htdemucs_ft -o stems ../audio/pdoom.mp3
# + karaoke lead-vocal model, as in README
uv run python ctc_emissions.py
uv run python whisper_run.py
uv run python vocal_feats.py
uv run python align.py --plots   # -> data/lyrics.json (+ QA plots)
uv run python analyze.py         # -> data/audio.json
```
- Re-measure `STEM_OFFSET_*` in `analysis/common.py` for the new track.
- Fix low-confidence words by adjusting pronunciations or rough line timings, then re-run.

### 4.4 Visuals: new scenes, same style
Keep as-is:
- `app/src/engine/*`: renderer, typography, post, HUD, motion blur
- `app/src/scenes/_motifs.ts`
- Palette: ink `#0A0A0B`, bone `#EEE9DF`, signal orange `#FF4D12`, and one rare accent `#D8FF3C`
- Fonts: Archivo (lyrics), IBM Plex Mono (machine), Cormorant Garamond italic (prophetic)
- Tone: deadpan scientific. Each scene is a "plate from an illustrated treatise".

Work:
1. **Treatment.** Write `docs/TREATMENT-new.md`. Give each scene slot in §3 one visual idiom
   (engraving, oscilloscope, forms, blueprint, raymarched 3D, textile, UI) that literalizes the new lyric.
2. **Hook word.** `app/src/engine/hud.ts:16` finds the sung word `P(doom)` and raises a counter on each
   hit. The `hook` and `outro` scenes build on that counter. Map the new hook word to this mechanic,
   or replace the mechanic with the new hook's own escalating gag.
3. **Scenes.** For each slot, copy the closest existing module as a template. For example: `loss.ts` for
   chart → landscape, `bureau.ts` for forms/schematics, `prompt.ts` + `prompt-data.ts` for a chat-UI
   pre-chorus. Give the copy a new name and rewrite its content. Keep `hook.ts` and update its text.
4. **Prompt variants.** Replace the `chatgpt`/`sydney`/`gato` variants in `prompt-data.ts` with the new pre-chorus addressees.
5. **Timeline.** In `app/src/timeline.ts`, replace every `cut('<old line>')` anchor with the new line's
   opening words, then map each entry to its new scene module.
6. **Cleanup.** Remove old scene modules once nothing imports them.

### 4.5 Render
```bash
cd app && bun install && bunx vite                 # preview at http://localhost:5173
bun scripts/render.ts sheet ...                    # contact sheets for review
bun scripts/render.ts video --samples auto --shutter 0.2 --out ../out/new.mp4
# 4K: add --scale 2 --x264 aq-mode=3:rc-lookahead=30  (~2.5 h on M5 Pro)
```

## 5. Acceptance criteria
- [ ] Suno take: same voice and tempo as the original, every word intelligible, length within ±3 s.
- [ ] `align.py --plots`: word boundaries sit on vocal onsets, and no line has low confidence.
- [ ] Preview: karaoke highlights track the vocal. Every cut lands on a beat.
- [ ] Running `grep` for old lyric anchors (e.g. `shoggoth`, `Sydney`, `Gato`, `Ilya`) in `app/src` returns nothing,
      unless a match is intentionally reused.
- [ ] Every scene uses only palette tokens and the three font families.
- [ ] Contact sheet reviewed, then a full 1080p60 render reviewed end to end.

## 6. Open items
- [ ] New lyrics text: paste into `lyrics/new-lyrics.txt`, then convert to `lyrics.src.js`.
- [ ] Original Suno style tags: check deckard's post.
- [ ] Chosen Suno take (file).
- [ ] New hook word and its HUD gag (§4.4.2).
- [x] Remotes: `origin` = `LizetCh/pdoom-video-remix`, `upstream` = `mexicat/pdoom-video`.
