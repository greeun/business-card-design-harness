---
name: business-card-design-harness
description: Generate 20 upload-ready HTML business-card mockups (single-file, inline CSS, web fonts) spanning distinct layout archetypes, plus a one-page visual-companion gallery with toggleable bleed/trim/safe overlays for human review, sized to online 명함 print services (KR default 90×50mm trim + 2mm bleed + 3mm safe / 94×54mm working canvas; US 3.5×2in and EU 85×55mm presets). Runs as a Planner→Generator→Evaluator harness with anti-slop variety enforcement, spec-compliance probes, and a mandatory human visual-aesthetic checkpoint gate. Trigger phrases — KO "명함 디자인", "명함 시안", "명함 시안 20개", "명함 HTML 만들어줘", "온라인 명함 업로드용 시안", "비주얼 컴패니언 검토", "명함 갤러리 생성", "명함 디자인 하네스". EN "business card", "business card design", "card design", "business card mockups", "HTML business card", "name card design", "business card gallery", "design business cards for print".
version: 1.0.0
---

# Business Card Design Harness

Generate a batch of distinctive, print-spec, anti-slop **HTML business-card mockups** (default 20)
for upload to online 명함 services, plus a one-page **visual-companion gallery** with toggleable
bleed/trim/safe overlays for human review. The skill runs as a three-agent harness
(Planner → Generator → Evaluator) the orchestrator dispatches as separate `Agent` calls.

The main session does **intake, the human gates, and final packaging only**. It never writes cards
itself — it collects the brief, runs the safety gates, dispatches the subagents, routes their file
signals, and assembles the deliverable folder (README + docs/preview/_archive) at the end.

## What gets produced

The run packages into a single **deliverable folder**. During the run the subagents write their
working files flat at the run root (spec.md, generator_report.md, critique.md, cards/, gallery.html,
card-spec.json); the orchestrator assembles the structure below at the END (packaging step, after the
Evaluator PASS) — do NOT make the subagents manage this layout:

```
<deliverable>/
├─ README.md            ← team handoff: structure + 검토법 + 인쇄 책임 고지 + 연락처 맵 + 상태
├─ gallery.html         ← review entry point (tiles all cards, bleed/trim/safe overlay toggle)
├─ card-spec.json       ← preset + per-card archetype map + contact-field map + sRGB/CMYK advisory
├─ cards/               ← card-01.html … card-NN.html (upload/convert targets — HTML only, no json)
├─ docs/                ← spec.md, generator_report.md, critique.md
├─ preview/             ← screenshots (gallery + representative cards)
└─ _archive/            ← any prior/superseded concepts, kept out of the live set
```

- `cards/card-01.html` … — single-file print-spec cards (inline/embedded CSS + web fonts), each a
  DISTINCT layout archetype, geometry in `mm`, trim/bleed/safe defined.
- `gallery.html` — tiles all N on a neutral gray ground (#f2f2f0), figcaption = archetype + size, with
  a global bleed/trim/safe overlay toggle (green=bleed, magenta=trim, cyan=safe) that hides under
  `@media print`.
- `card-spec.json` — records the preset, the per-card archetype map, and the contact-field map. Lives
  at the deliverable root, NOT inside `cards/` (cards/ holds only upload targets).
- `README.md` — generated at packaging time so a teammate opening the folder cold knows what each part
  is, how to review, the print-responsibility advisory, the contact-field map, and deliverable status.

### Output stages (산출물 포맷)

- **Stage 1 (primary — this skill's default deliverable): HTML.** `cards/*.html` + `gallery.html`. The
  review / handoff / source-of-truth artifacts. The skill completes Stage 1 and STOPS for the user; it
  does not convert formats by default.
- **Stage 2 (optional, AFTER design approval — user picks which):** convert the approved HTML to a
  print/share format. Offer three, user-selected (not all by default):
  1. **인쇄용 PDF** — per card, front/back as separate pages at trim+bleed (e.g. 92×52mm for a 90×50
     card), overlay guides removed — for 인쇄소 입고.
  2. **고해상 JPG** — per card per side — preview / SNS / service upload.
  3. **통합 1개 PDF** — all N cards in one file — team / print-shop review.
  KR online 명함 services (오프린트미·비즈하우스·레드프린팅 …) accept PDF/AI/JPG, not HTML — so Stage-1
  HTML is an intermediate, and reaching a print shop requires a Stage-2 conversion. Never run Stage 2
  before the human-checkpoint approval (don't render formats of cards that may still be revised).

## Scope boundary (sensory limit)

Visual aesthetics — typographic hierarchy, whitespace, color harmony — are sensory judgments an LLM
cannot self-certify by claiming it "saw" the output. They pass ONLY through the human visual
checkpoint (step 7). Print color is also out of scope: the companion is an RGB emissive screen and
cannot verify CMYK. The skill emits an sRGB recommendation and states explicitly that **CMYK
conversion, physical proof (교정쇄), and print color accuracy are the print-shop/human's
responsibility.** It never declares "ready to upload" without explicit user confirmation.

---

## Files in this skill

| File | Role |
|------|------|
| `references/business-card-design-reference.md` | BAKED-IN static research (BCR). Exact print specs, craft principles, the 22-entry archetype catalog (BCR §5), anti-slop tells (BCR §6), HTML/CSS patterns (BCR §8). The Generator's primary supply. |
| `references/planner-prompt.md` | Planner role prompt (writes `spec.md`). |
| `references/generator-prompt.md` | Generator role prompt (writes the cards + gallery + card-spec.json). |
| `references/evaluator-prompt.md` | Evaluator role prompt (8 adversarial probes incl. the human-gate probe). |
| `references/evaluator-calibration.md` | Few-shot 1/3/5 score anchors per criterion C1–C5. |
| `references/rubric.md` | 5 criteria, 2× weighting, verdict logic, calibration checkpoint. |
| `references/sprint-playbook.md` | Full-tier sprint decomposition + sprint-contract negotiation. |

---

## Activation flow

Dispatch each role as a SEPARATE `Agent` call (V1-1), passing that role's prompt file as the agent's
instructions. The subagents communicate ONLY through files (V1-19) — they never see each other's
reasoning.

1. **Intake collection.** Confirm with the user: (a) brand/company name; (b) the FULL set of contact
   fields (name, title, phone, email, company, address, website — whatever they have, name each one);
   (c) tone/industry; (d) brand color palette as hex (1–2 brand + 1–2 neutrals; if none given, propose
   a neutral monochrome + single accent default); (e) preset (KR-standard default / KR-global / US /
   EU); (f) card quantity (default 20). **Do not generate before intake is complete** — empty input
   would produce placeholder cards (Your Name / Lorem) that FAIL the placeholder-sweep probe.

2. **Live-research gate (safety gate 1, user consent).** Ask: "런타임 웹 리서치로 최신 트렌드/업종
   레퍼런스를 조사할까요?" If yes AND the web is available → flag ON. If declined or the web is
   unavailable → flag OFF, gracefully fall back to the static BCR document. The skill must never
   DEPEND on the web.

3. **Planner dispatch.** Give the Planner `references/planner-prompt.md` + the intake. It converts the
   brief into `spec.md` (the only file the Generator reads). Wait for `SPEC_READY: spec.md`.

4. **Sprint-contract negotiation.** The Generator proposes `sprint_contract.md` (deliverables + the
   exact observable checks + the preset's mm numbers); the Evaluator reviews, strengthens any weak
   checks, and approves (V1-18). See `references/sprint-playbook.md` for the sprint decomposition.

5. **Generator dispatch (generate).** Give the Generator `references/generator-prompt.md`. It consumes
   the baked-in BCR, selects 20 distinct archetypes (variety recipe: ≥2 portrait, ≥2 one-bold-move, no
   archetype+alignment+color duplicate), optionally runs live research, generates the 20 card-NN.html
   in `mm` units with trim/bleed/safe CSS, writes `card-spec.json`, self-verifies, and emits
   `READY_FOR_QA`.

6. **Gallery render.** Ensure `gallery.html` exists tiling all 20 with the overlay toggle. The
   orchestrator renders it in a browser (capture a screenshot if possible). **Headless caveat:** a
   full-page headless screenshot of the gallery often under-renders off-screen cards — eager-loaded
   `<iframe>` cards paint lazily during scroll-capture, so the shot shows only the first row and the
   rest read as blank tiles. This is a capture artifact, NOT broken cards. Confirm by also
   screenshotting a few INDIVIDUAL `cards/card-NN.html` files (they render fully standalone), and have
   the human review in a REAL browser (`open gallery.html`), never the headless full-page shot.

7. **★ HUMAN VISUAL CHECKPOINT (safety gate 2, P-1) — STOP HERE.** Ask the user to OPEN `gallery.html`
   in a browser, review it visually (toggle the bleed/trim/safe overlays), and approve. **Do not
   proceed to the Evaluator's aesthetic verdict until the user has confirmed.** Visual-aesthetic axes
   (typographic hierarchy, whitespace, variety) require a human eye — the Evaluator may NOT self-pass
   them. Pass the user's recorded approval signal into the Evaluator as the gate input. This gate is
   mandatory and cannot be replaced by "the Evaluator will handle it."

8. **Evaluator dispatch (evaluate).** Give the Evaluator `references/evaluator-prompt.md`,
   `references/rubric.md`, and `references/evaluator-calibration.md`. It runs every sprint-contract
   check + all 8 adversarial probes (including probe 6, the human-gate probe). If any aesthetic axis
   was scored ≥4 WITHOUT the recorded human approval from step 7 → probe 6 FAILS → whole verdict FAIL.
   Wait for `CRITIQUE_READY: critique.md` (PASS/FAIL).

9. **Iterate.** On FAIL, the Generator opens `generator_report.md` with a Strategic Decision
   (REFINE / PIVOT / ESCALATE) and runs the next round. Repeat steps 5–8. **Iteration cap is a RANGE
   of 5–15, default 8** — do not pick the low end (V1-6, V1-10).

10. **Package the deliverable folder.** On PASS, the orchestrator assembles the structure from "What
    gets produced": create `cards/` (card HTML only), move `spec.md` + `generator_report.md` +
    `critique.md` into `docs/`, screenshots into `preview/`, any prior/superseded concepts into
    `_archive/`, keep `gallery.html` + `card-spec.json` at the root, and WRITE `README.md` (structure +
    검토법 + 인쇄 책임 고지 + 연락처 맵 + deliverable status). This is Stage-1 (HTML) complete.

11. **Final-design selection (+ optional sub-variant comparison loop).** The N cards are CONCEPTS, not
    N final cards — after PASS the user SELECTS which design(s) actually become the printed card.
    Stage-2 converts ONLY the selected design(s), never the whole batch by default. Present the
    selection in a REAL browser (open `gallery.html` or a focused comparison page) — screenshots are an
    orchestrator sanity-check, NOT the user's review surface; show the actual UI. If the user likes a
    direction but wants one element changed (a motif, a phrase, the script — e.g. "good direction, but
    swap the unfamiliar 한자 for 한글"), run a focused SUB-VARIANT round: the Generator writes the variants
    as NEW files (e.g. `card-16a.html` / `-16b` / `-16c`) WITHOUT overwriting the approved originals,
    and the orchestrator builds a small comparison page that tiles the variants WITH the
    bleed/trim/safe guide toggle (reuse the gallery's `postMessage({rionGuides})` mechanism) and opens
    it in the browser. Loop until the user picks the final.

12. **Stage-2 format conversion (on the SELECTED design only, optional, user-picked).** Once a final
    design is chosen, offer the optional Stage-2 conversions (인쇄용 PDF 앞/뒤 분리 · 고해상 JPG · 통합 PDF —
    see "Output stages"). Convert ONLY the formats the user picks, for the SELECTED card(s) only. The
    skill's default deliverable is Stage-1 HTML; Stage 2 is opt-in.

13. **Upload-confirmation gate (safety gate 3, user consent).** Only with the user's explicit approval
    declare "업로드 준비 완료." Attach the sRGB recommendation and the "CMYK / physical proof / print color
    accuracy = print-shop/human responsibility" advisory. Never start an upload without explicit user
    consent.

**Three user-confirmation gates require an explicit user response:** (2) live-research consent,
(7) human visual checkpoint, (13) upload confirmation. Final-design selection (11) and Stage-2 (12)
are further user-driven choices.

---

## Iteration wisdom

- **Cap is a range: 5–15, default 8 (V1-6).** Document the range, not a single value, and do not pick
  the low end.
- **Don't pick the low end — late leaps are real (V1-10).** In the Dutch Art Museum case the decisive
  quality jump arrived around iteration 10. Business cards behave the same: typographic hierarchy and
  intentional whitespace frequently only "click" at iteration 6+. Do not declare done before iteration
  8 just because the cards render — rendering is not quality.
- **Wall-clock tolerance ~4 hours (V1-7).** A coherent batch of 20 distinctive cards plus iteration
  takes time. Do not artificially rush; do not pad.
- **Middle iterations can beat the final (V1-11).** If an earlier iteration had a stronger card-07
  hierarchy than the current one, the Evaluator's Iteration Quality Note says so and the Generator
  recovers it. The last iteration is not automatically the best.

---

## Evaluator tuning workflow

An untuned Evaluator is too lenient — its first runs are drafts. Calibrate it with few-shot examples
(V1-8, V1-20, G-4, P-3). Run these NUMBERED operational steps; skipping any one is an audit failure:

(a) **Read the critique logs.** After a full Planner→Generator→Evaluator cycle on a known input, read
    `critique.md` alongside the actual cards. For every rubric score ask: *would a picky senior
    graphic designer + a prepress inspector have scored this card the same?*

(b) **Identify divergence patterns.** Name the specific places the Evaluator and a human expert
    disagree. Typical misses on this domain: passing a color-swap-only batch as "varied" (C4),
    accepting a flattened 5-line stack as "hierarchy" (C1), trusting that a gallery toggle works
    without confirming it flips the guides (C5), self-passing an aesthetic axis without the human
    checkpoint (probe 6).

(c) **Update the evaluator prompt + calibration with concrete counter-examples.** Add a new 1/3/5
    anchor to `references/evaluator-calibration.md` describing the exact missed state (e.g. "1/5 C4:
    cards 14–20 are card-03's layout with only --brand hex changed"), and tighten the matching probe
    wording in `references/evaluator-prompt.md`. Do NOT loosen the verdict logic — add counter-examples.

(d) **Rerun on the same input and confirm the prior miss is now caught.** Re-run the cycle on the
    identical brief and verify the Evaluator now FAILS the previously-passed defect, with reproducible
    evidence. If it still passes, return to (b). Stop tuning when Evaluator verdicts correlate with a
    careful human expert pass and every blocking issue it raises is reproducible.

---

## Principles

- **Every component encodes an assumption (G-1).** The sprint construct assumes context anxiety; the
  2× weighting assumes Claude flattens hierarchy; the human gate assumes the LLM cannot self-certify
  aesthetics. When you upgrade the target model, remove components ONE AT A TIME and re-test — do not
  strip several at once (G-2: radical simplification failed in the source article).
- **Context reset ≠ compaction (V1-22).** When the Generator hits a context-anxiety signal (card depth
  dropping toward the tail of the batch, reaching for "the rest similar," skipping verification) it
  writes `handoff.md` (done cards / remaining cards / archetypes already used) and emits
  `HANDOFF_NEEDED`. The orchestrator starts a FRESH Generator session. **Compaction preserves the
  anxiety state — only a fresh session clears it.**
- **Smallest harness that works (G-3).** From Anthropic's *"Building Effective Agents"*: *"find the
  simplest solution possible, and only increase complexity when needed."* This skill uses Full tier
  deliberately (the human gate + 20-card variety justify per-sprint Evaluator passes), not by reflex.
- **File-based communication only (V1-19).** Subagents exchange files, never reasoning. The
  handoff contract lives in `references/sprint-playbook.md`.
- **Encode brand meaning in the audience's familiar, on-trend script (R-1).** A meaning mark only
  "transmits" if the viewer can read it. For a Korean audience, a hanja glyph the brand's etymology
  happens to use (e.g. 利溫) reads as unfamiliar and dated when set as the hero mark — carry the meaning
  in 한글 (the brand's meaning phrase) or an abstract on-trend motif instead. The Planner states the
  etymology AND flags which source-language glyphs must NOT become the hero mark (planner-prompt §3).
- **The browser is the review surface, not the screenshot (R-2).** The human checkpoint (step 7) and
  final selection (step 11) happen in a REAL browser — open the gallery / a comparison page. Orchestrator
  screenshots are a sanity check only (and a full-page headless gallery shot under-renders off-screen
  iframes — step 6 caveat); they never replace the user seeing the live UI. For sub-selection between
  variants, build a comparison page with the bleed/trim/safe guide toggle and open it in the browser.
- **The harness shifts the space of outcomes, it does not shrink it (G-5).** Good harness design
  changes WHICH 20 cards get made (distinctive, spec-clean) — it does not merely reduce variance
  toward a safe mean. Keep ambition high; the rubric and probes catch the misses.

---

## V1 vs V2 guidance (tier = Full)

This skill ships at **Full** tier: sprint-contract negotiation + per-sprint Evaluator passes +
file-based communication are all retained. The sprint construct and per-sprint Evaluator are NOT
removed (that would be the Simplified V2 architecture, which this skill does not use).

Whether to simplify toward V2 depends on the model you run the harness on. Context anxiety — the
tendency to wrap up prematurely — is the main thing the Full structure defends against, and newer
models have largely eliminated it:

| Model class | Context anxiety | Recommended tier | Notes |
|-------------|-----------------|------------------|-------|
| Sonnet 4.5 | Strong — wraps up prematurely | Full (V1) | Small sprints, aggressive Evaluator, firm context resets |
| Opus 4.5 | Largely eliminated | Simplified (V2) | Multi-hour coherent sessions; sprint decomposition droppable |
| Opus 4.6 | Eliminated; improved planning, long-context, debugging | Simplified or Single-session | 2+ hour builds sustainable; re-examine every component, drop what's not load-bearing |

- **On Sonnet 4.5**, keep the full structure: split the 20 cards into small sprints (1–10 / 11–20 or
  finer), run the Evaluator after each, and reset context aggressively via `handoff.md`.
- **On Opus 4.5+**, you may generate all 20 cards in a single sprint and collapse the per-sprint
  Evaluator into one end-of-run pass (V2-2, V2-3 — Evaluator cost is not a fixed yes/no; weigh it).
  Remove components one at a time and re-test (G-2).
- **Regardless of model, never remove:** the human visual checkpoint (P-1, step 7), the rubric with
  its 2× weighting, or the print-spec probes. These guard sensory and deterministic quality that no
  model upgrade addresses.

**Closing guidance (G-5):** re-examine every component when the target model improves, and drop what
is no longer load-bearing — but the goal is to keep shifting outcomes toward distinctive, spec-clean
cards, not to shrink the harness for its own sake.
