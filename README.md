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

- **Stage 8.3 (CLIP zero-shot attacker):** implementation complete — new
  cells in `CAPTCHA_Part2_Step1_Dataset_EDA_6.ipynb` (Step 5) load a
  pretrained CLIP checkpoint, encode the real target/distractor class names
  as text prompts, cache an embedding per crop (`coco_subset/clip_embeddings_cache.pkl`,
  reused by later stages), and run the same 60-grid seeded evaluation the
  Stage-4 CNN used, for a directly comparable solve rate. Not yet executed —
  see `STAGE_8_3_STATUS.md` for why (a network limitation in the session that
  wrote it, not a code issue) and what running it in Colab will produce.
- **Stages 8.4–8.9** (PGD dual-attacker hardening, CNN-only control,
  multi-attacker evaluation matrix, human usability study, consolidation,
  final write-up): not started.
