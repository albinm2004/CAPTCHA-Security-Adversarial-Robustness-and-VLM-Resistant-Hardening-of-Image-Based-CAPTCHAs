# CAPTCHA Security: Adversarial Robustness and VLM-Resistant Hardening of Image-Based CAPTCHAs

## Overview

This project studies how modern image-based CAPTCHAs hold up against two very
different kinds of automated attackers, and builds a pipeline for making them
tougher against both:

- **Simple bots (CNNs)** — Convolutional Neural Networks trained to recognize
  specific object categories in an image (e.g. "bus", "car"). Fast and narrow.
- **Smart AI bots (CLIP / VLMs)** — Vision-Language Models that produce a full
  natural-language description of an image, giving them a much richer
  understanding than a simple classifier.

A CAPTCHA that only defends against the first kind of bot is not safe from the
second. This project builds an image-grid CAPTCHA generation and evaluation
pipeline, benchmarks classifier performance against it, and prototypes an
adversarial hardening loop (based on PGD — Projected Gradient Descent) that
nudges CAPTCHA image pixels just enough to fool both bot types while staying
visually unchanged to a human.

## Project Stages

1. **Dataset & EDA** — Built a labeled subset from COCO 2017 images, checked
   class balance, image quality, and bounding-box size distribution.
2. **Preprocessing & Augmentation** — Quality filtering, cropping/resizing,
   normalization, and train-only augmentation.
3. **Classifier Benchmarking** — Compared classical ML, CNN, and MobileNetV2
   models on grid-solve accuracy (5-way classification), with confusion-matrix
   analysis.
4. **Adversarial Hardening (in progress)** — CLIP embedding extraction, PGD
   baseline attacks, and an ensemble adversarial-training loop that hardens
   CAPTCHA images against both CNN and CLIP/VLM bots simultaneously, while
   keeping the image human-recognizable.

## Team

- **Albin Mathew**
- Francis Vishal S
- Neha Jose

**Mentor:** Dr. Aruna S K

CHRIST (Deemed to be University)

## Status

Progress report (Part 2) covers the dataset pipeline, classifier benchmarking,
and the adversarial hardening plan.

- **Critical finding (2026-09-20, updated): lowering epsilon fixed
  perceptibility but broke the novelty claim.** Two runs at
  `PGD_EPSILON = 8/255` showed only ~15.7–15.8% of hardened crops passing
  the SSIM ≥ 0.90 "unchanged to a human" threshold — most "hardened"
  images were visibly corrupted. Lowering to `PGD_EPSILON = 2/255` fixed
  that (**99.8–99.9% SSIM pass rate**, mean SSIM ~0.971), but the
  dual-hardening-beats-CNN-only-against-CLIP result that had held across
  two runs at 8/255 **reverses at 2/255**: dual hardening's CLIP solve
  rate (0.717) is now *higher* (worse defended) than CNN-only's (0.683).
  **The apparent novelty was an artifact of an epsilon too large to be
  perceptually valid.** See `RESULTS_2026-09-20_run3.md` for the full
  breakdown and next steps (raising `CLIP_LOSS_WEIGHT`, or trying an
  intermediate epsilon like 4/255, are the two untried options before
  concluding the dual-hardening advantage doesn't exist at a usable
  budget).
- **Stages 8.3, 8.4/8.5, 8.6 — executed for real across three independent
  Colab runs (2026-09-19, and twice on 2026-09-20)**. See
  `RESULTS_2026-09-19.md`, `RESULTS_2026-09-20.md`, and
  `RESULTS_2026-09-20_run3.md` for full breakdowns.
  - CLIP zero-shot solve rate **85.0%** (51/60), higher than the Stage-4
    CNN's 76.7% — confirms both attacker types need defending against.
    This part is stable across all three runs (CLIP is unaffected by the
    hardening epsilon, since it's only ever evaluated on original images
    for this baseline number).
  - The 3×4 solve-rate matrix (original / CNN-only-hardened /
    dual-hardened × CNN / CLIP / Ensemble / Held-out ResNet18) is built
    and has now been run three times, at two different epsilon values.
  - **Novelty claim (dual beats CNN-only against CLIP): held at
    epsilon=8/255 across two runs (0.600 vs 0.683, then 0.500 vs 0.683),
    but reverses at the perceptually-valid epsilon=2/255 (0.717 vs
    0.683)** — currently not supported at a usable budget. This is the
    single most important open question in the project right now.
  - **Ensemble column is uninformative across all three runs** (CNN's
    confidence collapse dominates the 50/50 vote) — still needs
    redesigning.
  - **Held-out transfer**: dual hardening has transferred worse than
    CNN-only hardening to the untrained ResNet18 in all three runs (the
    gap narrows at the lower epsilon: 0.617 vs 0.583, vs. 0.350 vs 0.267
    and 0.383 vs 0.317 at the higher epsilon) — consistent direction,
    smaller magnitude.
- **Stages 8.7–8.9** (human usability study, consolidation, final
  write-up): not started — blocked on resolving the novelty question
  above for 8.8/8.9, and on the team's own recruitment/consent plan for
  8.7.
