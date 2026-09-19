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

- **Critical finding (2026-09-20): `PGD_EPSILON = 8/255` is too large.**
  The perceptual check (SSIM/LPIPS) that was missing on 2026-09-19 has now
  actually been run — only **~15.7–15.8%** of hardened crops pass the
  SSIM ≥ 0.90 "unchanged to a human" threshold, across both hardening
  variants and confirmed on 1,058 crops (mean SSIM ~0.852, worst ~0.54).
  LPIPS is more forgiving but still only ~47.5% pass. **This means most
  "hardened" CAPTCHA images are currently visibly corrupted, not
  imperceptibly perturbed** — undermining the core "hard for bots, easy
  for humans" requirement. See `RESULTS_2026-09-20.md` for the full
  breakdown. **Next step: lower `PGD_EPSILON` (try 2/255 or 1/255) and
  re-run Sections 6.3–7.5** before treating any solve-rate number as a
  final result.
- **Stages 8.3, 8.4/8.5, 8.6 — executed for real across two independent
  Colab runs (2026-09-19 and 2026-09-20)**. See `RESULTS_2026-09-19.md`
  and `RESULTS_2026-09-20.md` for full breakdowns.
  - CLIP zero-shot solve rate **85.0%** (51/60), higher than the Stage-4
    CNN's 76.7% — confirms both attacker types need defending against.
  - The 3×4 solve-rate matrix (original / CNN-only-hardened /
    dual-hardened × CNN / CLIP / Ensemble / Held-out ResNet18) is built
    and now reproduced across two independent runs.
  - **Novelty claim confirmed across both runs**: dual hardening beats
    CNN-only hardening against CLIP specifically (run 1: 0.600 vs 0.683;
    run 2: 0.500 vs 0.683) — the actual point of the CLIP loss term, and
    it holds consistently. *However*, both runs used the epsilon now
    known to cause visible corruption (see the critical finding above),
    so this comparison needs re-confirming once epsilon is lowered.
  - **Two honest caveats, also confirmed across both runs**: the
    Ensemble column is uninformative (CNN's confidence collapse
    dominates the 50/50 vote) in both runs, and dual hardening
    transferred *worse* than CNN-only hardening to the held-out ResNet18
    in both runs (run 1: 0.350 vs 0.267; run 2: 0.383 vs 0.317) — the
    opposite of what "dual defense generalizes better" would predict.
    Neither is smoothed over.
- **Stages 8.7–8.9** (human usability study, consolidation, final
  write-up): not started — blocked on the epsilon fix above for 8.8/8.9,
  and on the team's own recruitment/consent plan for 8.7.
