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

- **Stages 8.3, 8.4/8.5, 8.6 — executed for real on 2026-09-19** (Colab,
  GPU runtime). Real results, not estimates — see `RESULTS_2026-09-19.md`
  for the full breakdown:
  - CLIP zero-shot solve rate **85.0%** (51/60), actually *higher* than the
    Stage-4 CNN's 76.7% — confirms both attacker types need defending against.
  - The 3x4 solve-rate matrix (original / CNN-only-hardened /
    dual-hardened x CNN / CLIP / Ensemble / Held-out ResNet18) is built.
    Dual hardening beats CNN-only hardening against CLIP specifically
    (0.600 vs 0.683 solve rate) — the actual novelty claim, and it holds.
  - Two honest caveats found in this run: the Ensemble column is
    currently uninformative (CNN's confidence collapse dominates the
    50/50 vote), and dual hardening transferred *worse* than CNN-only
    hardening to the held-out ResNet18 (0.350 vs 0.267) — the opposite of
    what "dual defense generalizes better" would predict. Both are
    documented, not smoothed over.
  - **Known gap, now fixed**: the SSIM/LPIPS perceptual check was defined
    in Step 6.4 but never actually run before Step 7's matrix was
    computed. A new Section 6.5 (added 2026-09-20) runs it for real —
    run this before treating the matrix numbers as final.
- **Stages 8.7–8.9** (human usability study, consolidation, final
  write-up): not started.
