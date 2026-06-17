# Rubric — business-card-design-harness

This rubric governs how the Evaluator scores a batch of 20 HTML business-card mockups
plus the visual-companion gallery. It is self-contained: the Evaluator subagent reads
this file and `evaluator-prompt.md` and needs nothing else to grade.

All criteria are scored **1–5**. Two axes are weighted **2×** because they are where Claude
is weakest by default on this domain. The verdict logic below is binary and non-negotiable.

---

## The 5 criteria

| # | Criterion | Weight | What it measures | Reference |
|---|-----------|--------|------------------|-----------|
| C1 | Typographic hierarchy (타이포 위계) | **2×** | Name/title/contact form a real 3-tier hierarchy (distinct size + weight + case per tier); a single committed alignment system per card (no mixed alignments); ≤2 type families; the name is kerned; tracking is intentional. | BCR §4.1 |
| C2 | Whitespace balance (여백 균형) | **2×** | Margins keep content off the trim; visual weight is distributed deliberately; one protected focal point; intentional asymmetry rather than center-everything; roughly 40–60% empty for premium-minimal cards. | BCR §4.2 |
| C3 | Brand color coherence (브랜드 색 일관성) | 1× | The specified palette is applied consistently (1–2 brand colors + 1–2 neutrals); small contact text contrast ≥ 4.5:1; the brand color is used as a sparing accent, not a wash behind small text. | BCR §4.3 |
| C4 | Design variety / anti-slop (디자인 다양성) | 1× | The 20 cards traverse distinct archetypes from the BCR §5 catalog; no two cards share archetype + alignment + color-treatment; no color-swap-only variants; none of the BCR §6 slop tells (center-everything, default Arial, gradient+drop-shadow, clip-art icon row) are present. | BCR §5, §6 |
| C5 | Print-spec compliance / upload fit (규격 정확) | 1× | Each card defines trim/bleed/safe in CSS; all text is contained inside the safe zone; every contact field is present on every card; 20/20 render cleanly; geometry is in `mm`, never `px`. | BCR §1.2–1.3, §8 |

---

## IMPORTANT — why C1 and C2 are weighted 2× (style-magnet-safe wording)

> C1 타이포 위계 and C2 여백 균형 are weighted 2×. Claude already performs C3 color coherence,
> C4 rule-based variety, and C5 spec compliance adequately by default — those are mechanical,
> deterministic, or rule-traversable. What collapses without pressure is **real typographic
> hierarchy** (Claude flattens every text line to the same weight and mixes alignments) and
> **intentional whitespace distribution** (Claude center-fills space and treats voids as waste).
> These two axes decide whether a business card looks designed or templated.

Write criteria as **qualities** (hierarchy, balance, coherence, variety, compliance), never as
**reference names**. Phrases like "MOO-like" or "museum-quality" are a style magnet — they push
every card toward one convergence. Brand/studio reference names (MOO, Pentagram, Base Design)
belong ONLY in `spec.md` design intent, never in this rubric or in any score justification.

---

## Verdict logic (binary, per V1-17)

```
All criteria ≥ 4  AND  all adversarial probes clean  AND  human visual checkpoint = APPROVED
                                                                          → PASS
Any 2×-weighted criterion (C1, C2) < 4                                    → FAIL
Any 1×-weighted criterion (C3, C4, C5) < 3                                → FAIL
Any Definition-of-Done item from spec.md unverified                       → FAIL
Human visual checkpoint NOT yet APPROVED on a visual-aesthetic axis       → FAIL
    (P-1: the Evaluator may NOT self-pass C1/C2 aesthetic judgment or C4
     aesthetic judgment without the orchestrator's recorded human approval)
```

A surface-level completion that does not achieve the deep quality is a FAIL, not a partial pass.
Examples specific to this domain:
- A card with three "tiers" that are all the same size/weight → C1 is 1–2, not "close enough".
- A gallery whose overlay toggle exists in markup but does not actually flip the guides → C5 FAIL.
- 20 cards that are the same centered layout with only hex values swapped → C4 is 1, FAIL.

---

## Scoring guide

| Score | Meaning |
|-------|---------|
| 5 | Exceeds expectations — a picky senior graphic designer would be impressed. |
| 4 | Meets expectations — solid, professional, upload-ready. |
| 3 | Acceptable but noticeably weak — needs another iteration. |
| 2 | Below expectations — significant craft gaps. |
| 1 | Unacceptable — fundamental problems (templated, off-spec, broken). |

The detailed 1/3/5 anchors per criterion live in `evaluator-calibration.md` and are pasted into
the Evaluator prompt. Use them to keep scoring stable across iterations.

---

## Calibration checkpoint (dual lens, per V1 calibration §85–91)

If every category scores ≥ 4, re-evaluate through TWO lenses before approving:

1. **Picky senior graphic designer lens** — "What would a typographer catch? Is the name actually
   kerned? Is there one alignment system, or did contact info quietly drift to a second alignment?
   Is the asymmetry intentional or accidental? Could I delete a card and lose no variety?"
2. **Print-shop prepress inspector lens** — "Does the background actually reach the bleed edge, or
   is there a white sliver risk? Is the longest realistic email/phone string still inside safe?
   Is geometry in `mm`? Does `@page` match the working canvas with `margin:0`? Will the overlay
   guides hide under `@media print`?"

Add any findings from these lenses to `critique.md`. Do not pass on "it looks fine."
