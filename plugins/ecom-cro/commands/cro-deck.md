---
name: cro-deck
description: Build a CRO recommendation deck end-to-end — from client brief to a ready-for-Claude-Design output package.
---

# /cro-deck — CRO Deck Workflow

You are producing a CRO recommendation deck for an ecom client. Walk through this workflow step by step. Do not skip steps. Do not run them out of order. After each step, briefly confirm to the user what was done before continuing.

**Hard constraints (apply throughout):**

- This is for an ecom client. PDPs and PLPs are the most leveraged pages. Cart drawer matters. Standard Shopify checkout is low-leverage; only audit it deeply if the user flagged Shopify Plus + Checkout Extensibility, or asks for it.
- Mobile-first by default unless the user picks web in Step 2.
- The plugin ships with NO brand assets — every run, you ask the user to provide (or describe) their style. Then you save a per-client style profile to `outputs/<client-slug>/style-profile.json` and treat it as law for the rest of that run.
- **The primary deliverable is an interactive HTML prototype** (`prototype.html`) that recreates the client's current site at near-1:1 fidelity with toggleable rec controls. The user can share this URL directly with the client. The `.pptx` is an optional follow-on artifact via Claude Design hand-off; previous attempts with Claude Design have consistently failed at photoreal mockup synthesis, so HTML is preferred.
- Every recommendation must cite at least one heuristic by name from the `cro-heuristics` skill. Read it before generating recs.

All output files for one deck go to `outputs/<client-slug>/` where `<client-slug>` is the client name in lowercase with hyphens.

---

## Step 0 — Brand assets and style intake (run once per session)

Before any client work, gather the user's (the agency's / team's) brand assets and visual style. This is what the deck will be styled in.

Ask the user (use AskUserQuestion if available, else ask in chat):

> "Before I build the deck, I need your brand style. You can either:
> 1. **Drop an existing CRO deck (.pptx)** — I'll extract colors, fonts, and layout patterns
> 2. **Drop a brand guidelines doc** — I'll extract the relevant style fields
> 3. **Answer a few quick questions** — I'll build a style profile from your input
>
> Which would you like? (You can also drop a logo image — optional.)"

### If the user drops a deck or brand guide:

Extract these fields. Confirm them with the user before saving.

- Slide aspect ratio (16:9 default, but check)
- Slide width × height in inches (default 10 × 5.625")
- Primary brand color (hex)
- Secondary / accent color (hex, if any)
- Body text color (typically a gray/charcoal)
- Header/section title color
- Font family (e.g. Montserrat, Inter, Helvetica)
- Header bar style (color, position, texture/pattern, header text color)
- Page number style and position
- Logo asset (path, placement)
- Any decorative elements (lines, motifs, image-frame styles)

### If the user picks Q&A:

Ask in one message:

1. Aspect ratio? (default 16:9)
2. Primary brand color (hex)? Secondary/accent (hex)?
3. Font family? (default Inter or Montserrat)
4. Do you have a logo to include? (yes → ask them to drop it; no → skip)
5. Do you have a header bar style? (e.g. "blue bar at top with white title", "no bar — minimal")
6. Anything else you want to enforce visually? (free text)

### Save to `outputs/<client-slug>/style-profile.json`

After client name is known (Step 1), write the style profile to disk in this exact schema:

```json
{
  "deck_format": {
    "aspect_ratio": "16:9",
    "width_inches": 10,
    "height_inches": 5.625,
    "device_target_for_screenshots": "iPhone 16 Pro",
    "screenshot_logical_resolution": "402x874",
    "screenshot_pixel_resolution": "1206x2622",
    "screenshot_pixel_density": "3x"
  },
  "fonts": {
    "primary_family": "<Montserrat | Inter | etc>",
    "title_weight": "Bold",
    "section_header_weight": "Medium",
    "body_weight": "Regular",
    "label_weight": "Bold"
  },
  "colors": {
    "primary": "<#hex>",
    "secondary": "<#hex or null>",
    "section_title": "<#hex e.g. #434343>",
    "body_text": "<#hex e.g. #525252>",
    "current_label": "#FF0000",
    "suggested_label": "#34A853",
    "white": "#FFFFFF"
  },
  "header_bar": {
    "use_header_bar": true,
    "fill": "<#hex or null>",
    "texture_note": "<description if any pattern>",
    "title_text": "Website Recommendation",
    "title_color": "#FFFFFF",
    "title_weight": "Bold"
  },
  "logo": {
    "use_logo": true,
    "path": "<path the user provided or null>",
    "placement_when_used": "top-left, ~1 inch wide"
  },
  "rec_slide_template": {
    "left_rail_width_inches": 3.5,
    "current_label": {"text": "Current", "color": "#FF0000", "weight": "Bold"},
    "suggested_label": {"text": "Suggested", "color": "#34A853", "weight": "Bold"},
    "separator": "vertical dashed line",
    "screenshots": {
      "aspect": "mobile portrait (iPhone 16 Pro)",
      "approx_width_inches": 1.95,
      "approx_height_inches": 4.3
    }
  },
  "deck_flow": {
    "include_cover_slide": false,
    "slides_in_order": [
      "Contents",
      "Section divider: UI/UX Recommendation",
      "UI/UX rec slides",
      "Section divider: Strategic Recommendation",
      "Strategic rec slides",
      "Closing: sources + comments link"
    ]
  }
}
```

Fill in defaults for fields the user didn't specify. Confirm the final profile with the user before continuing.

---

## Step 1 — Context

### 1.1 Get the brief

Ask the user to drop the client brief. Accepted: .md, .docx, .pdf, or pasted text. If nothing arrives, ask once more, then fall back to gathering the bare minimum directly: audience, products, top 3 competitors, primary website URL, blockers.

### 1.2 Parse the brief

Extract these fields. Treat missing or "Unknown" as null. Do NOT invent.

**Pull from the brief:**
- `client_name`
- `primary_url` — main website to audit
- `audience` — demographics, psychographics, geography, income/lifestyle (split if mixed in one cell)
- `products` — categories sold; top categories if specified
- `competitors` — list of competitor URLs (clean to `https://domain.com/`)
- `differentiators` — what client claims makes them different (often blank — note if so)
- `purchase_obstacles` — reasons customers don't buy
- `platform` — Shopify / Shopify Plus / Magento / custom (infer if not stated, ask if unclear)
- `brand_assets` — links to brand guidelines, fonts, colors mentioned
- `seasonality_or_constraints` — anything affecting timing

**Filter OUT (do not load into context):**
- ROAS / CPA targets, budget caps, paid channel performance, paid campaign tactics, email/SMS list metrics

**Edge cases:** markdown tables with `| field | value |` rows; single-cell narrative paragraphs; mixed formats; "Unknown" / blanks; competitor URLs in mixed formats.

### 1.3 Show the parsed context for confirmation

Print a compact summary like:

> **Client:** Example Linens — example.com
> **Audience:** Women 45+, top 20% wealth, college-educated, US
> **Products:** Luxury linens, bed/bath, decorative pillows
> **Differentiators:** Not specified
> **Purchase obstacles:** Heavy competition; customers can't find specific products
> **Top competitors found:** competitor1.com, competitor2.com, competitor3.com
> **Platform:** Shopify (inferred)
>
> Ready to continue?

### 1.4 Clarifying questions

Ask these four in a single message:

1. **Designs based on mobile or web screens?** (default: mobile)
2. **How many recommendations?** (default: 10)
3. **Which 5 competitors to use as reference?** Show parsed list; user confirms / deselects / adds. Cap at 5.
4. **Any extra context to weight the recs?** (Optional. Examples: "quick wins only", "focus on AOV not CR", "client only has design + copy resources, no dev", "client wants the cart drawer fixed", "client is on Shopify Plus with Checkout Extensibility")

### 1.5 Save context

Write `outputs/<client-slug>/client-context.json` with all parsed + clarified fields. Slug the client name (lowercase, hyphens, no special chars).

---

## Step 2 — Heuristic + audit research

### 2.1 Load the heuristic library

Read the `cro-heuristics` skill from this plugin. This is the framework you'll use in Step 3. Note the audience-tuning table — find the row closest to the client's audience signal (or infer the closest analog).

### 2.2 Pick relevant heuristics for THIS audience

Select 8–12 heuristics from the library most likely to surface high-impact recs for this client. For each, write down:

- **heuristic_name** — exact name from the library (e.g. "Cialdini: Authority", "NN/g #5 Error prevention", "PDP: trust signals near CTA")
- **audience_tuned_execution** — what "good" looks like for THIS audience, not generic
- **page_types_to_check** — PDP / PLP / cart / homepage / nav
- **likely_anti_patterns** — what to flag during the site audit as violations

Also identify 3–5 **audience anti-patterns** — things this audience strongly dislikes (e.g. luxury 45+ + aggressive scarcity = trust damage). These will cut recs in Step 4.

### 2.3 Audit the client website

Use browser tools. Set viewport to mobile (logical 402×874, 3x density) — or desktop (1440×900) if user picked web.

Visit pages in this priority order:

1. **2–3 PLPs** — top categories from the brief or top nav
2. **1–2 PDPs per PLP** (PDPs are the most important — bias coverage here)
3. **Homepage** — single visit, full scroll
4. **Cart drawer / mini-cart** — add an item, observe the drawer
5. **Skip standard checkout** unless flagged in Step 1

Before each observation: close cookie banners, popups, and chat widgets. Decline cookies (privacy-preserving default). Don't auto-fill forms. Don't complete real purchases.

For each page, record:

- `page_type` (homepage / PLP / PDP / cart)
- `url`
- `funnel_stage` (Discovery / Consideration / Decision / Add-to-cart / Cart)
- `above_fold_observations` — what does the user see first viewport?
- `friction_points` — list of issues, each tagged by category (`clarity`, `trust`, `friction`, `value_communication`, `visual_hierarchy`, `mobile_specific`, `performance`, `cross_sell`, `error_prevention`) + severity (`high`, `medium`, `low`)
- `strengths` — what the page does well (don't recommend changing these)
- `competitor_gaps` — patterns the client lacks vs. typical category competitors

**Failure handling:** If site is auth-walled, geofenced, or anti-bot: ask the user to drop screenshots themselves and continue with manual notes. If individual pages fail: skip them, note why, continue.

Save observations as `outputs/<client-slug>/site-audit.json`.

---

## Step 3 — Ideation

Generate `ceil(N × 1.3)` candidate recommendations (e.g. 13 if user wants 10). The +30% buffer covers cuts in Step 4.

For each recommendation, fill ALL fields below. No nulls in required fields.

```json
{
  "index": 1,
  "title": "Short title — 5–8 words",
  "category": "UI/UX" | "Strategic",
  "recommendation": "ONE specific sentence describing the change",
  "issue": "What's currently wrong (null if rec is purely additive)",
  "why": "ONE sentence: heuristic + audience signal + (optional) competitor pattern",
  "heuristics_cited": ["Cialdini: Authority", "PDP: trust signals near CTA"],
  "audience_signal_used": "Older female luxury audience values reassurance and brand heritage",
  "competitor_pattern_observed": "competitor1.com shows return policy + warranty next to ATC button",
  "impact_metrics": ["↑ PDP conversion", "↓ bounce on PDP"],
  "expected_lift": "+3–8% on PDP add-to-cart",
  "expected_lift_basis": "Heuristic + competitor pattern (no public stat available)",
  "supporting_stat": {
    "stat": "70% of online eyewear buyers abandon cart due to unclear sizing",
    "source": "Fittingbox",
    "source_url": "https://fittingbox.com/..."
  },
  "funnel_stage": "PDP",
  "page_type_affected": "PDP",
  "ice": {"impact": 8, "confidence": 7, "ease": 9, "total": 504},
  "effort_estimate": "small | medium | large",
  "owner_role": "design + dev",
  "site_evidence_url": "URL of the audited page where the issue was observed"
}
```

**Quality bar:**

- `recommendation` is a specific change. NOT "improve the PDP" — instead "add a sticky 'Add to Cart' button on PDP mobile that persists when the user scrolls past the hero image".
- `why` is one sentence with at least one heuristic cited by name.
- Don't invent supporting stats. If no real public source, leave `supporting_stat` as null.
- Wider expected-lift ranges (5–15%) when basis is heuristic-only. Narrow ranges (3–6%) only when a comparable public study supports it.
- ICE scores: use the full 1–10 scale; don't bunch everything at 7–8.

**Mix rules:**

- Specify `category`: UI/UX (visual / structural / on-page) or Strategic (cross-sell flows, gift cards, content/blog, loyalty).
- Aim for ≥2 strategic recs unless inputs strongly justify all UI/UX.
- Cover multiple funnel stages — don't give all 10 PDP recs.
- Don't propose changes to anything the audit flagged as a `strength`.

**Audience guardrails:** Re-read the audience anti-patterns from Step 2.2. Do NOT generate any rec that violates them.

Sort by ICE total descending. Save as `outputs/<client-slug>/recommendations-draft.json`.

---

## Step 4 — Self-validation passes

Three independent re-reads. Between each pass, briefly clear your reasoning — read the rec list as a fresh evaluator, not as the author. The point is to catch things ideation missed.

### 4a — Audience-fit pass

For each rec, assign one of:

- `pass` — clear audience fit
- `pass-with-tuning` — fits in principle but execution needs adjustment for the audience
- `fail` — violates an audience anti-pattern or is meaningfully misaligned

Hard failures (always `fail`):

- Aggressive scarcity / countdown timers for luxury / 45+ / wellness / B2B
- Heavy BNPL emphasis on luxury PDPs
- Casual / Gen Z UGC framing for trust-driven older audiences
- Technical jargon for mass-market casual audiences
- Aggressive popups before browse for considered-purchase audiences

Soft conflicts → `pass-with-tuning`. Be specific in the rationale ("Aggressive countdown conflicts with audience preference for considered, reassurance-driven flow"), not generic ("doesn't match audience").

**Cut every rec marked `fail`.** For `pass-with-tuning`, apply the tuning to the rec's `recommendation` and `why` fields and keep it.

### 4b — Competitor-pattern scan (advisory only — do not cut)

For each surviving rec, scan the relevant page type on each of the 5 user-confirmed competitors. Browser tools, mobile viewport (or desktop if user picked web). Close cookies + popups before observing. Skip competitors that fail to load.

Label each rec:

- `validated` — ≥2 competitors implement this pattern → high confidence
- `partial` — 1 competitor, or partial implementation
- `differentiation` — no competitor does it (often a STRENGTH — flag, do NOT cut)
- `contradicted` — competitors actively avoid it (rare; investigate before pushing)

For each rec, capture:
- Specific competitor(s) implementing it
- Direct URL where the pattern is visible (this becomes the competitor reference screenshot in Step 6)
- One-sentence note on how their implementation differs from what we're proposing

**Critical: differentiation is often valuable. Never cut a rec because no competitor does it.**

### 4c — Feasibility classification

For each surviving rec, label:

- **Ship** — low risk, predictable upside, no A/B test needed. Examples: clearer copy, missing trust badges, restoring hidden FAQ, fixing broken element, industry-standard cross-sell.
- **Test** — meaningful upside but enough variance / opinion / brand risk to A/B test first. Examples: hero messaging, pricing display, checkout-step changes, large reflows, anything affecting brand voice.
- **Research-first** — fundamentally uncertain; needs qual or quant data first. Examples: structural taxonomy redesigns, audience-segment-dependent personalization, anything dependent on traffic patterns we don't have.

User-constraint overrides:
- "Quick wins only" → bias toward Ship on borderline calls.
- "No dev resources" → recs requiring dev → Test or Research-first regardless of upside.
- "We want to fix CR specifically" → don't penalize CR-focused recs.

Don't over-classify as Research-first — it's the safe-but-useless bucket. For `Test` items, write a one-line test design hint (e.g. "50/50 split, 14 days, primary metric: PDP add-to-cart rate").

### Synthesis

Merge all three pass results into each rec's metadata. Re-sort by ICE total descending. If the surviving count is below the user-requested N, loop back to Step 3 ONCE with a note about which themes / funnel stages need more recs.

---

## Step 5 — User confirmation

Show the user the final ordered list. Compact format — one rec per line:

```
1. [ICE 504 | Ship | PDP] Add sticky ATC button on PDP mobile
2. [ICE 480 | Test | Cart] Add free-shipping progress bar to cart drawer
3. [ICE 432 | Ship | PLP] Show review counts on product cards
...
```

Ask: "Approve this list, or want to drop / edit / add any?" Wait for explicit approval.

---

## Step 6 — Build interactive HTML prototype

Build a single-file HTML interactive prototype that recreates the client's current key pages (PDP, PLP, category landing, homepage, footer) at near-1:1 fidelity, with toggleable rec controls. The user can flip each rec on/off and watch the design compound toward the Suggested state.

This artifact replaces the static "Current + Suggested screenshot pair" pattern that consistently fails downstream. It is the primary deliverable.

### 6.0 — Two-path source-of-truth strategy

**ALWAYS try Path A first. Only fall back to Path B if Path A fails.**

#### Path A — Build from the client's actual DOM (PREFERRED)

Use browser tools to visit each key page on the client site. For each page:

1. Navigate (mobile viewport — set width to narrow or use `?country=US` if geo-redirect interferes)
2. Wait for full layout settle (5–8s, scroll page top→bottom→top to trigger lazy-load)
3. Close cookie banners, chat widgets, accessibility overlays
4. Extract the DOM structure using `read_page` (accessibility tree) AND `javascript_tool` to inspect computed styles, font sizes, colors, padding, section ordering. Pull real content: product titles, brand labels, prices, variant labels, swatches, description text, footer accordions.
5. Note the EXACT visual quirks the audit found: oversized header wordmark, mixed text+swatch selectors, dropdowns instead of chips, empty bordered circle swatches, sticky bottom popup behavior, etc. These are precisely what the rec toggles will fix in the Suggested state — they must be reproduced in the Current state.

Recreate each page in HTML as a phone-frame mockup, ~420px wide × ~840px tall, with:
- Sticky chrome (URL bar) at top
- Sticky page header below (so the header behavior matches reality — e.g. for Fig the huge wordmark sticks during scroll)
- Scrollable content area
- Sticky bottom recommendation popup if the client uses one (Shopify themes commonly do)

#### Path B — Fallback: ask the user for phone screenshots

If Path A fails (Cloudflare-blocked, geofenced, auth-walled, anti-bot, browser tool offline, or the site's responsive behavior cannot be reliably reproduced from DOM inspection), tell the user the exact pages and scroll states you need, and ask them to capture phone screenshots. Then build the HTML using their screenshots as visual reference (and optionally as base layer for image-heavy areas via `<img>`).

Before falling back, state explicitly what failed and why. Do NOT default to asking for screenshots without first attempting Path A.

### 6.1 — Pages to recreate

Minimum set (always):
- 1 PDP showing the most complex variant pattern audited (mixed swatches, huge stack, or dropdowns — whichever the client has)
- 1 PLP showing the existing product-card pattern
- 1 category landing page (the top-level destination of one main nav item)
- 1 homepage view (top of page, hero + first section)

Add for richer prototypes:
- Additional PDP variants if the client has multiple distinct patterns (e.g. PDP A for mixed swatches + PDP B for dropdowns)
- A scrolled-down PDP variant showing reviews / bundle / footer

### 6.2 — Toggle controls

For each approved rec from Step 5, define a toggle key (`r1`, `r2`, ... in rec_index order). Each toggle, when ON, modifies the rendered page to apply that rec's change. Toggles compound — flipping multiple ON stacks their effects.

For each rec, document in `outputs/<client-slug>/prototype-rec-map.json`:

```json
[
  {
    "rec_index": 1,
    "toggle_key": "r1",
    "applies_to_pages": ["pdpA","pdpB","pdpC","plp","cat","home"],
    "suggested_view_setup": {
      "page_tab": "pdpA",
      "toggles_on": ["r1"]
    },
    "compounded_view_setup": {
      "page_tab": "pdpA",
      "toggles_on": ["r1","r4","r7"],
      "note": "By slide N, the Suggested view should show this rec plus all prior compounded fixes."
    },
    "label_in_ui": "Compact header"
  }
]
```

The `suggested_view_setup` is for the isolated demonstration of that single rec. The `compounded_view_setup` is the cumulative state when the deck reader has progressed through all prior recs (the deck's compound-storytelling rule).

### 6.3 — Recommended UI layout for the prototype

- Top of widget: tab row for page selection (PDP A / PDP B / PLP / Cat landing / Homepage / etc.)
- Below tabs: control panel listing the toggles applicable to the current page. Each toggle is a labeled pill with a state dot. "Reset all" and "Apply all" buttons. Counter "N / Total on".
- Below controls: the phone frame, ~420px wide, with a fixed chrome and scrollable inner content area. Re-renders on every toggle change.

### 6.4 — Fidelity standards

The prototype must be honest. Older audiences and clients reviewing the deck need to recognize their own site immediately.

- Use real text content from the audit (real product names, real brand labels, real prices, real variant labels).
- Use real fonts where possible — fall back to system serif/sans-serif if the brand font can't be loaded from a CDN.
- Reproduce visual quirks: oversized wordmarks, sticky behaviors, sparse vs dense layouts, color palettes, button styling. The deck reader should look at the "Current" state and say "yes that's our site."
- For images, use SOLID FILL or subtle CSS gradients matching the typical product photography palette (off-white, beige, soft gray). Do not attempt to embed real product photos — they bloat the file and rarely render reliably.
- For the Suggested state, apply each rec's targeted change ONLY. Do not redesign surrounding UI.

### 6.5 — Output files

- `outputs/<client-slug>/prototype.html` — single-file, all CSS+JS inline. Self-contained. Openable in any browser.
- `outputs/<client-slug>/prototype-rec-map.json` — per-rec toggle mapping (see schema in 6.2).

### 6.6 — Quality gate

- All approved recs from Step 5 are mapped to toggles
- Every page tab renders cleanly with all toggles OFF (Current state)
- Every toggle visibly modifies the rendering when flipped ON
- "Apply all" on each page tab produces a coherent Suggested state — no broken layout, no overlap, no missing content
- Honest about audited quirks (e.g. sticky header, sticky bottom popup, exact swatch/dropdown patterns)
- File is under 200KB (no embedded images, no large fonts; CSS+JS+text only)

---

## Step 7 — Write the deck brief

Read the per-client `outputs/<client-slug>/style-profile.json` from Step 0 and embed its rules verbatim into the brief. Generate three files in `outputs/<client-slug>/`:

### File 1 — `deck-brief.md`

Use the template below. Replace all `{{TOKENS}}` with real values. Embed the style profile rules verbatim (do not paraphrase).

```markdown
# CRO Deck Brief — {{CLIENT_NAME}}

**Date:** {{DATE_YYYY_MM_DD}}
**Client URL:** {{CLIENT_URL}}
**Audience summary:** {{AUDIENCE_ONE_LINE}}
**Total recs:** {{TOTAL_N}} ({{UI_UX_COUNT}} UI/UX + {{STRATEGIC_COUNT}} Strategic)

---

## Style rules — LAW

<embed the contents of style-profile.json here, formatted as readable markdown sections>

### Mockup rules for "after" / Suggested screens — CRITICAL

Keep the original screenshot pristine. On the "after" version, modify ONLY the targeted element via overlay or composition. Every other element of the UI must remain visually identical — same fonts, colors, spacing, surrounding sections, header, footer, padding, status bar, scroll position. The targeted element must remain visible and clearly the focus.

DO NOT:
- Regenerate the entire UI from scratch
- Change unrelated colors or fonts
- Reflow surrounding layout
- Add or remove unrelated elements
- Use a different device frame from the "before" shot

PREFERRED METHODS:
- Crop + overlay the changed element on top of the original
- Inpaint only the affected region
- Replace a single component (e.g. a button, a section, a label) and keep everything around it

### Competitor reference

Per slide: small thumbnail or inset showing a competitor implementing the recommended pattern, with caption "as seen on {{competitor.com}}" in 10pt body color. Position: bottom-right of the right zone, OR small inline thumbnail below the Suggested screen.

---

## Deck flow

Per style profile: <state whether cover slide is included or not>

1. Contents
2. Section divider: "UI/UX Recommendation"
3. UI/UX rec slides
4. Section divider: "Strategic Recommendation"
5. Strategic rec slides
6. Closing: sources + comments link

---

## Slide 1 — Contents

Title: "Contents". Two-line list:
- UI/UX Recommendations  ........  p. 2
- Strategic Recommendations  ......  p. {{STRATEGIC_DIVIDER_PAGE}}

## Slide 2 — Section divider: UI/UX Recommendation

Large centered title "UI/UX Recommendation" per style profile. "Back to Contents" link bottom-left.

## Slide N — Recommendation slide template

For each rec, render this layout:

- **Header bar (if style profile uses one):** full-width per style profile; title "Website Recommendation" left-aligned, white bold
- **Left rail (~3.5" wide, x=0.16"):**
  - **Recommendation:** {{recommendation}}
  - **Why:** {{why}}
  - **Impact:**
    - {{impact_metric_1}}
    - {{impact_metric_2}}
  - **Expected lift:** {{expected_lift}}
  - **Supporting stat:** "{{stat}}" — [{{source}}]({{source_url}}) (omit if null)
- **Right zone:**
  - "Current" label (red, bold) and "Suggested" label (green, bold) above the two phone frames
  - Vertical dashed separator between the two sides
  - Phone frames: iPhone 16 Pro aspect (~1.95" × 4.3"), thin device frame
  - **Current** = screenshot of `prototype.html`, page tab from `prototype-rec-map.json[rec].suggested_view_setup.page_tab`, all toggles OFF
  - **Suggested** = screenshot of `prototype.html`, same page tab, with toggles from `prototype-rec-map.json[rec].compounded_view_setup.toggles_on` flipped ON (this is the compounded view — by slide N, the Suggested has this rec PLUS all prior recs visible together)
  - Layout type: symmetric | additive-only | removal-only
  - Competitor reference (optional): small inset thumbnail with caption "as seen on {{competitor.com}}" if a competitor implements the pattern visibly
- **Page number:** bottom-right per style profile

<repeat for every rec, in deck order>

## Closing slide

Title: "Websites used for the examples"

List every unique competitor URL referenced.

Footer line: "Share your thoughts, comments, and implementation plans in this spreadsheet — [{{COMMENTS_LINK_TEXT}}]({{COMMENTS_LINK_URL}})"
```

### File 2 — `claude-design-prompt.txt`

```
I'm handing off a CRO deck for {{CLIENT_NAME}}. Please produce the .pptx following deck-brief.md exactly.

Attached:
- deck-brief.md — master instructions, including embedded style rules and per-slide content
- prototype.html — interactive HTML prototype recreating the client's current site with rec toggles. This is the source of truth for every Current and Suggested screen — do NOT regenerate or mockup anything from scratch.
- prototype-rec-map.json — per-rec mapping of which prototype page tab + which toggles to flip ON for each rec's Current and Suggested view
- recommendations-validated.json — full structured rec data
- style-profile.json — visual style law
- (optional) logo or other brand assets

Critical rules:
1. The style profile embedded in deck-brief.md is law — do not deviate.
2. For each slide, screenshot the prototype with the specified toggle states (per prototype-rec-map.json):
   - Current = the prototype page tab, all toggles OFF
   - Suggested = the same page tab, with this rec's compounded_view_setup.toggles_on flipped ON
3. Compose the two screenshots into the slide's right zone with Current/Suggested labels and dashed separator.
4. Do NOT redraw or regenerate UI. Use only the prototype's rendered output as the visual source.
5. For purely additive recs, you may show only the Suggested side. For removal recs, only the Current side.
6. Font: install the brand font (per style profile) on the build environment before generating. If embedding fails, fall back to Helvetica Neue or Calibri. NEVER Arial.
7. The closing slide lists all competitor URLs referenced in the rec spec + the comments/feedback link.

Output: a single .pptx file named "{{CLIENT_NAME}} _ Website Recommendations.pptx".

Before finalizing, do a visual QA pass: render every slide as an image and check for left-rail text overflow, mis-aligned columns, broken page numbers, font fallback to Arial, and any prototype screenshots that show the wrong toggle state for the slide.
```

**Alternative (recommended):** the user may choose to skip the .pptx entirely and share `prototype.html` directly with the client. The HTML is a more honest, more interactive artifact than any static deck. Tell the user this option in Step 8.

### File 3 — `metadata.json`

Full structured rec data (every field from Step 3 + validator labels from Step 4) for traceability and future reuse.

### Quality gate before delivery

- All files present in `outputs/<client-slug>/`: `prototype.html`, `prototype-rec-map.json`, `deck-brief.md`, `claude-design-prompt.txt`, `metadata.json`, `recommendations-validated.json`, `style-profile.json`
- Every rec slide schema is filled (no nulls in required fields)
- Every rec in `prototype-rec-map.json` has a `toggle_key`, `suggested_view_setup`, and `compounded_view_setup`
- `prototype.html` opens cleanly in a browser, all tabs render, all toggles work, "Apply all" produces a coherent Suggested state per page
- Sources / closing slide in the brief lists every competitor URL referenced
- Style rules section is pasted from `style-profile.json` (not paraphrased)
- Page numbers are sequential
- The prototype and deck reflect the user's own brand from Step 0

---

## Step 8 — Hand-off

Present the user with both delivery paths and let them choose.

**Path A (recommended) — share the HTML prototype directly with the client:**

1. Open the output folder (provide `computer://` link).
2. Tell the user the primary deliverable is `prototype.html`. They can:
   - Open it in a browser to verify it looks right
   - Share the file directly with the client, OR
   - Host it via any static-file host (or just open from local) and share the URL
3. Pair it with `recommendations-validated.json` (or convert to Notion / Google Doc) for the per-rec "why" content the deck would have shown in the left rail.
4. Optional: walk the client through the toggles on a call so they can see the Current → Suggested transition live.

This is the strongest deliverable. It's honest (no fake mockups), interactive (better than static slides), and recreate-able by anyone who opens the file. Previous CRO deck attempts via Claude Design have failed at photoreal mockup synthesis; the HTML prototype solves that by being the artifact itself.

**Path B (optional) — produce a .pptx via Claude Design:**

If the client expects a static deck:

1. Open the output folder.
2. Start a new Claude Design session.
3. Attach: `prototype.html`, `prototype-rec-map.json`, `deck-brief.md`, `recommendations-validated.json`, `style-profile.json`, any brand assets.
4. Paste the contents of `claude-design-prompt.txt`.
5. Claude Design opens the prototype, screenshots each toggle state per `prototype-rec-map.json`, and composes the .pptx.

Note: Claude Design's reliability on this workflow has been mixed. If it fails, fall back to manually screenshotting the prototype toggle states and assembling a deck in PowerPoint/Keynote (about an hour of work for 12–15 recs).

Provide the `computer://` link to the output folder so the user can pick a path.
