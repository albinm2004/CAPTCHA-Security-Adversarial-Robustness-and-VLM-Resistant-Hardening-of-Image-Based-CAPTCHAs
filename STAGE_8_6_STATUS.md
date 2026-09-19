# Stage 8.6 status — 2026-09-19

## What was added

New cells appended to `CAPTCHA_Part2_Step1_Dataset_EDA_6.ipynb` (Step 7):
the multi-attacker evaluation matrix —

- **Held-out attacker:** a ResNet18 transfer-learning classifier (same
  recipe as Step 3.7's MobileNetV2), trained on the same 5-way split, but
  never referenced anywhere in Stage 8.4/8.5's PGD loss. This is the
  transfer-robustness check the plan specifically called for — without
  it, a "generalizes" claim would really just mean "beats the exact bots
  it was trained against."
- **Ensemble attacker:** averages CNN softmax confidence and CLIP
  softmax-over-cosine-similarity confidence per candidate label, argmax
  vote. Simple and transparent by design, not a learned meta-classifier.
- **The 3×4 solve-rate matrix:** original / CNN-only-hardened /
  dual-hardened image sets, each run through all 4 attackers (CNN, CLIP,
  Ensemble, Held-out), reusing the exact same `generate_grid_captcha`
  function and re-seeding `pyrandom` before each run so grid layouts stay
  identical across the comparison — only the pixels differ.
- A heatmap and an explicit statement of what result pattern would
  actually support Stage 8.4's novelty claim (dual-hardening beating
  CNN-only hardening specifically on the CLIP/Ensemble/Held-out columns,
  not collapsing back to baseline on the held-out one).

## What was verified

Same two-tier approach as Stage 8.3/8.4-8.5:

- `nbformat.validate()` passes on the full 114-cell notebook; every new
  code cell compiles cleanly.
- The ensemble voting and matrix-construction logic was functionally
  tested against mock CNN/CLIP/held-out models: every attacker returns a
  valid label, the CLIP softmax sums to 1.0, the resulting matrix is
  exactly 3×4 with every cell populated and in `[0,1]`, and re-seeding
  `pyrandom` reproduces the identical target class and target-cell mask
  across separate calls — confirming the "same grid layout across the
  three image sets" design actually works, not just that it was intended.

## Why it still wasn't run for real

This stage is downstream of Stage 8.3 (embeddings) and 8.4/8.5 (hardened
crops) — none of which have been executed yet, since all three depend on
the same blocked network (COCO images, CLIP/torchvision weights) that has
applied all session. There's nothing to load from
`coco_subset/crops_hardened_cnn_only/` or `crops_hardened_dual/` until
Step 6 has actually been run once with a real GPU + real weights. Running
Step 7 before that would either crash (empty pools) or, worse, silently
produce meaningless numbers from empty/placeholder data — so it wasn't
run, and no numbers are reported.

## What to do next

Run Steps 5, 6, and 7 top-to-bottom in Colab with a GPU runtime, in that
order — each depends on files the previous one writes to `coco_subset/`.
Check Step 6.4's perceptual-check results before trusting anything Step 7
computes from the hardened image sets.

## Honest scope check

Stage 8.7 (human usability study — needs a real recruitment/consent plan,
not code), 8.8 (consolidation script), and 8.9 (final write-up) are still
untouched.
