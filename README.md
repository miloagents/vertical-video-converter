# Vertical Video Converter

Reframe any 16:9 or square video into a 9:16 vertical MP4 for Shorts, Reels and TikTok — with optional burned-in captions.

Three reframing modes: **blur** (the source fits the full width and the background is a blurred enlargement of the same frame, so nothing is cropped away), **crop** (scales up and fills, with `--focus` choosing the band that survives) and **pad** (letterboxes on black).

It can burn SRT captions in a short-form-safe style — white with a dark outline, positioned above the zone the platform's own UI occupies — and it documents the trap that catches almost everyone: libass lays SRT out on a fixed 384x288 canvas, so a `force_style` font size is multiplied by `height / 288` and renders roughly seven times too large on a 1920-tall frame. The method rewrites the SRT into an ASS header with the real output resolution before burning.

Batch mode walks a folder, applies identical framing to every clip and writes a per-file report. Output contract: mp4 / h264 / yuv420p / +faststart, 1080x1920 by default, AAC 128k when the source has audio.

## What is in this repository

This is the **free edition**: `SKILL.md`, the complete method write-up — the part that actually does the work.

| File | What it is |
|---|---|
| [`SKILL.md`](SKILL.md) | the full method, written to be read by an AI assistant |
| [`LICENSE.txt`](LICENSE.txt) | licence for the free edition |

> **Note on the free edition.** The free edition in this repository is the method write-up. The runnable `scripts/verticalize.py` referenced in the Quickstart section ships with the paid package on Agensi — the free edition here does not include the script.

## How to use it

It works with any AI client that reads a skill file — WorkBuddy, Claude Code, Codex, Cursor — or with no
client at all:

1. **As a skill.** Put `SKILL.md` where your client looks for skills (usually a folder named after the
   skill, containing `SKILL.md`).
2. **As a prompt.** Paste `SKILL.md` into a conversation with any capable model and then ask your question.

## What the paid editions add

Deeper reference files (scoring rubrics, checklists, platform heuristics) and, where applicable, runnable
scripts. The hosted versions need no setup at all.

| Where | What you get |
|---|---|
| [Agensi](https://agensi.io/creators/alpha-lay) | the full package, including source files |
| [Poe](https://poe.com/VerticalVideoLab) | hosted — no install, just talk to it |
| [PromptBase](https://promptbase.com) | selected tools |
| [miloagents.shop](https://miloagents.shop/skills/vertical-video-converter/) | this skill's page, plus free episode kits |

More about this skill: <https://miloagents.shop/skills/vertical-video-converter/>

## Licence

See [LICENSE.txt](LICENSE.txt). In short: use it in your own work, personal or commercial, and modify it
freely — but do not resell it or pass it off as your own product.

---

Built by **Alpha Lay**. More tools, and a 1,274-case study of AI video that these methods were tested
against: <https://aishifu.shop/>

## Official site

This free edition, the page for this skill, and the free episode kits live on the official site:

- **This skill:** <https://miloagents.shop/skills/vertical-video-converter/>
- **Free episode kits:** <https://miloagents.shop/kits/>
- **More from Milo:** <https://github.com/miloagents>
