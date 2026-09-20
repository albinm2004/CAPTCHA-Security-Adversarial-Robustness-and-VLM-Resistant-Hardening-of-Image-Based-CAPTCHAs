# Real results — Colab run, 2026-09-20 (fresh Run-All at PGD_EPSILON = 2/255)

This was a fully fresh run — disconnected/deleted the old runtime, uploaded
the fixed notebook, `Runtime → Run all` end to end — the third independent
execution of Stages 8.3–8.6 (after 2026-09-19 and the first 2026-09-20 run,
both at `PGD_EPSILON = 8/255`). This run used the lowered budget
(`PGD_EPSILON = 2/255`, `PGD_ALPHA = 0.5/255`, `PGD_STEPS = 20`) committed
in `5356056`.

## The good news: the perceptual check now passes

```
Checking Stage 8.5 (CNN-only hardened) crops...
Stage 8.5 (CNN-only): 1058 crops checked, SSIM pass rate = 99.9% (mean SSIM 0.9714, worst 0.8556)

Checking Stage 8.4 (dual hardened) crops...
Stage 8.4 (dual): 1058 crops checked, SSIM pass rate = 99.8% (mean SSIM 0.9715, worst 0.8561)
```

Up from ~15.7–15.8% at epsilon=8/255. At the new budget, hardened crops are
essentially indistinguishable from their originals by SSIM. LPIPS was still
unavailable this run (no network path to the AlexNet backbone in this Colab
session), so SSIM is the only confirmed perceptual signal so far, but it
went from strongly failing to strongly passing. **The epsilon fix worked as
intended.**

## The bad news: the novelty claim does not survive the fix

**3×4 solve-rate matrix, this run (epsilon = 2/255):**

| image_set | CNN | CLIP | Ensemble | Held-out (ResNet18) |
|---|---|---|---|---|
| original | 0.783 | 0.850 | 0.783 | 0.817 |
| cnn_only_hardened | 0.000 | 0.683 | 0.000 | 0.583 |
| dual_hardened | 0.000 | 0.717 | 0.000 | 0.617 |

Compare against the two previous runs at epsilon=8/255:

| run | epsilon | cnn_only CLIP | dual CLIP | cnn_only held-out | dual held-out |
|---|---|---|---|---|---|
| 2026-09-19 | 8/255 | 0.683 | 0.600 | 0.267 | 0.350 |
| 2026-09-20 (run 2) | 8/255 | 0.683 | 0.500 | 0.317 | 0.383 |
| 2026-09-20 (run 3) | 2/255 | 0.683 | **0.717** | 0.583 | 0.617 |

**The CLIP column has reversed direction.** At 8/255, dual hardening beat
CNN-only against CLIP specifically in both runs (dual's solve rate lower —
better defended — than CNN-only's 0.683: 0.600 and 0.500 respectively).
This was the core novelty claim, and it looked solid across two independent
runs. At 2/255, dual's CLIP solve rate is 0.717 — *higher* than CNN-only's
0.683. **Dual hardening now defends worse against CLIP than CNN-only
hardening does** — the opposite of the claim, at the epsilon that actually
produces a usable CAPTCHA.

The held-out column's pattern (dual doing worse than CNN-only) is at least
consistent across all three runs — that part of the story hasn't flipped,
just gotten less dramatic in absolute terms (0.617 vs 0.583, a smaller gap
than 0.350 vs 0.267 was).

The Ensemble column remains uninformative in this run too (0.000 for both
variants, same root cause as before).

## Interpretation

The apparent evidence for "dual (CNN+CLIP) hardening generalizes better"
was built entirely on runs at an epsilon now known to produce visibly
corrupted images. At a small enough epsilon to actually pass a perceptual
check, that evidence does not hold — if anything, dual hardening is now the
*worse* choice against CLIP specifically, not the better one. This is a
genuine, important, negative finding, not a bug: it says the CLIP loss
term's benefit (found at 8/255) may have depended on room to move pixels
that a perceptually-valid budget doesn't allow.

**This directly changes what should go in the write-up.** The story is no
longer "dual hardening beats CNN-only against smarter attackers" — as of
this run, that hasn't been demonstrated at a usable epsilon. The honest
current state is: CNN-only hardening and dual hardening perform similarly
(dual slightly worse) against CLIP and the held-out model, at the epsilon
that actually keeps the CAPTCHA legible.

## What to try next (not yet done)

1. **Raise `CLIP_LOSS_WEIGHT`** (currently 1.0) so the CLIP loss term has
   more relative pull within the now-smaller epsilon ball, without touching
   epsilon itself. This is the most direct way to test whether the earlier
   effect can be recovered without sacrificing perceptual quality.
2. **Try an intermediate epsilon** (e.g. 4/255) and re-check Section 6.5 —
   there may be a middle ground where perceptual quality is still
   acceptable (even if not 99%+) and the CLIP-specific advantage
   reappears.
3. **If neither recovers the effect**, report it as-is: a real, negative
   result is still a result. "We found that a larger, perceptually-invalid
   perturbation budget produces an apparent dual-hardening advantage that
   disappears at a properly-constrained budget" is itself a legitimate and
   interesting finding about the sensitivity of adversarial-training-style
   objectives to their perturbation budget — worth a paragraph in the
   paper regardless of which way it lands.
4. Don't proceed to Stage 8.7 (human usability study) until one of the
   above resolves the novelty question — the "hardened" condition to test
   with human participants depends on which hardening approach (if either)
   actually earns its claim.
