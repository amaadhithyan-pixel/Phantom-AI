# MIRAGE Evaluation (Day 11)

Evaluated on the 40-image synthetic invoice set (`phantom/mirage/make_samples.py`,
20 benign/suspicious pairs across 3 template families). Document-level prediction:
"suspicious" if any non-quality finding is suspicious or inconclusive.

```
python -m phantom.mirage.evaluate --samples data/samples
```

## Results (all templates)

| Metric | Value | v1 target (SCOPE.md) |
|---|---|---|
| Precision | 0.67 | — |
| Recall | 0.60 | — |
| F1 | 0.63 | ≥ 0.70 |
| False-positive rate | 0.30 | — |
| Mean runtime / document | 0.30 s | ≤ 5 s/page |

**F1 is below the v1 target.** This is reported as-is rather than tuned to look
better on this sample set. Runtime is well within budget.

## What's driving the gap

- `field_alignment` (robust z-score on amount-column left edges) was initially a
  major false-positive source on benign documents (natural digit-width variation
  tripped a fixed-pixel tolerance); switched to a MAD-based z-score, which removed
  those false positives entirely in this set.
- `region_inconsistency` rarely fires on this synthetic set: our generator paints
  over a field with white before redrawing it, so the patched region's background
  matches the page background exactly. On real scanned/photographed documents
  (where a paste leaves a slightly different white level, shadow, or compression
  artifact) this check should be more informative — it needs testing on
  non-synthetic manipulated documents next.
- `font_consistency` (ink-density outlier) is the main signal catching
  `font_mismatch` manipulations, but misses some `altered_amount` edits where the
  replacement text uses the same font and only the digits changed — those have
  no visual tell for this heuristic to find; catching them needs a numeric
  cross-check (e.g. against OCR'd totals) rather than a purely visual one.

## Next steps (Week 3)

- Add a numeric consistency check (do item amounts sum to the stated total?).
- Re-run this evaluation against real/scanned manipulated documents, not just the
  synthetic set, before relying on these numbers.
- Consider holding out one template family entirely from threshold-tuning to get
  a cleaner read on generalization.
