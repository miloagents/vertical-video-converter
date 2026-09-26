---
name: vertical-video-converter
description: Convert any horizontal or square video into a 9:16 vertical MP4 ready for YouTube Shorts, Instagram Reels and TikTok — blurred-background fill, focal-point crop or letterbox pad, with optional burned-in captions and batch mode. Use when someone says "make this vertical", "convert to 9:16", "resize this for Reels / Shorts / TikTok", "reframe this video", or hands you a 16:9 file that has to ship as a short.
---

# Vertical Video Converter

Reframes a landscape (or square) video into a 1920×1080-safe vertical master. One command in, one vertical MP4 out — no editor, no manual cropping, no guessing where the subject went.

## When to use

- A 16:9 capture needs to ship as a Short / Reel / TikTok.
- A client sent a horizontal export and the platform rejects anything not 9:16.
- You have a folder of clips and need them all vertical with identical framing.

Do **not** use it when the source is already 9:16 and only needs a re-encode — that is a plain transcode. This skill exists to change the *aspect*, not the codec.

## Requirements

- `ffmpeg` and `ffprobe` on `PATH` (any 5.x+ build; the `subtitles` filter needs `--enable-libass`, which every standard full build has).
- Python 3.9+. Nothing else — no pip installs, no network access.

## Quickstart

```bash
python3 scripts/verticalize.py input.mp4                    # blurred-fill vertical
python3 scripts/verticalize.py input.mp4 --mode crop --focus upper
python3 scripts/verticalize.py input.mp4 --captions subs.srt --out short.mp4
python3 scripts/verticalize.py --batch ./clips --outdir ./vertical
```

The script prints a QC table (source vs output: resolution, fps, duration, codecs) and exits non-zero if the output is not the requested size.

## Modes

| Mode | What it does | Use when |
|---|---|---|
| `blur` (default) | Source fits the full width, background is a blurred enlargement of the same frame | Screen recordings, talking heads, anything where **nothing may be cropped away** |
| `crop` | Scales up and crops to fill the frame; `--focus` picks the band that survives | Subject is centred or you know which third matters |
| `pad` | Letterboxes on black | Deliverables that must show the entire frame at native aspect |

`--focus` accepts `center` (default), `top`, `upper`, `lower`, `bottom`, or an explicit fractional point like `0.4,0.35` meaning 40% across, 35% down.

## Captions

`--captions subs.srt` burns SRT subtitles in a short-form-safe style: white, dark outline, bottom-centred, sitting above the zone the platform's own UI occupies. Size and placement are in **output pixels**, not point sizes:

| Flag | Default | Meaning |
|---|---|---|
| `--caption-size` | 56 | caption text height, in pixels of the output frame |
| `--caption-margin` | 320 | pixels between the caption baseline and the bottom edge |

Captions are burned **after** reframing, so they stay inside the safe area instead of being cropped off.

### Why the script rewrites your SRT first

libass lays SRT out on a fixed **384×288** canvas. Anything you pass through `force_style` is therefore multiplied by `height / 288` — on a 1920-tall frame a `FontSize=16` renders as ~107 px of text, and `MarginV=180` pushes the line a third of the way *down* the screen instead of up from the bottom. Both bugs are invisible on short test clips and obvious on the first real one.

`srt_to_ass()` writes an ASS header with `PlayResX`/`PlayResY` set to the real output size, so `Fontsize` and `MarginV` mean exactly what they say. If you hand-roll the command elsewhere, replicate that step — `force_style` alone cannot set the script resolution.

## Output contract

- Container `mp4`, video `h264` (`yuv420p`, `+faststart`), audio AAC 128k if the source has audio.
- Default `1920×1080`… i.e. `1080×1920` vertical: width 1080, height 1920, both even.
- `--width/--height` change it; odd values are rounded down to even automatically.
- CRF 20, preset `medium` by default. `--crf 18 --preset slow` for a final master.

## Batch mode

`--batch DIR --outdir DIR` walks the folder for `mp4/mov/mkv/webm/m4v/avi`, applies the same framing to every clip, and writes `name_vertical.mp4` next to a `batch-report.json` listing per-file source and output properties. A file that fails does not stop the batch; it is recorded as `"status": "failed"` with the ffmpeg stderr tail.

## Verification

Never report success from an exit code alone. The script's QC step already re-probes the output, but when you are working by hand, confirm all four:

```bash
ffprobe -v error -select_streams v:0 -show_entries stream=width,height,codec_name -of csv=p=0 out.mp4
ffprobe -v error -show_entries format=duration -of csv=p=0 out.mp4
ffmpeg -v error -i out.mp4 -f null -            # decode the whole file; silence = no corruption
```

Expect exactly the requested width and height, and a duration within one frame of the source.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| `Invalid argument` on the subtitles filter | Windows path with `:` not escaped | Pass the SRT with a forward-slash path; the script escapes it, hand-written commands need `C\:/path/subs.srt` |
| Captions enormous, or floating mid-frame | `force_style` used without a matching script resolution | Let the script build the ASS header; see *Why the script rewrites your SRT first* |
| Output is 1080×1918 | Odd source height propagated | The script forces even dimensions; if you built the command by hand add `,crop=1080:1920` |
| Blur band appears stretched | `gblur` running on the small pre-scale | Blur before upscaling — the script's graph does this on purpose |
| Audio missing | Source had none, or `-map` dropped it | The script only adds the audio map when the source reports an audio stream |
| Crop cuts the subject | Default centred crop | Re-run with `--focus upper` / `--focus lower` / explicit `x,y` |

## Limitations

- Cropping is geometric, not semantic. There is no face or subject tracking: `--focus` is where *you* point it. A clip whose subject moves across the frame will still need an editor, or a fixed band that happens to cover the action.
- Rotation metadata (`rotate=90` side-data) is respected via `-autorotate`, but a source that mixes rotated and unrotated clips in one batch will produce mixed framing.
- Variable-frame-rate sources are normalised to the first clip's fps in batch mode, which can shift audio sync by a few milliseconds on very long clips.
