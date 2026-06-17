# Business Card Design Reference

Curated reference knowledge for generating distinctive, print-spec, anti-slop business-card
mockups in HTML/CSS. All numbers are concrete and usable. Where a spec could not be verified
against a primary source it is marked "(verify)". Sources cited inline as [Source: name/url].

---

## 1. Standard Specifications (exact numbers)

### 1.1 Dimensions by region (TRIM / finished size)

| Region | Inches | Millimeters | Aspect ratio | Notes |
|---|---|---|---|---|
| US / Canada | 3.5 × 2.0 in | 88.9 × 50.8 mm | 1.75 : 1 | The global default; ~1.75 ratio. [Source: PrintPlace, https://www.printplace.com/articles/standard-business-card-sizes] |
| Europe / UK / most of Asia | 3.35 × 2.17 in | 85 × 55 mm | 1.545 : 1 | ISO-ish "credit-card-adjacent" size; fits wallets. [Source: papersizechart, https://papersizechart.com/business-card-sizes/] |
| Korea (KR) — standard | 3.54 × 1.97 in | **90 × 50 mm** | 1.8 : 1 | Most common domestic 명함 size; finance/law/medical/government default. [Source: Ohprint.me, https://www.ohprint.me/blog/business-card-size-guide-90x50-85x55] |
| Korea (KR) — global | 3.35 × 2.17 in | 85 × 55 mm | 1.545 : 1 | Preferred by fashion/design/IT/startups & global-facing firms; credit-card proportion. [Source: Ohprint.me] |
| Japan (meishi 名刺) | 3.58 × 2.17 in | **91 × 55 mm** | 1.655 : 1 | Slightly wider than EU to fit Japanese characters alongside English. [Source: Japan Dev, https://japan-dev.com/blog/business-cards-in-japan; Gloture, https://blog.gloture.co.jp/how-to-make-the-perfect-japanese-business-card/] |
| Square | 2.17 × 2.17 in | 55 × 55 mm | 1 : 1 | Also 65×65 mm. Distinctive, creative-industry signal. [Source: MOO, https://www.moo.com/us/business-cards] |
| Mini / slim | 2.75 × 1.1 in | 70 × 28 mm | 2.5 : 1 | MOO's signature "MiniCard" (70×28 mm). Long, slim, memorable. [Source: MOO] |
| Square mini | — | 45 × 45 mm | 1 : 1 | Rare boutique format (verify). |

**Korea verification note:** Both 90×50 mm and 85×55 mm are offered by every major KR printer.
90×50 mm is the *most common domestic standard* (cited explicitly for 금융·법률·의료·관공서). The
"90×55 mm" sometimes mentioned is NOT a common KR online-service trim size — KR services standardize on
**90×50** (domestic) and **85×55** (global). Treat 90×50 as the KR default.
[Source: Ohprint.me, https://www.ohprint.me/blog/business-card-size-guide-90x50-85x55]

### 1.2 The three lines: bleed, trim, safe

Printing cuts a large sheet down to the card; the cut ("trim") drifts ±1–2 mm. Three concentric
rectangles manage this:

1. **Bleed line (outermost)** — the artwork edge. Background colors/images must extend to here so that
   when the blade drifts outward, no white sliver appears. Bleed is typically **3 mm (0.125 in)** per
   side in Western print; **2 mm per side** is the prevailing Korean convention.
   - US/EU bleed: 3 mm each side → a 88.9×50.8 card is drawn at **94.9 × 56.8 mm** (≈3.75 × 2.25 in).
     [Source: VistaPrint, https://www.vistaprint.com/hub/crop-marks-explained]
   - KR bleed: 2 mm each side → a 90×50 card is worked at **94 × 54 mm** (도련 2 mm 사방).
     [Source: Redprinting, https://www.redprinting.co.kr/ko/guide2/view/2/49]

2. **Trim line (middle)** — the actual finished cut edge = the trim/재단 dimension (e.g. 90×50 mm).
   This is the "real" card size.

3. **Safe line / safety zone (innermost)** — keep all critical content (name, title, contact, logo)
   **3–5 mm inside the trim line** so an inward blade drift never clips text. KR convention: **3 mm**
   inside trim (안전 영역). Western convention: 3 mm minimum, 5 mm comfortable.
   - US safe zone for a 88.9×50.8 card ≈ **82.6 × 44.5 mm** (3.25 × 1.75 in).
     [Source: Apex, https://apexworkwear.ca/business-card-bleed-and-safe-area/]
   - KR safe zone for 90×50 ≈ **84 × 44 mm** (90−6 × 50−6, using 3 mm inset).

Mnemonic: **Background to BLEED. Text inside SAFE. Cut happens at TRIM.**
[Source: Gimmio, https://blog.gimm.io/what-is-the-bleed-cut-line-and-safety-line-in-business-cards/]

### 1.3 Resolution / units for HTML

- **Use physical units (`mm`) in CSS, never `px`, for print-spec geometry.** CSS treats `1in = 96px`
  and `1mm = 96/25.4 ≈ 3.7795px` as a *fixed* ratio independent of screen DPI, so `width: 90mm`
  renders as a true 90 mm box on a correctly-configured print path. Hardcoding pixels couples your
  layout to a screen-DPI assumption and prints wrong. [Source: MDN @page/size,
  https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/At-rules/@page/size; Stian Jørgensrud,
  https://stianjo.no/print-css-correct-size/]
- **DPI context:** Print is 300 DPI. CSS/screen is 96 DPI. Conversion at 300 DPI: **1 mm = 11.81 px**,
  **1 in = 300 px**. So a 90×50 mm card at 300 DPI = **1063 × 591 px**; the KR working size 94×54 mm =
  ~**1110 × 638 px** (Ohprint quotes 94×53 → 1110 × 626 px — printers differ by 1 mm).
  [Source: DesignYourWay, https://www.designyourway.net/t/mm-to-px-converter/; Ohprint.me]
- **@page** at-rule sets the printed page box: `@page { size: 90mm 50mm; margin: 0; }`. Use
  `margin: 0` so the card fills the page with no printer-margin offset, then draw your own
  bleed/trim/safe overlays inside.
- **Why px is risky:** a 1063px-wide div looks "card-sized" only on a 300-DPI assumption; on screen it
  is ~11 inches wide, and on print the browser may rescale it. Always anchor to `mm`.

### 1.4 Orientation

- **Landscape** is the default for ~90% of cards (wider than tall). Conservative, expected, maximal
  horizontal room for name + contact rows.
- **Portrait (vertical)** signals creative/fashion/design industries and reads as deliberately
  modern. Same hierarchy rules apply; you simply rotate the canvas (e.g. 50×90 mm).
  [Source: Looka, https://looka.com/blog/business-card-trends/]
- Front/back: backs are commonly a full-bleed brand color, logo lockup, or pattern. Treat each side as
  an independent canvas sharing one type/color system.

---

## 2. Korean Online 명함 Services (upload targets)

KR printers all use a **trim size + bleed (도련) → working/작업 size** model, almost always **2 mm
bleed per side**, **3 mm inner safe zone**, **300 DPI**, **CMYK**, fonts **outlined to paths**.

| Service | URL | Trim (재단) | Working (작업) | Bleed | Safe | Notes |
|---|---|---|---|---|---|---|
| 레드프린팅앤프레스 (Redprinting) | redprinting.co.kr | 90 × 50 mm | **94 × 54 mm** | 2 mm/side | 3 mm inside trim | Formats: JPG, PNG, PSD, PDF, CDR, EPS, AI. 300 DPI+. Strong for 박/형압(foil/deboss), specialty stock. [Source: https://www.redprinting.co.kr/ko/guide2/view/2/49] |
| 오프린트미 (ohprint.me) | ohprint.me | 90 × 50 mm (KR std) / 85 × 55 mm (global) | **94 × 53 mm** (≈1110×626 px @300dpi) | 2 mm/side | 3 mm | 300 DPI, CMYK, outline fonts. Template-style online editor. Industry guidance: 90×50 for finance/law/medical/gov; 85×55 for design/IT/global. [Source: https://www.ohprint.me/blog/business-card-size-guide-90x50-85x55] |
| 비즈하우스 (Bizhows) | bizhows.com | 90 × 50 mm; also 86×52, 86×54(카드명함) | **92 × 52 mm** (1 mm/side standard) or **94 × 54 mm** (2 mm/side, 당일명함) | 1–2 mm/side (product-dependent) | (per template) | Working size varies by product — *must* match exact working size or print is rejected. Has online editor + template uploads. [Source: https://www.bizhows.com/cms/help_center/namecard_size_2/] |
| 성원애드피아 (Sungwon Adpia) | swadpia.co.kr | 90 × 50 mm (typical) | ~94 × 54 mm (2 mm/side) (verify) | 2 mm/side (verify) | 3 mm (verify) | Long-running bulk B2B printer; AI/PDF upload workflow. Confirm per-product template. [Source: http://www.swadpia.co.kr/] |
| 와우프레스 (Wowpress) | wowpress.co.kr | 90 × 50 mm (typical) (verify) | 94 × 54 mm (2 mm/side) (verify) | 2 mm/side (verify) | 3 mm (verify) | Specialty-finish printer. Confirm template per product. |
| 모다82 (Moda82) | moda82.com | 90 × 50 mm (typical) (verify) | (verify) | (verify) | (verify) | Budget bulk 명함. Confirm template. |
| 스냅스 (Snaps) | snaps.com | photo-product oriented | (verify) | (verify) | (verify) | Photo-centric; business cards secondary. Confirm template. |

**Canonical KR upload target for a generated mockup:** design at **trim 90 × 50 mm**, extend background
to **working 94 × 54 mm** (2 mm bleed all sides), keep text within **84 × 44 mm** safe rectangle, export
300 DPI CMYK. This satisfies Redprinting and the strict end of Bizhows simultaneously; Ohprint's 94×53
differs by 1 mm vertically (their bleed is 2 mm L/R but 1.5 mm T/B equivalent — when in doubt the larger
94×54 working size is safe because excess bleed is simply trimmed).
[Sources: Redprinting; Ohprint.me; Bizhows — URLs above]

**Practical rule for the skill:** since KR services standardize on **2 mm bleed / 3 mm safe / 90×50
trim**, bake those as the KR preset. Use **3 mm bleed / 3–5 mm safe / 88.9×50.8 trim** as the US preset
and **3 mm bleed / 85×55 trim** as the EU/global preset.

---

## 3. Globally Notable Services & Their Design Ethos (taste anchors)

- **MOO** — the premium-minimal benchmark. Ethos: the card is an *object*, not a flyer. Heavy uncoated
  Mohawk Superfine stock, four-layer "Luxe" cards with colored edge seams, Spot Gloss, **letterpress**
  emboss/deboss, soft-touch. Design implication: with great paper and finishes, "the card doesn't need
  to be much more than a beautifully minimalist logo." Restraint, generous whitespace, one logo, one
  type treatment. Also originated the slim **MiniCard (70×28 mm)** and square formats. *Mentally anchor
  "premium" here.* [Source: MOO, https://www.moo.com/blog/inspiration/minimal-business-cards-with-powerful-minimalist-design; https://www.moo.com/us/business-cards/luxe]
- **Vistaprint** — print-first, volume-first. Huge template library, mainstream/"safe" aesthetic,
  optimizes for "get it printed" over design distinction. Useful as a *baseline of the generic* — what
  to deliberately out-design. [Source: https://www.vistaprint.com/hub/business-card-layout]
- **Canva** — accessible, free, template-driven. Fast and competent, but customization hits a ceiling
  and outputs trend toward "indistinguishable from everyone else." Represents the **slop risk**: drag-
  and-drop sameness. [Source: comparison roundups, quickfast.blog / shareecard.com]
- **Adobe Express** — design-first free tool. 20,000+ Adobe Fonts, exports print-ready files for any
  printer (not locked to one print vendor). Higher typographic ceiling than Canva; better font pairing
  raw material. [Source: https://quickfast.blog/business-card-creator-comparison-canva-adobe-express-vistaprint-more/]

Taste ladder for the generator: **aim at MOO/Adobe-Express quality, never settle at Canva/Vistaprint
template-default.**

---

## 4. Design Principles — the Craft (most important)

### 4.1 Typographic hierarchy

**Three tiers, three sizes, two typefaces max.** Stick to 2–3 type sizes total. [Source: VistaPrint,
https://www.vistaprint.com/hub/best-fonts-for-business-cards; MOO,
https://www.moo.com/blog/inspiration/business-card-font-size]

| Tier | Element | Recommended print size | Treatment |
|---|---|---|---|
| 1 (hero) | Name (or company) | **10–16 pt** (commonly 11–14 pt); name is the anchor | Heaviest weight or largest size; often tracked/kerned, sometimes uppercase or small-caps |
| 2 (mid) | Job title / company / role | **8–10 pt** | Lighter weight, often a contrasting case (e.g. uppercase + letter-spacing) |
| 3 (small) | Contact: phone, email, web, address | **7–9 pt** (never below **7 pt**; 8 pt is the safe floor) | Regular weight; align in a tidy block |

[Sources: 4over4, https://www.4over4.com/content-hub/stories/what-size-font-on-a-business-card;
Banana Print, https://www.banana-print.co.uk/blog/business-card-font-size/]

**Type scale ratios (apply a modular scale).** Use a ratio of **1.25–1.6** between tiers. Example with a
~1.4 ratio: contact 8 pt → title 8 pt (tracked) → name 13 pt. Or a louder card: contact 8 pt → name
18–22 pt as a typographic hero. In px-on-screen terms at the working canvas, 1 pt ≈ 1.333 px @96dpi, but
**set sizes in `pt` or `mm` for print fidelity** (e.g. `font-size: 3.4mm` ≈ 9.6 pt).

**HTML/CSS size cheats (print pt):**
- Name hero: `font-size: 12pt` (subtle) up to `22pt` (type-as-hero).
- Title: `font-size: 8pt; letter-spacing: 0.12em; text-transform: uppercase;`
- Contact: `font-size: 8pt; line-height: 1.45;`

**Font pairing principles:**
- **Max 2 typefaces.** Three reads as chaos at this scale.
- Three reliable strategies: (a) **one sans family, multiple weights** (e.g. Light for contact, Semibold
  for name) — safest, cleanest; (b) **serif name + sans contact** (editorial contrast, name carries
  personality); (c) **display/serif hero + neutral grotesque body**.
- Sans-serif improves small-size legibility (~31% better at small sizes per MIT AgeLab citation), so
  **contact details should almost always be sans** even when the name is a serif. [Source:
  numberanalytics, https://www.numberanalytics.com/blog/typography-in-business-cards]
- Pair across a clear contrast axis (serif vs sans, or weight Light vs Bold) — never two similar sans
  that merely look "slightly off."

**Kerning, tracking, alignment:**
- **Kern the name.** Names are short and prominent; default tracking often looks loose or has awkward
  pairs (e.g. "AV", "To"). Tighten the name; optionally add positive tracking (0.05–0.15em) for an
  uppercase, spaced, luxury feel.
- **One alignment system per card.** Pick left-aligned OR centered OR right-aligned and commit; mixing
  alignments is the #1 amateur tell. Left-aligned flush to a grid column reads most "designed."
- **Optical alignment:** align to the visual edge, not the bounding box — hang punctuation/quotes, and
  let round letters (O, C) slightly overshoot a baseline/edge so they appear aligned.
- **Baseline rhythm:** set contact lines on a consistent leading (line-height 1.35–1.5) so the block
  forms a clean rectangle.

### 4.2 Whitespace / balance

- **"Less is more."** Whitespace is the dominant tool for perceived quality. Isolated information is
  remembered far more (isolation effect); clutter destroys hierarchy. Aim for a card that is **40–60%
  empty** for premium minimal looks. [Source: Mobilo,
  https://www.mobilocard.com/post/minimalist-business-cards; FasterCapital]
- **Margins:** never let content touch the trim. Beyond the 3–5 mm safe zone, add **visual** margin —
  often 6–10 mm of breathing room on the "open" side of an asymmetric layout.
- **One focal point.** Decide what the eye hits first (usually the name or the logo) and protect it with
  surrounding space. Don't give two elements equal loudness.
- **Symmetry vs asymmetry:** centered/symmetric = formal, classic, can feel generic if default.
  Asymmetric (content anchored to one corner/column, large void opposite) = modern, intentional,
  "designed." Asymmetry is the easiest single move to escape template-look. [Source: OddPlan,
  https://oddplan.com/blogs/articles/business-card-layout-guide]
- **Grid systems:** impose a column grid (e.g. a **2- or 3-column** structure, or a 12-unit micro-grid)
  and a baseline grid. Swiss/International style = strict grid + flush-left sans + generous void.
  Even a single consistent left margin "rail" makes a card look engineered.
- **Visual weight distribution:** balance a heavy element (logo, color block, bold name) against
  whitespace or a light element on the opposite side so the card doesn't feel tippy. Think of the card
  as a scale.
- **Logo vs contact placement:** common, reliable arrangements — logo top-left + contact bottom-left
  (single rail); logo centered top + contact centered bottom (symmetric); logo on back, type-only
  front; name top + contact bottom-right corner (diagonal tension). Put the **logo and contact block in
  different regions** so neither crowds the other.

### 4.3 Brand color coherence

- **Limited palette: 1–2 brand colors + 1–2 neutrals.** A typical premium card is 1 ink color + paper
  white, or 1 brand color + black + white. More than two saturated colors looks cheap.
- **Contrast / legibility for small print:** small type needs strong contrast. Target body/contact text
  contrast comfortably above WCAG AA for normal text (**≥ 4.5:1** luminance contrast against its
  background); for a name at large size **≥ 3:1** is acceptable but more is safer in print. Avoid pale
  gray text on white for contact info — it disappears at 8 pt.
- **Accent usage:** use the brand color sparingly as an **accent** — a rule line, the name, an icon, a
  single keyword, or the card back — not as a wash behind small text. The accent earns attention
  precisely because it is rare on the card.
- **When to invert (dark cards):** dark/black cards read as premium and high-contrast and make a single
  accent color or foil pop. Use them when the brand is bold/luxury/tech; ensure text is near-white (not
  pure #FFF on pure #000 if you want a softer, less harsh feel — e.g. #F4F4F2 on #111). Dark backgrounds
  must still be full-bleed to the 2–3 mm bleed line.
- **Color space:** screen mockups are **sRGB**; production print is **CMYK** and cannot reproduce every
  vivid sRGB value (especially bright blues/greens/oranges). Keep brand colors print-realistic; avoid
  neon sRGB that will shift dramatically in CMYK. Note this caveat in any export.

---

## 5. Layout Archetypes / Pattern Catalog (drives variety)

Use this named catalog to make N mockups look genuinely different. Mix archetype × orientation × color
treatment × type strategy for maximal variance.

1. **Centered Minimal** — everything centered, deep whitespace, name + small contact line. Classic,
   restrained. Distinct via extreme spacing and one type weight.
2. **Left-Rail Accent Bar** — a vertical color bar down the left edge (4–8 mm), content flush-right of
   it. Distinct via the structural color rail anchoring the grid.
3. **Split Diagonal** — a diagonal line/band divides the card into two color fields; content sits in one
   half. Distinct via dynamic, non-orthogonal geometry. [Source: ZillionDesigns]
4. **Full-Bleed Color Block** — entire card is one saturated brand color; type reversed out (white/ink).
   Distinct via bold mono-color confidence; logo-led.
5. **Type-Only Editorial** — no logo; the name set huge as the hero in a display/serif face, contact as
   tiny footnote. Distinct via treating typography itself as the brand. [Source: Looka trends]
6. **Logo-Dominant** — large centered logo/wordmark, minimal supporting text. Distinct via brand-mark
   scale; for strong existing identities.
7. **Swiss / Grid (International)** — strict column grid, flush-left grotesque sans, asymmetric void,
   thin hairline rules. Distinct via rigorous modular structure.
8. **Monogram Center** — large initials/monogram centered or in a corner medallion; contact small below.
   Distinct via the crafted monogram focal point.
9. **Vertical / Portrait** — rotated 50×90 canvas; stacked hierarchy. Distinct purely via orientation
   signaling creativity.
10. **Border Frame** — a thin inset keyline frame (inside the safe zone) containing centered content.
    Distinct via classic stationery framing; reads formal/traditional.
11. **Two-Tone Fold-Mimic** — card split into two horizontal/vertical color fields (e.g. dark top /
    light bottom) implying a fold or duotone. Distinct via the two-field color block composition.
12. **Photographic Back** — front type-only; back full-bleed photo/texture/pattern. Distinct via
    image-as-surface; pairs minimal front with rich back.
13. **Letterpress / Deboss Illusion** — simulate impressed type via subtle inner shadow + highlight on a
    textured off-white; tonal (ink-less) marks. Distinct via tactile craft cue. [Source: MOO Luxe]
14. **Foil-Accent** — one element (name, rule, icon) rendered as metallic gold/silver/copper gradient.
    Distinct via the single luxe metallic moment on an otherwise restrained card.
15. **Corner-Anchored Asymmetry** — all content packed into one corner (e.g. bottom-left), vast empty
    diagonal opposite. Distinct via aggressive asymmetry and negative space.
16. **Big-Number / Big-Initial** — an oversized single glyph or initial bleeding off an edge as graphic
    element behind/beside the contact block. Distinct via scale contrast and edge-cropping.
17. **Hairline-Rule System** — thin horizontal/vertical rules organize content into labeled rows
    (Tel / Email / Web), table-like and editorial. Distinct via the rule grid as ornament.
18. **Duotone Gradient Field** — a restrained two-stop gradient (within one brand hue family, NOT a
    rainbow) as the field; reversed type. Distinct via controlled gradient (anti-cliché version).
19. **Sticker / Badge Lockup** — a circular or rounded badge containing the mark, placed off-center,
    contact set against plain ground. Distinct via the contained badge shape.
20. **Index-Card / Form Aesthetic** — labeled fields, monospace contact, ruled lines, faux-functional
    "spec sheet" look. Distinct via the deliberate utilitarian/technical voice.
21. **Die-Cut / Cropped-Corner Illusion** — one corner visually clipped or rounded asymmetrically as a
    structural device. Distinct via implied non-rectangular silhouette.
22. **Texture-Ground Minimal** — subtle paper/linen/concrete texture as the whole ground, single ink
    color of type. Distinct via material surface doing the work.

**Variety recipe for 20 mockups:** assign each card a *different* archetype from the list above; rotate
through ~3 color treatments (light, dark, brand-block) and ~3 type strategies (single-sans-weights,
serif+sans, type-as-hero); include at least 2 portrait cards and at least 2 "one bold move" cards. No two
cards should share archetype + alignment + color treatment.

---

## 6. Anti-Slop Guidance (generic vs distinctive)

**Tells of a generic Word/template card** (avoid all): everything center-aligned; default Arial/Times/
Calibri; a rainbow or blue→teal gradient background; an outer drop shadow on the whole card or on text;
bevel/emboss filter effects; clip-art phone/email/pin icons in a colored circle row; a fake swoosh or
"corporate Memphis" curve; logo + name + 5 contact lines all the same weight; both sides crammed; stock
"abstract globe" mark. [Sources: Mobilo; VistaPrint layout hub; design roundups]

**Instead of X, do Y:**
1. **Instead of** centering everything **→ do** flush-left to a single grid column with a large
   asymmetric void; let the empty space be the design.
2. **Instead of** default Arial/Calibri at one size **→ do** a deliberate pairing (one grotesque +
   one serif, or one family in Light/Bold) with a clear 3-tier size hierarchy.
3. **Instead of** a gradient + drop-shadow background **→ do** one flat brand color block (full-bleed)
   OR plain paper-white with a single hairline rule; remove all shadows.
4. **Instead of** clip-art icons in colored circles **→ do** plain text labels (Tel / Email / Web) or
   one consistent thin monoline icon set, unboxed.
5. **Instead of** five equal-weight contact lines **→ do** a tight, small, single-weight contact block
   subordinate to a clearly larger name; let hierarchy breathe.
6. **Instead of** decorating every corner **→ do** ONE bold move (an oversized initial, a foil rule, a
   diagonal split, or the name set huge) and keep everything else quiet.
7. **Instead of** stretching the logo to fill space **→ do** size the logo to its optical weight and
   pad it with whitespace; scale ≠ importance.

**One-line doctrine:** Restraint + one intentional grid + one bold move + real type hierarchy. If a card
has three "bold moves," it has none.

---

## 7. Designers / Studios / Sources to Emulate

- **Pentagram** — world's largest independent design consultancy; gold standard for identity +
  stationery systems (custom lettering, rigorous logic, restraint). Anchor "good identity stationery"
  here. [Source: https://www.pentagram.com/]
- **Base Design** — multidisciplinary identity studio known for typographic, editorial, system-driven
  stationery. Anchor "editorial type-led" cards. (verify specific projects via basedesign.com)
- **Bibliothèque, Spin, Build (Michael C. Place), Mucho, Order, Collins, Pentagram NY** — studios known
  for disciplined grid + type identity work worth emulating for card systems. (verify per studio)
- **Swiss/International Typographic Style** — Müller-Brockmann, Emil Ruder: the grid + grotesque + void
  doctrine that underpins the cleanest cards.
- **MOO Luxe / letterpress portfolio** — the tactile-premium reference (paper, edge color, deboss,
  spot gloss). [Source: https://www.moo.com/us/business-cards/luxe]
- **Where to browse current craft:** Behance (search "business card identity", "stationery system",
  "letterpress business card", "brand identity card"), Dribbble (search "business card", "name card",
  "stationery"), and case studies on **brand new / underconsideration.com**. Filter for restraint and
  grid logic; ignore the gradient-and-shadow majority.
- **Books to mentally anchor:** *Grid Systems in Graphic Design* (Müller-Brockmann); *Thinking with
  Type* (Ellen Lupton); *Logo/Letterhead/Stationery* design annuals.

---

## 8. HTML/CSS Implementation Notes

### 8.1 Build one print-spec card

Set the **physical size in mm**, use `@page`, and keep everything in `mm`/`pt`:

```html
<!-- One card: KR 90×50 trim, 2mm bleed → 94×54 working -->
<style>
  :root{
    --trim-w: 90mm; --trim-h: 50mm;       /* finished cut size */
    --bleed: 2mm;                          /* per side (KR=2mm, US/EU=3mm) */
    --safe: 3mm;                           /* inset from trim */
    --work-w: calc(var(--trim-w) + 2*var(--bleed));  /* 94mm */
    --work-h: calc(var(--trim-h) + 2*var(--bleed));  /* 54mm */
  }
  @page { size: 94mm 54mm; margin: 0; }    /* match working size, no printer margin */

  .card{
    position: relative;
    width: var(--work-w); height: var(--work-h);
    background: #fff;                        /* background fills to BLEED edge */
    overflow: hidden;                        /* nothing escapes the working canvas */
    font-family: "Inter", Arial, sans-serif;
  }
  /* content lives inside the SAFE rectangle: bleed + safe inset on all sides */
  .safe-area{
    position: absolute;
    inset: calc(var(--bleed) + var(--safe));  /* 5mm in from working edge */
  }
  .name   { font-size: 13pt; font-weight: 600; letter-spacing: -0.01em; }
  .title  { font-size: 8pt; text-transform: uppercase; letter-spacing: 0.12em; }
  .contact{ font-size: 8pt; line-height: 1.45; }
</style>
```

### 8.2 Toggleable bleed / trim / safe guide overlays

Draw three absolutely-positioned outlines and gate them behind a class so they show on screen for review
but can be hidden for the "clean" render and for print:

```css
.guides .trim,
.guides .safe{ position:absolute; pointer-events:none; }
.guides .trim{ inset: var(--bleed); outline: 0.2mm dashed #e0006d; }   /* magenta = trim/cut */
.guides .safe{ inset: calc(var(--bleed) + var(--safe)); outline: 0.2mm dashed #00a0e0; } /* cyan = safe */
.card::after{ /* bleed edge marker */ content:""; position:absolute; inset:0; outline:0.2mm solid #00b050; } /* green = bleed */
@media print { .trim, .safe, .card::after { display:none; } }  /* hide guides when printing */
```

Add a checkbox/button toggling `.guides` on the `.card` to flip overlays. Standard color legend: green =
bleed, magenta/red = trim, cyan/blue = safe.

### 8.3 Fonts

- Embed via Google Fonts `<link>` with `display=swap`, OR (better for print fidelity) self-host/`@font-face`
  with `font-display: block` so the final font is used at render time and metrics don't shift.
- **Pin a metrics-compatible fallback** in the stack (e.g. `"Inter", Arial, sans-serif`) and, ideally,
  use `size-adjust`/`ascent-override` on an `@font-face` fallback to prevent reflow if the web font
  loads late. Layout shift from a late web font is the #1 print-mockup bug.
- Keep to **2 families max** (see §4.1).

### 8.4 Pitfalls

- **Web-font fallback shifting layout:** if the web font arrives after layout, line breaks and the name's
  width change → text can cross the safe line. Use `font-display: block` or preload fonts; verify the
  final render, not the first paint.
- **em vs mm:** set the *canvas and guides in `mm`*, but type in `pt` (print) — don't size the card in
  `em` (it rescales with font). Margins/insets in `mm`.
- **Background not reaching bleed:** if a colored/photo background stops at the trim, an off-cut shows
  white. Always paint the background to the full **working** size (the `.card` element), never only the
  `.safe-area`.
- **Text overflow:** constrain the contact block; test the longest realistic strings (long emails,
  international phone numbers). Use `overflow:hidden` on the card as a safety net but design so nothing is
  actually clipped.
- **Crisp render:** prefer vector text and CSS shapes over raster images; if rasterizing for export,
  render at **300 DPI** (the working 94×54 mm = ~1110×638 px). Avoid sub-pixel `box-shadow` blur for
  "deboss" — use small, hard inner shadows so it survives downscaling.
- **px creep:** never specify card geometry in `px`; it couples to a DPI assumption and prints wrong.

### 8.5 One-page gallery to tile N cards for human review

```css
.gallery{
  display: grid;
  grid-template-columns: repeat(2, 94mm);  /* 2-up; use repeat(auto-fill, 94mm) to flow */
  gap: 12mm;
  padding: 12mm;
  background: #f2f2f0;                       /* neutral gray ground so light cards read */
  justify-content: center;
}
.gallery .card{ box-shadow: 0 1mm 4mm rgba(0,0,0,.12); }  /* screen-only lift; not for print */
@media print { .gallery{ background:#fff; gap:0; } .gallery .card{ box-shadow:none; } }
```

- Put each card (front + optional back) in a labeled cell (`<figcaption>` with archetype name + size) so
  a human can scan all 20 at once and judge variety.
- Sit cards on a **mid-gray ground** (#f2f2f0) so both white and dark cards are clearly bounded.
- Provide a global toggle to show/hide all guides across the gallery for clean visual review.
- For an actual print proof, switch to `@page` per-card and one card per page; for human *screen* review,
  the tiled grid above is correct.

---

### Quick presets to bake into the skill

| Preset | Trim | Bleed/side | Working | Safe inset | Aspect |
|---|---|---|---|---|---|
| KR-standard | 90 × 50 mm | 2 mm | 94 × 54 mm | 3 mm | 1.8:1 |
| KR-global | 85 × 55 mm | 2 mm | 89 × 59 mm | 3 mm | 1.545:1 |
| US/Canada | 88.9 × 50.8 mm | 3 mm | 94.9 × 56.8 mm | 3–5 mm | 1.75:1 |
| EU/UK | 85 × 55 mm | 3 mm | 91 × 61 mm | 3 mm | 1.545:1 |
| Japan | 91 × 55 mm | 3 mm | 97 × 61 mm | 3 mm | 1.655:1 |
| Square | 55 × 55 mm | 3 mm | 61 × 61 mm | 3 mm | 1:1 |
| MOO MiniCard | 70 × 28 mm | 3 mm | 76 × 34 mm | 3 mm | 2.5:1 |

All presets: 300 DPI for raster export, sRGB for screen / CMYK for production, fonts outlined for KR
print upload.
