---
name: video-use
description: Process raw footage — cut silence, remove filler words, color grade, normalize loudness, and extract word-level transcripts. Use as the first stage of the full studio pipeline before handing off to HyperFrames for motion graphics. Invoke when the user says "edite o bruto", "corte os silêncios", "remova os vícios de linguagem", "faça o primeiro corte", "transcreva o vídeo", "normalize o áudio", "pipeline completo desde o bruto", or when raw recordings (`.mov`, `.mp4`, `.mkv`) need to be processed before motion-graphics work.
---

# video-use — Raw Footage Pipeline

Python-based AI video editing pipeline that operates on raw recordings and produces a clean-cut MP4 ready for HyperFrames motion-graphics work.

**Upstream of HyperFrames.** Always run video-use first; HyperFrames is the finishing layer.

---

## Pipeline position in this studio

```
raw.mov / raw.mp4
  ↓
[Stage 1 — video-use]
  transcribe.py     → word-level timestamps + speaker diarization
  pack_transcripts.py → compact .md for LLM reasoning
  LLM proposes EDL  → user confirms strategy
  render.py         → cut, grade, loudnorm → edit/final.mp4
  ↓
[Handoff: copy to project assets]
  cp edit/final.mp4 assets/edited.mp4
  (re-encode to H.264 if needed — see Asset Prep in CLAUDE.md)
  ↓
[Stage 2 — HyperFrames]
  <video src="assets/edited.mp4"> in composition
  kinetic-type captions from transcript JSON
  motion graphics, overlays, transitions, outros
  npx hyperframes render → final.mp4
```

---

## One-time installation (Windows)

```bash
# 1. Clone the repo (do NOT symlink on Windows — dev mode required)
git clone https://github.com/browser-use/video-use C:/tools/video-use
cd C:/tools/video-use

# 2. Install Python package (requires Python >= 3.10)
pip install -e .

# 3. FFmpeg — use winget or chocolatey (brew doesn't exist on Windows)
winget install --id=Gyan.FFmpeg -e
# or: choco install ffmpeg

# 4. API key for ElevenLabs Scribe (word-level transcription)
cp .env.example .env
# Edit .env and add: ELEVENLABS_API_KEY=your_key_here
# Key: https://elevenlabs.io — billing is per-minute of audio transcribed

# 5. Optional (for Manim animations)
pip install manim
```

> **Cost note**: ElevenLabs Scribe bills per audio minute. A 10-minute recording ≈ $0.10–0.30 depending on tier. Transcripts are cached per file — re-running on the same file doesn't re-charge.

---

## Per-project workflow

Always run helpers from inside the project folder where the raw videos live:

```bash
cd video-projects/<project-slug>
```

---

## Step 1 — Transcribe

Extracts word-level timestamps + speaker diarization via ElevenLabs Scribe.

```bash
# Single file
python C:/tools/video-use/helpers/transcribe.py raw.mp4

# With options
python C:/tools/video-use/helpers/transcribe.py raw.mp4 \
  --edit-dir edit \
  --language pt \
  --num-speakers 1

# Batch (all .mp4 in current dir)
python C:/tools/video-use/helpers/transcribe_batch.py .
```

**Output**: `edit/transcripts/<stem>.json` — word arrays with timestamps, speaker IDs, audio events.

**Caching**: skips re-upload if transcript already exists. Safe to re-run.

---

## Step 2 — Pack transcripts

Consolidates word-level JSON into a compact phrase-level markdown (one hour of audio ≈ 12KB).

```bash
python C:/tools/video-use/helpers/pack_transcripts.py \
  --edit-dir edit \
  --silence-threshold 0.5
```

**Output**: `edit/takes_packed.md` — time-coded phrases, silence gaps, speaker tags.

Format example:
```
  [000.00-005.23]S0 Olá pessoal, hoje a gente vai falar sobre
  [006.10-009.45]S0 os principais erros de iniciantes em motion design
```

---

## Step 3 — LLM proposes EDL (conversational)

After packing, share `edit/takes_packed.md` with the LLM and describe the editing intent:

> "Corte os silêncios acima de 0.8s, remova os 'hmm' e 'né' isolados, e mantenha só o take 2 da intro."

The LLM proposes an editing strategy in plain English. **Wait for user confirmation before building the EDL.**

The EDL is a JSON array of `{source, start, end, grade?}` objects:

```json
[
  { "source": "raw.mp4", "start": 12.4, "end": 45.1 },
  { "source": "raw.mp4", "start": 47.8, "end": 112.3, "grade": "subtle" },
  { "source": "broll.mp4", "start": 0.0,  "end": 8.0,  "grade": "neutral_punch" }
]
```

Save as `edit/edit.json`.

---

## Step 4 — Render

```bash
# Full render with loudness normalization (default)
python C:/tools/video-use/helpers/render.py edit/edit.json -o edit/final.mp4

# Preview quality (faster, 720p)
python C:/tools/video-use/helpers/render.py edit/edit.json -o edit/preview.mp4 --preview

# Skip loudnorm (if HyperFrames will mix audio separately)
python C:/tools/video-use/helpers/render.py edit/edit.json -o edit/final.mp4 --no-loudnorm

# ⚠️ DO NOT use --build-subtitles in this studio
# Subtitles belong to HyperFrames (kinetic-type, MOTION_PHILOSOPHY-aligned)
# The transcript JSON is the source of truth for captions at Stage 2
```

**Quality tiers**:
| Flag | Resolution | Preset | CRF | Use when |
|------|-----------|--------|-----|----------|
| (default) | 1080p | fast | 20 | deliverable |
| `--preview` | 720p | ultrafast | 28 | quick checks |

**Loudness normalization**: two-pass, targets **-14 LUFS** integrated, **-1 dBTP** peak, LRA 11 — social media standard. Runs automatically unless `--no-loudnorm`.

---

## Step 5 — Visual timeline inspection (optional, before render)

Generates filmstrip + waveform PNG for a time range — useful for checking a cut boundary before committing.

```bash
python C:/tools/video-use/helpers/timeline_view.py raw.mp4 \
  --start 12.0 --end 20.0 \
  --transcript edit/transcripts/raw.json \
  -o edit/timeline_12-20.png
```

Read the PNG with the `Read` tool to inspect cut points visually.

---

## Color grade presets

Per EDL entry (`"grade": "<preset>"`). Default when omitted: `auto`.

| Preset | Effect | Use when |
|--------|--------|----------|
| `auto` | Math-based correction ±8% (exposure, saturation, contrast) | **default** — safe, no color shift |
| `subtle` | `contrast=1.03 saturation=0.98` | Barely-perceptible cleanup. Use for face-cams going into HyperFrames |
| `neutral_punch` | Light S-curve, no hue shift | Talking-head with good lighting |
| `warm_cinematic` | Crushed blacks, warm shadows, cool highlights | **Opt-in only** — conflicts with HyperFrames shader overlays |
| `none` | No filter | Skip grading for a specific segment (b-roll already graded) |

**Studio rule for this workspace**: use `subtle` or `auto` on face-cam footage. `warm_cinematic` only if no HyperFrames color passes are planned (it clashes with grain/vignette/shader layers).

---

## Step 6 — Hand off to HyperFrames

```bash
# Move the clean edit to the project's assets folder
cp edit/final.mp4 assets/edited.mp4

# Re-encode to H.264 if the source isn't already (see CLAUDE.md Asset Prep)
ffmpeg -i edit/final.mp4 -c:v libx264 -preset medium -crf 20 \
  -c:a aac -b:a 192k -movflags +faststart assets/edited.mp4
```

In the HyperFrames composition:
```html
<!-- The edited clip — no class="clip" on <video> -->
<video src="assets/edited.mp4" muted
  data-start="0" data-duration="<duration>" data-track-index="0"
  data-has-audio="true">
</video>

<!-- Captions: build from edit/transcripts/<stem>.json, NOT from burned SRT -->
```

For captions, use the transcript JSON (`edit/transcripts/<stem>.json`) to time-align kinetic-type elements in HyperFrames. Invoke `/hyperframes` for the caption authoring pattern.

---

## Self-evaluation loop

After rendering, `render.py` self-evaluates cut boundaries (up to 3 re-render attempts). If the render exits with a non-zero code, read the stderr — it usually flags the specific boundary that failed validation.

---

## Non-negotiables for this studio

1. **Never `--build-subtitles`** — subtitle design belongs to HyperFrames, aligned to MOTION_PHILOSOPHY.
2. **Grade conservatively** — `subtle` or `auto` on face-cams. Aggressive grading clashes with HyperFrames vignette + grain + shader layers.
3. **Always loudnorm** unless re-mixing in HyperFrames (rare). -14 LUFS is the social standard.
4. **All outputs go to `<project>/edit/`** — never write to project root or workspace root.
5. **Cache transcripts** — don't re-transcribe the same file; it bills again and produces drift.
6. **Confirm the strategy in plain English before building the EDL.** Ask → confirm → execute.
7. **Word boundaries only** — all cuts must snap to transcript word boundaries. No mid-word cuts.

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `ELEVENLABS_API_KEY not found` | Check `.env` in video-use repo root or set env var |
| `ffmpeg not found` | `winget install Gyan.FFmpeg` then restart shell |
| Transcript has gaps / wrong timestamps | Check `--language` flag; default is `en` |
| Render crashes partway | Run `npx hyperframes doctor` to verify FFmpeg version |
| Cut has audible pop | 30ms fades are applied automatically; check if segment is < 60ms |

---

## Related skills

- `/hyperframes` — Stage 2: compose motion graphics over the edited clip
- `/make-a-video` — full end-to-end wizard (includes Stage 1 + Stage 2)
- `/hyperframes-cli` — `transcribe`, `tts`, `render` CLI commands
