# Bugfix — 2026-09-19: working-directory bug in Step 5.3

## What broke

Running Step 5.3 (the CLIP embedding cache) in Colab failed with:

```
FileNotFoundError: [Errno 2] No such file or directory: 'coco_subset/clip_embeddings_cache.pkl'
```

## Root cause

The original notebook (before any of today's Stage 8.3/8.4-8.5/8.6 work)
already contained four leftover cells right after Step 4's summary:
`!git status`, `!git clone .../CAPTCHA-Security-....git` followed by
`%cd CAPTCHA-Security-...`, another `!git status`, and an empty cell.

`%cd` in Colab permanently changes the notebook's working directory for
every cell that runs after it. Since Step 5, 6, and 7 (all added today)
were appended *after* those four cells, every relative path they use
(`coco_subset/...`) resolved inside a freshly cloned, empty copy of the
repo instead of the directory Step 1 actually built `coco_subset/` in.
Step 5.3 was simply the first of the new cells to *write* a file, so it
was the first to surface the problem — Steps 6 and 7 would have hit the
same issue if Step 5.3 hadn't failed first.

This was a latent bug already sitting in the notebook; today's new cells
didn't create it, but they didn't account for it either, and should have.

## Fix applied

- Removed the four leftover cells and replaced them with a one-line
  markdown note explaining why (kept for history, not silently deleted).
- Added a defensive `CLIP_CACHE_PATH.parent.mkdir(parents=True, exist_ok=True)`
  immediately before Step 5.3 writes the cache file, so a future
  directory-change anywhere upstream fails safe instead of crashing.
- Scanned the entire notebook for any other `%cd` / `os.chdir` — none
  found; Step 6's hardened-crop directories already use `mkdir(parents=True)`
  and were never at risk.
- Re-validated with `nbformat.validate()` after the edit — passes.

## What you need to do

In your **already-running** Colab session (don't restart — Steps 1-4's
data is still in memory): run `%cd /content` in a new cell, then re-run
the Step 5.3 cell onward. Everything after that will now write to the
correct location.

For any **future fresh run**, pull this fix first (`git pull`) — the
broken cells are gone, so this won't recur.
