# Stage 8.4 / 8.5 status — 2026-09-19

## What was added

New cells appended to `CAPTCHA_Part2_Step1_Dataset_EDA_6.ipynb` (Step 6):
one shared PGD (Projected Gradient Descent) engine, run twice via a single
`use_clip` flag —

- **Stage 8.4 (dual hardening, `use_clip=True`):** untargeted PGD that
  simultaneously raises the CNN's cross-entropy loss on the crop's true
  label and lowers CLIP's cosine similarity to that label's text prompt,
  within an L∞ ball of radius `PGD_EPSILON = 8/255`.
- **Stage 8.5 (CNN-only control, `use_clip=False`):** the identical engine
  and epsilon, only the CNN loss term active — per the plan's explicit
  warning that 8.4 and 8.5 must share one codepath or the comparison
  becomes invalid.
- A differentiable CNN preprocessor (matches Step 2.4's `normalize_for_cnn`
  exactly, just as a tensor op) and a differentiable CLIP preprocessor
  (bilinear resize + CLIP's own mean/std), since PGD needs gradients to
  flow all the way back to the original pixels through both.
- An SSIM perceptual check (always runs, no external weights needed) plus
  an LPIPS check that's attempted and clearly reported as skipped if its
  backbone weights aren't reachable — never silently dropped.

## What was actually verified (not just written)

Unlike Stage 8.3, this one got a real functional test, not just a syntax
check: the whole PGD loop, copy-pasted verbatim from the notebook cell,
was run against tiny mock CNN/CLIP models (random-but-real PyTorch
networks, since torch itself installs fine from PyPI — only the *pretrained
checkpoints* are blocked). That confirmed, with actual numbers:

- The CNN's cross-entropy loss genuinely increases after hardening (the
  point of an untargeted attack) — both with and without the CLIP term.
- CLIP's cosine similarity to the true label genuinely decreases when
  `use_clip=True`.
- The L∞ perturbation never exceeds the configured epsilon (checked
  exactly, not approximately).
- Gradients actually reach the original input pixels through both the CNN
  and CLIP differentiable preprocessors — a common bug source when mixing
  PIL-based and tensor-based preprocessing pipelines.
- SSIM returns 1.0 for identical images and a sane value for the hardened
  ones.

This is meaningfully more confidence than "the code parses" — the math is
correct. What it does *not* tell you is what real MobileNetV2 and real
CLIP actually do to real COCO crops; that needs the real checkpoints.

## Why it wasn't run on the real pipeline, and what that means

Same network wall as Stage 8.3, documented in `STAGE_8_3_STATUS.md`: no
path from this session to COCO's images, HuggingFace, OpenAI's CLIP
weights, or torchvision's pretrained-weight server. On top of that, Stage
8.4/8.5 is far more compute-heavy than 8.3 — every crop needs
`PGD_STEPS = 15` full forward+backward passes through both models, so even
with weights available, this genuinely wants a GPU (a free Colab T4 is
enough) to run in reasonable time over the full crop pool. This is the
stage where the "AI lab GPU" question actually matters.

## What to do next

Run Step 5 (Stage 8.3) and Step 6 (Stage 8.4/8.5) top-to-bottom in Colab
with a GPU runtime. Before trusting any Stage 8.6 number built on the
hardened crops, check the Step 6.4 perceptual-check output — any crop
failing the SSIM (and LPIPS, if available) threshold should be re-hardened
at a smaller epsilon or excluded, not passed downstream as-is.

## Honest scope check

Stage 8.6 (multi-attacker evaluation matrix), 8.7 (human usability study),
8.8 (consolidation) and 8.9 (final write-up) are untouched. 8.6 in
particular needs a held-out attacker (a second CNN architecture or CLIP
checkpoint not used anywhere in 8.4/8.5) that hasn't been built yet.
