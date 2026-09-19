# Stage 8.3 status — 2026-09-19

## What was added

New cells appended to `CAPTCHA_Part2_Step1_Dataset_EDA_6.ipynb` (Step 5,
Stage 8.3): a complete CLIP zero-shot attacker —

- Loads a pretrained CLIP checkpoint (`open_clip`, `ViT-B-32`, `openai` weights).
- Encodes the 4 target classes + 6 real distractor classes as text prompts
  (not the CNN's artificial "distractor" bucket — see the notebook's Step 5
  intro for why that's the fairer zero-shot setup).
- Extracts and caches an embedding for every crop used anywhere in the
  pipeline, both in-memory (by object identity, for this run) and on disk
  (`coco_subset/clip_embeddings_cache.pkl`, by a stable `class:ann_id` key)
  so Stage 8.4/8.5's PGD engine can reuse it without recomputing.
- Runs the same 60-grid, same-seed (`pyrandom.seed(2024)`) evaluation the
  Stage-4 CNN used, so the two solve rates are directly comparable.
- Produces a CLIP-vs-CNN-vs-Hossen-et-al. comparison table and chart.

The code was validated for structure (`nbformat.validate`) and syntax
(every new cell compiles cleanly). It was **not executed** — see below.

## Why it wasn't run, and what that means for today's numbers

This session's network (both the cloud workspace and the bridge into your
laptop) blocks every host needed to actually run this:

- `images.cocodataset.org` — the COCO image server Step 1 also depends on
- `huggingface.co` / `cdn-lfs.huggingface.co` — where `open_clip`'s hub
  integration and most CLIP checkpoints live
- `openaipublic.azureedge.net` — OpenAI's original CLIP weight host
- `download.pytorch.org` — torchvision's own pretrained-weight server

All four returned connection-refused/403 from the proxy when tested
directly (not assumed — I checked). PyPI itself works fine, so `pip install
open_clip_torch` will succeed, but the actual model weights have nowhere to
download from in this environment. There is no dataset or checkpoint
already cached in this repo to fall back on — the original Colab run's
processed crops were never committed here (only the notebook code and its
saved output plots were).

**This has nothing to do with your laptop's specs and everything to do with
this session's network policy** — the Colab environment the rest of this
notebook was written for has open internet and will run these new cells
exactly as designed.

## What to do next

Open the notebook in Colab (same as Steps 1–4) and run it top to bottom —
the new Step 5 cells will execute like any other cell once GPU/CPU runtime
with real internet is available. That will produce the real CLIP zero-shot
solve rate, the embedding cache file, and the comparison chart — genuine
numbers, not estimates.

## Honest scope check against the full plan

Stages 8.4 (PGD dual-attacker hardening, **L**), 8.5 (PGD CNN-only control,
**S**), 8.6 (multi-attacker evaluation matrix, **M**), 8.7 (human usability
study, **M**), 8.8 (consolidation, **S**) and 8.9 (final write-up, **M**)
are untouched. Per your own plan's effort sizing these total roughly
4-6+ weeks of work; none of it is achievable in an hour regardless of
network access, and PGD generation (8.4/8.5) specifically wants a GPU to
be practical at any real image count. Nothing here was stubbed out or
faked to look further along than it is.
