# Sprint Playbook — business-card-design-harness (Full tier)

This skill runs the **Full** V1 harness architecture: sprint-contract negotiation, per-sprint
Evaluator passes, and file-based communication only. This playbook is the operational guide for how
the orchestrator decomposes the 20-card batch into sprints, what each sprint's contract contains, and
how context resets are handled. It exists because the Full tier was chosen deliberately (spec §4):
the visual-aesthetic human gate plus the 20-card variety requirement mean per-sprint Evaluator passes
catch quality drift and slop repetition early, before they compound across all 20 cards.

---

## Why Full tier (not Simplified)

| Tier | When | This skill |
|------|------|-----------|
| Full (V1) | Long, drift-prone builds; weak-by-default axes need per-sprint pressure; sensory human gate | **CHOSEN.** 20 cards drift toward color-swap repetition late in the batch; the human visual gate must sit before the Evaluator's aesthetic verdict. |
| Simplified (V2) | Stronger model, shorter run, single end-of-run Evaluator acceptable | Not used. The sprint construct and per-sprint Evaluator are retained. |

If you later run this on Opus 4.5/4.6 (context anxiety largely eliminated), see SKILL.md
"V1 vs V2 guidance" — you may collapse the per-sprint Evaluator into a single end-of-run pass and
drop sprint decomposition, removing components ONE AT A TIME and re-testing (G-2). Do not remove the
human visual checkpoint (P-1) or the rubric in any tier.

---

## Sprint decomposition of a 20-card batch

20 cards is a natural unit to split. Decompose into 2 generation sprints + a finishing sprint so the
Evaluator catches variety/quality drift at the halfway mark, not only at the end.

| Sprint | Deliverables | Evaluator focus this sprint |
|--------|--------------|-----------------------------|
| **Sprint 0 — Contract** | `sprint_contract.md` only | Approve/strengthen the observable checks; lock the preset + the per-card check list. No cards yet. |
| **Sprint 1 — Cards 1–10** | card-01…card-10.html, partial card-spec.json | Probes 1–5,7 on cards 1–10; verify ≥1 portrait + ≥1 one-bold-move already present; flag any color-swap drift early. |
| **Sprint 2 — Cards 11–20** | card-11…card-20.html, completed card-spec.json | Probes 1–5,7 on cards 11–20; verify the FULL batch hits ≥2 portrait + ≥2 one-bold-move + 0 combo duplicates across all 20. |
| **Sprint 3 — Gallery + finish** | gallery.html, advisory, final self-verify | Probe 6 (human gate) + overlay-toggle + @media print + sRGB advisory (probe 8) + full-batch re-score. |

The Evaluator runs after each generation sprint. A FAIL in Sprint 1 is cheaper to fix than the same
slop tell repeated across all 20 cards. The human visual checkpoint (orchestrator step 7) happens
once the full gallery exists (after Sprint 3's gallery build) and BEFORE the Evaluator's final
aesthetic verdict.

For small models or tight context, split further (cards 1–5 / 6–10 / 11–15 / 16–20). For Opus
4.5/4.6, a single generation sprint of all 20 cards is sustainable — see V1 vs V2 guidance.

---

## Sprint contract — what `sprint_contract.md` must lock (V1-18)

The Generator proposes; the Evaluator strengthens and approves. The contract must list, concretely:

1. **Deliverables** for the sprint (exact file paths: cards/card-NN.html, gallery.html, card-spec.json).
2. **The preset in use** with exact mm numbers (KR-standard 90×50 trim / 2mm bleed / 3mm safe / 94×54
   working by default), so the Evaluator checks against the correct geometry.
3. **The observable checks** the Evaluator will run, each binary:
   - per-card trim/bleed/safe mm values match the preset and `@page` matches the working canvas with
     `margin:0`;
   - all text inside the safe rectangle, tested with the longest realistic strings;
   - background reaches the bleed edge (no off-cut white sliver);
   - 20/20 (or sprint-slice) render with no blank/overflow/unloaded font; `font-display` present;
   - every contact field present on every card;
   - archetype + alignment + color-treatment uniqueness across the batch; ≥2 portrait, ≥2 one-bold-move;
   - geometry in `mm`, never `px`;
   - gallery overlay toggle flips green/magenta/cyan guides and hides them under `@media print`;
   - placeholder sweep clean;
   - the sRGB / CMYK-human-responsibility advisory present.
4. **Out of scope** for the sprint (so the Evaluator does not fail on things deferred to a later sprint).

The Evaluator MUST reject a contract whose checks are weaker than spec.md's Definition of Done and
add the missing checks before generation begins.

---

## File handoff contract (file-based communication only, V1-19)

```
spec.md            (Planner → Generator)        the frozen design spec
sprint_contract.md (Generator ↔ Evaluator)      negotiated, Evaluator-approved checks
generator_report.md(Generator → Evaluator)      Strategic Decision + deliverables + self-verify
critique.md        (Evaluator → Generator)      verdict + scores + probe results + redirect
design_memo.md     (Generator → Evaluator)      PIVOT rationale, awaits Evaluator approval
handoff.md         (Generator → next Generator) context-reset handoff: done/remaining/used-archetypes
card-spec.json     (Generator → Evaluator)      preset + per-card archetype + contact-field map
```

Subagents never read each other's reasoning — only these files. The orchestrator routes the signals
(`SPEC_READY`, `READY_FOR_QA`, `CRITIQUE_READY`, `HANDOFF_NEEDED`, `DEADLOCK`).

---

## Context-reset policy across sprints (V1-21, V1-22)

- 20 cards is the natural split unit. When the Generator hits a context-anxiety signal (card depth
  dropping toward the tail, reaching for "the rest similar," skipping verification), it writes
  `handoff.md` and emits `HANDOFF_NEEDED: handoff.md` rather than compacting.
- `handoff.md` records: cards COMPLETE (e.g. card-01…card-12), cards REMAINING (card-13…20), and the
  LIST OF ARCHETYPES ALREADY USED — so the fresh Generator session preserves variety and assigns only
  unused archetypes.
- The orchestrator immediately starts a FRESH Generator session (a new `Agent` call) to continue.
  **Compaction preserves the context-anxiety state — it is not a reset. Only a fresh session clears
  it.** (V1-22.)

---

## Iteration discipline within and across sprints (V1-6, V1-7, V1-10)

- Iteration cap is a RANGE: 5–15, default **8**. Do not pick the low end. A tight cap blocks late
  breakthroughs.
- Late-leap wisdom (V1-10): in the Dutch Art Museum case the decisive jump arrived at iteration ~10.
  Business cards behave the same way — typographic hierarchy and intentional whitespace often only
  "click" at iteration 6+. Do not declare done before iteration 8 just because the cards render.
- Wall-clock tolerance ~4 hours (V1-7). Do not artificially rush; do not pad either.
- Middle iterations can beat the final (V1-11). If iteration N had a better card-07 hierarchy than the
  current one, the Evaluator's Iteration Quality Note must say so, and the Generator should recover it.
