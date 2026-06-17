# Evaluator Calibration — business-card-design-harness

Few-shot score anchors for the Evaluator (V1-5, V1-8). Each criterion C1–C5 has THREE anchors —
1/5 (unacceptable), 3/5 (acceptable but weak), 5/5 (excellent) — written as concrete, observable
business-card states. The Evaluator cites which anchor a card matches when it scores. These anchors
keep scoring stable across iterations and stop score drift.

The article found that "calibrating the evaluator using few-shot examples with detailed score
breakdowns ensured the evaluator's judgment aligned with my preferences, and reduced score drift
across iterations." Add new counter-example anchors here whenever a real miss is found (see
SKILL.md §"Evaluator tuning workflow").

---

## C1 — Typographic hierarchy (2×)

- **1/5:** Name, title, phone, email, address are all the SAME font, SAME size, SAME weight, set as a
  flat 5-line stack. Alignment is mixed (name centered, contact left-aligned). No tier reads as the
  anchor. Default system sans, no kerning, no tracking. — *This is the flatten-everything failure
  Claude defaults to.*
- **3/5:** The name is slightly larger than the rest, but title and contact are undifferentiated from
  each other (same size and weight). One font family, but no tracking on the title and no kerning on
  the name. A hierarchy exists but is weak and timid.
- **5/5:** A clear 3-tier system — e.g. name 13pt semibold, kerned; title 8pt uppercase with
  0.12em tracking; contact block 8pt regular on 1.45 line-height forming a clean rectangle. ONE
  committed alignment system (a single left rail). At most 2 families paired across a real contrast
  axis (serif name + sans contact, or one family in Light/Semibold). The eye lands on the name first.

## C2 — Whitespace balance (2×)

- **1/5:** Content fills all four corners; visible margin ~10%; text sits right up against the trim
  line (clip risk). No focal point — everything competes. The card feels crammed and tippy.
- **3/5:** The safe zone is respected, but every element is evenly distributed in a symmetric centered
  block. Safe, but bland and template-like — no intentional asymmetry, no protected void.
- **5/5:** Roughly 40–60% of the card is empty. Content is deliberately anchored to one corner or a
  single rail, with a large void opposite that protects a single focal point (name or logo). Visual
  weight is balanced (a heavy element offset by whitespace). The asymmetry reads as a decision.

## C3 — Brand color coherence (1×)

- **1/5:** Four or more saturated colors mixed; 8pt contact text is pale gray on white (contrast
  <3:1) and disappears. The palette has no logic.
- **3/5:** The palette is consistent (brand color + neutrals) but the brand color is overused as a
  wash behind small text, and the accent is not restrained — color appears in too many places to feel
  intentional.
- **5/5:** 1–2 brand colors + 1–2 neutrals. The brand color appears as a sparing accent in exactly one
  place (a rule line, the name, or a single icon). Small contact text contrast ≥4.5:1. The accent
  earns attention because it is rare.

## C4 — Design variety / anti-slop (1×)

- **1/5:** All 20 cards are the same centered layout with only hex values swapped (color-swap-only
  variants). Every card is default sans with a gradient + drop-shadow ground. Zero real variety.
- **3/5:** Only 5–6 archetypes are recycled across 20 cards, so clusters look alike. Zero portrait
  cards and zero "one bold move" cards. No outright slop tells, but the batch reads repetitive.
- **5/5:** 20 distinct archetypes from BCR §5, ≥2 portrait, ≥2 one-bold-move, zero
  archetype+alignment+color-treatment duplicates, and no BCR §6 slop tells anywhere. A human scanning
  the gallery sees 20 genuinely different cards.

## C5 — Print-spec compliance / upload fit (1×)

- **1/5:** Card geometry is specified in `px` (px-creep). Bleed/trim/safe are not defined. Text
  crosses the trim line. Only 19/20 cards render (one is blank or font-broken).
- **3/5:** Trim/bleed/safe are defined and geometry is in `mm`, but one preset dimension is off by 1mm,
  OR one card is missing one contact field. Close, but not upload-clean.
- **5/5:** 20/20 cards render. Exact mm geometry (KR-standard: 94×54 working / 90×50 trim / 2mm bleed
  / 3mm safe, `@page` matches with margin:0). All text inside safe with the longest realistic strings.
  Zero contact fields missing. The gallery overlay toggle actually flips the green/magenta/cyan guides
  and hides them under `@media print`. card-spec.json records preset + per-card archetype + field map.

---

## How the Evaluator uses these anchors

1. For each card and each criterion, find the closest anchor and cite it (e.g. "card-09 C1 ≈ 3/5
   anchor: name larger but title/contact undifferentiated, no kerning").
2. The batch score for a criterion is the worst representative state across the 20 cards for that axis
   (one color-swap pair caps C4; one px-geometry card caps C5 at FAIL).
3. Remember the 2× weighting: C1 or C2 below 4 is an automatic FAIL regardless of the other scores.
4. Aesthetic portions of C1/C2/C4 still require the recorded human visual-checkpoint approval
   (evaluator-prompt.md probe 6) before they may pass — these anchors describe the target, but a human
   confirms the aesthetic verdict.
