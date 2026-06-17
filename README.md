# Business Card Design Harness

> Generate a batch of distinctive, print-spec, **anti-slop HTML business-card mockups** (default 20)
> plus a one-page **visual-companion gallery** — built by a three-agent
> **Planner → Generator → Evaluator** harness with a mandatory human visual checkpoint.

**Languages:** English (this file) · [한국어](./README.ko.md)

A [Claude Code](https://claude.com/claude-code) skill. It turns a short brief (brand, contact fields,
tone, palette) into 20 single-file HTML business cards — each a *different* layout archetype, sized to
real online 명함 (business-card) print services — and a gallery to review them in. Visual taste is
never self-certified by the model; a human approves the aesthetics before anything is called done.

---

## Table of contents

- [Why this exists](#why-this-exists)
- [How it works (the harness)](#how-it-works-the-harness)
- [What you get](#what-you-get)
- [Quick start](#quick-start)
- [The activation flow](#the-activation-flow)
- [Quality model (rubric)](#quality-model-rubric)
- [Three human gates](#three-human-gates)
- [Scope boundary (RGB vs CMYK)](#scope-boundary-rgb-vs-cmyk)
- [Output stages (HTML → print formats)](#output-stages-html--print-formats)
- [Files in this skill](#files-in-this-skill)
- [The baked-in design reference (BCR)](#the-baked-in-design-reference-bcr)
- [Article fidelity](#article-fidelity)
- [Installation](#installation)
- [Limitations](#limitations)
- [Credits](#credits)

---

## Why this exists

Ask a general LLM for "20 business card designs" and you typically get **one layout in 20 colors**:
everything centered, default Arial, a gradient with a drop shadow, clip-art phone/email icons. That is
*slop* — templated output that no designer would ship. Two specific failure modes drive it:

1. **Self-evaluation bias.** A single session that both generates *and* judges its own work
   confidently praises mediocre output. Generation and evaluation must be **separate processes**.
2. **Sensory blindness.** An LLM cannot actually *see* typographic balance or whitespace. If it claims
   it "looks great," it is guessing. Visual aesthetics need a **human eye in the loop**.

This skill encodes the [*Harness Design for Long-Running Application Development*](https://www.anthropic.com/engineering/harness-design-long-running-apps)
pattern (Anthropic, 2026) to defeat both: a GAN-style **role split** (separate `Agent` calls that
communicate only through files) and a **mandatory human visual checkpoint** that the Evaluator cannot
bypass.

---

## How it works (the harness)

Three roles, dispatched by the orchestrator as **separate `Agent` calls**. They never see each other's
reasoning — they communicate **only through files**.

```
                    ┌─────────────────────────────────────────────┐
   brief ──▶ INTAKE │ brand · contact fields · tone · palette · preset · quantity │
                    └───────────────┬─────────────────────────────┘
                                    │  (+ live-research consent gate)
                                    ▼
                 ┌──────────┐   spec.md    ┌────────────┐
                 │ PLANNER  │ ───────────▶ │ GENERATOR  │  consumes baked-in reference (BCR)
                 │ "what"   │              │ "how"      │  20 distinct archetypes, mm geometry
                 └──────────┘              └─────┬──────┘  cards/*.html + gallery.html + card-spec.json
                       ▲                         │ READY_FOR_QA
                       │ sprint_contract.md      ▼
                       │ (negotiated)      ┌───────────────────────────┐
                       │                   │ ★ HUMAN VISUAL CHECKPOINT  │  open gallery.html, approve
                       │                   └─────────────┬─────────────┘
                       │                                 ▼
                 ┌──────────┐   critique.md       ┌────────────┐
                 │GENERATOR │ ◀───── FAIL ──────── │ EVALUATOR  │  8 adversarial probes + rubric
                 │ (REFINE/ │                      │ "is it real"│ may NOT self-pass aesthetics
                 │  PIVOT)  │ ─────── PASS ──────▶  └────────────┘
                 └──────────┘                            │
                                                         ▼
                                              package deliverable folder
                                          (optional Stage-2 format conversion)
```

- **Planner** decides *what* each card must contain (fields, 3-tier hierarchy, preset, the variety
  bar) and freezes it in `spec.md`. It never prescribes CSS.
- **Generator** decides *how* — picks 20 distinct archetypes from the reference catalog, writes the
  HTML/CSS in millimetre geometry, builds the gallery, and self-verifies before handoff.
- **Evaluator** is the adversary: 8 binary probes + a 5-criterion rubric. It **cannot** pass the
  visual-aesthetic axes without the recorded human approval.

Tier = **Full**: sprint-contract negotiation + per-sprint Evaluator passes + file-based handoffs.

---

## What you get

The run packages into a single **deliverable folder**:

```
<deliverable>/
├─ README.md          ← team handoff: structure, how to review, print-responsibility, status
├─ gallery.html       ← review entry point: tiles all cards + bleed/trim/safe overlay toggle
├─ card-spec.json     ← preset + per-card archetype map + contact-field map + sRGB/CMYK advisory
├─ cards/             ← card-01.html … card-NN.html (single-file, upload/convert targets)
├─ docs/              ← spec.md, generator_report.md, critique.md
├─ preview/           ← screenshots (gallery + representative cards)
└─ _archive/          ← superseded concepts, kept out of the live set
```

Each card is a **single self-contained HTML file** (inline CSS + web fonts), a *distinct* layout
archetype, geometry in `mm`, with `trim` / `bleed` / `safe` defined in CSS. The gallery tiles them on a
neutral gray ground and has one global overlay toggle: **green = bleed, magenta = trim, cyan = safe**
(hidden under `@media print`).

---

## Quick start

This is a Claude Code skill. Once installed (see [Installation](#installation)), just describe the job:

```
명함 디자인 만들어줘 — Withwiz, 김도윤 Founder & CEO, 브랜드 블루 #2D5BFF
```

**Trigger phrases**

- **EN:** `business card`, `business card design`, `card design`, `business card mockups`,
  `HTML business card`, `name card design`, `business card gallery`, `design business cards for print`
- **KO:** `명함 디자인`, `명함 시안`, `명함 시안 20개`, `명함 HTML 만들어줘`,
  `온라인 명함 업로드용 시안`, `비주얼 컴패니언 검토`, `명함 갤러리 생성`, `명함 디자인 하네스`

The skill runs intake first — it will not generate until it has your brand, **every** contact field,
tone/industry, palette, preset, and quantity. (Empty input would yield placeholder cards, which fail
the placeholder sweep.)

---

## The activation flow

| # | Step | Gate |
|---|------|------|
| 1 | **Intake** — brand, all contact fields, tone, palette (hex), preset, quantity | |
| 2 | **Live-research gate** — optionally research current trends; graceful fallback to static reference | 🔵 user consent |
| 3 | **Planner** → `spec.md` (the only file the Generator reads) | |
| 4 | **Sprint-contract negotiation** — Generator proposes observable checks, Evaluator strengthens & approves | |
| 5 | **Generator** → 20 `card-NN.html` + `gallery.html` + `card-spec.json`, self-verified | |
| 6 | **Gallery render** — orchestrator renders; individual cards screenshotted as a sanity check | |
| 7 | **★ HUMAN VISUAL CHECKPOINT** — open `gallery.html`, toggle overlays, approve | 🔴 **mandatory** |
| 8 | **Evaluator** → `critique.md` (PASS/FAIL): 8 probes + rubric; can't self-pass aesthetics | |
| 9 | **Iterate** on FAIL — Generator makes a Strategic Decision (REFINE / PIVOT / ESCALATE). Cap 5–15, default 8 | |
| 10 | **Package** the deliverable folder (cards/ docs/ preview/ _archive/ + README) | |
| 11 | **Final-design selection** — the N cards are *concepts*; the user picks which become the printed card (+ optional sub-variant compare loop) | 🔵 user choice |
| 12 | **Stage-2 conversion** (optional) — convert only the selected design(s) to PDF/JPG | 🔵 user choice |
| 13 | **Upload-confirmation gate** — declare "ready to upload" only on explicit approval | 🔴 user consent |

---

## Quality model (rubric)

The Evaluator scores every batch on **5 criteria, 1–5**. Two axes carry **2× weight** — the ones where
a general LLM is weakest by default:

| # | Criterion | Weight | What it checks |
|---|-----------|:------:|----------------|
| **C1** | Typographic hierarchy | **2×** | Real 3-tier name/title/contact hierarchy; one committed alignment per card; ≤2 type families; kerned name |
| **C2** | Whitespace balance | **2×** | Content off the trim; deliberate visual weight; one protected focal point; intentional asymmetry (not center-fill) |
| C3 | Brand color coherence | 1× | 1–2 brand + 1–2 neutrals applied consistently; small-text contrast ≥ 4.5:1; brand color as a *sparing* accent |
| C4 | Design variety / anti-slop | 1× | 20 distinct archetypes; **no** archetype+alignment+color-treatment duplicates; **no** color-swap-only variants; no slop tells |
| C5 | Print-spec compliance | 1× | trim/bleed/safe in CSS; all text inside safe; every field on every card; 20/20 render; geometry in `mm` not `px` |

**Verdict logic (binary):** all criteria ≥ 4 **and** all probes clean **and** human checkpoint
APPROVED → **PASS**. Any 2× criterion < 4, or any 1× criterion < 3, or any unverified
Definition-of-Done item → **FAIL**. The model **cannot** self-pass C1/C2/C4 aesthetic judgment without
the recorded human approval.

Why 2×? C3/C4/C5 are mechanical or rule-traversable — Claude does them adequately. What collapses
without pressure is *real* typographic hierarchy (every line flattened to one weight, alignments mixed)
and *intentional* whitespace (space center-filled, voids treated as waste). Those two axes decide
whether a card looks **designed** or **templated**.

---

## Three human gates

The skill refuses to be fully autonomous on the things a human must own:

1. 🔵 **Live-research consent** (step 2) — whether to hit the web at all.
2. 🔴 **Human visual checkpoint** (step 7) — *mandatory*. You open `gallery.html`, review the
   aesthetics in a real browser, and approve. The Evaluator's probe 6 **FAILS the whole verdict** if any
   aesthetic axis was scored ≥ 4 without your recorded approval.
3. 🔴 **Upload confirmation** (step 13) — "ready to upload" is never declared without your explicit OK.

---

## Scope boundary (RGB vs CMYK)

The visual companion is an **RGB emissive screen**. A printed card is **CMYK ink absorbed into paper** —
saturated screen blues/greens shift and dull in print, and paper stock changes them again. The gallery
**cannot** verify print color.

So the skill stays in scope: it emits an **sRGB recommendation** and states explicitly that **CMYK
conversion, physical proof (교정쇄), and print color accuracy are the print-shop / human's
responsibility.** It never claims a card is print-color-correct.

---

## Output stages (HTML → print formats)

- **Stage 1 (default deliverable): HTML.** `cards/*.html` + `gallery.html` — the review / handoff /
  source-of-truth. The skill completes Stage 1 and **stops**; it does not convert formats by default.
- **Stage 2 (optional, after design approval — user picks):** Korean online 명함 services
  (오프린트미 · 비즈하우스 · 레드프린팅 …) accept **PDF / AI / JPG, not HTML**, so reaching a print shop needs a
  conversion. The skill offers three, user-selected:
  1. **Print PDF** — per card, front/back as separate pages at trim+bleed, guides removed (인쇄소 입고).
  2. **High-res JPG** — per card per side (preview / SNS / service upload).
  3. **Combined PDF** — all cards in one file (team / print-shop review).

  Stage 2 never runs before the human checkpoint approval.

---

## Files in this skill

| File | Lines | Role |
|------|------:|------|
| `SKILL.md` | 277 | Orchestrator: activation flow, iteration wisdom, principles, tuning loop, V1/V2 guidance |
| `references/business-card-design-reference.md` | 457 | **Baked-in static research (BCR)** — specs, craft, the 22-archetype catalog, anti-slop tells, HTML/CSS patterns |
| `references/generator-prompt.md` | 215 | Generator role prompt (writes cards + gallery + card-spec.json) |
| `references/evaluator-prompt.md` | 190 | Evaluator role prompt — 8 adversarial probes incl. the human-gate probe |
| `references/planner-prompt.md` | 123 | Planner role prompt (writes `spec.md`) |
| `references/sprint-playbook.md` | 114 | Full-tier sprint decomposition + sprint-contract negotiation |
| `references/rubric.md` | 88 | 5 criteria, 2× weighting, verdict logic, dual-lens calibration |
| `references/evaluator-calibration.md` | 82 | Few-shot 1 / 3 / 5 score anchors per criterion C1–C5 |

Every role prompt is **self-contained** — a subagent dispatched with only its prompt file has
everything it needs.

---

## The baked-in design reference (BCR)

`references/business-card-design-reference.md` (457 lines) is curated research the Generator consumes
before writing any card. It is what keeps output off the slop floor without depending on the web:

| § | Section | Highlights |
|---|---------|------------|
| 1 | Standard specifications | Exact dimensions by region; bleed / trim / safe; mm-in-CSS rules; 300dpi context |
| 2 | Korean online 명함 services | Upload specs for Redprinting / Ohprint / Bizhows … (90×50mm trim, 2mm bleed, 3mm safe) |
| 3 | Globally notable services | MOO / Vistaprint / Canva / Adobe Express design ethos as taste anchors |
| 4 | Design principles — the craft | Type hierarchy ratios, whitespace, brand-color coherence, WCAG contrast |
| **5** | **Layout archetype catalog** | **22 named archetypes** + the variety recipe — the direct supply for 20 distinct cards |
| 6 | Anti-slop guidance | "Instead of X, do Y" pairs; the slop-tell blocklist |
| 7 | Designers / studios to emulate | Pentagram et al.; where to anchor "good" |
| 8 | HTML/CSS implementation | Print-spec mm card patterns, toggleable bleed/trim/safe overlays, the review gallery |

The 22-archetype catalog (§5) is what makes 20 mockups genuinely *different* — color-swap-only variants
are forbidden, and the Evaluator recomputes variety from `card-spec.json` to catch a Generator that lies
about it.

---

## Article fidelity

Every component maps to a principle from the source article and is verified by an auditor (35 checklist
rows + 8 adversarial probes, all PASS on the shipped skill):

- **GAN-style role split** — Planner / Generator / Evaluator are separate `Agent` calls; **file-based
  communication only**.
- **Rubric with few-shot anchors** — 1/3/5 concrete examples per criterion, not "looks good."
- **Iteration range 5–15, default 8** — *not* a single low cap; late breakthroughs are real (the
  article's "Dutch Art Museum" leap arrived at iteration ~10).
- **Strategic Decision on retry** — REFINE / PIVOT / ESCALATE; a pivot needs cited critique evidence,
  not context-reset amnesia.
- **Context reset ≠ compaction** — context anxiety is cleared by a fresh session + `handoff.md`, never
  by compaction.
- **Sensory-limit human checkpoint** — the gate the Evaluator cannot bypass.
- **Operational Evaluator-tuning loop** — numbered (a)–(d): read logs → find divergence → add
  counter-examples → rerun and confirm.
- **Every component encodes an assumption** — on a model upgrade, remove components one at a time
  (radical simplification fails); never remove the human checkpoint or the rubric.

A model-class table (Sonnet 4.5 / Opus 4.5 / Opus 4.6) tells you when you may collapse the Full tier
toward Simplified.

### Validated with a real run

A sample run (Withwiz, an AI startup; KR-standard preset; 20 cards) produced **20/20 distinct
archetypes, 20/20 unique archetype+alignment+color triples, 2 portrait + 7 one-bold-move cards, all 6
contact fields on every card, zero placeholders** — verified against the frozen `card-spec.json` schema.

---

## Installation

This skill lives in a Claude Code skills repository. To make Claude Code use it:

```bash
# from the repo root that contains business-card-design-harness/
ln -s "$(pwd)/business-card-design-harness" ~/.claude/skills/business-card-design-harness
```

Then start (or restart) Claude Code and use any trigger phrase above. No API keys, no external services
— the design reference is baked in, and live web research is optional and consented.

**Optional** — to package the skill for distribution:

```bash
python ~/.claude/skills/skill-creator/scripts/package_skill.py ./business-card-design-harness
```

---

## Limitations

- **Print color is out of scope.** RGB screen ≠ CMYK print; final color is the print shop's
  responsibility (see [Scope boundary](#scope-boundary-rgb-vs-cmyk)).
- **The model does not judge its own aesthetics.** A human must complete the visual checkpoint; the run
  cannot finish without it. This is a feature, not a gap.
- **HTML is an intermediate.** Korean print services want PDF/AI/JPG — Stage-2 conversion is required to
  actually order cards, and it is opt-in.
- **Headless full-page gallery screenshots under-render** off-screen cards (a lazy-paint capture
  artifact). Review in a *real* browser; screenshot individual cards for a sanity check.
- **The 20 cards are concepts, not 20 final cards.** You select which design(s) get printed.

---

## Credits

- Pattern: Anthropic — [*Harness Design for Long-Running Application Development*](https://www.anthropic.com/engineering/harness-design-long-running-apps)
  (Prithvi Rajasekaran, 2026) and *Building Effective Agents* — *"find the simplest solution possible,
  and only increase complexity when needed."*
- Built with the **skill-wizard-harness** meta-harness (the wizard applies the same
  Planner → Generator → Evaluator pattern to build skills).
- For [Claude Code](https://claude.com/claude-code).
