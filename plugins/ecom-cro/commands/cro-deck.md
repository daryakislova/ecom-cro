---
name: cro-deck
description: Build a CRO recommendation package end-to-end — from client brief to an interactive HTML prototype with toggleable recs plus an implementation-tracker XLSX. No .pptx output.
---

# /cro-deck — Interactive CRO Prototype Workflow

You are producing a CRO recommendation package for an ecom client. The deliverables are two files: an interactive HTML prototype + an implementation-tracker XLSX (NOT a .pptx deck). Walk through this workflow step by step. Do not skip steps. Do not run them out of order. After each step, briefly confirm to the user what was done before continuing.

**Hard constraints (apply throughout):**

- This is for an ecom client. PDPs and PLPs are the most leveraged pages. Cart drawer matters. Standard Shopify checkout is low-leverage; only audit it deeply if the user flagged Shopify Plus + Checkout Extensibility, or asks for it.
- Mobile-first by default unless the user picks web in Step 2.
- The plugin ships with NO brand assets — every run, you ask the user to provide (or describe) their style. Then you save a per-client style profile to `outputs/<client-slug>/style-profile.json` and treat it as law for the rest of that run.
- **The deliverables are two files: an interactive HTML prototype** (`prototype.html`) that recreates the client's current site at near-1:1 fidelity with toggleable rec controls, **and an implementation-tracker spreadsheet** (`implementation-tracker.xlsx`) that the client uses to manage rollout. The user shares both directly with the client. No `.pptx` — it is not produced by this workflow.
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

This artifact, paired with the XLSX tracker from Step 7, is the deliverable. There is no .pptx output.

### Templates to start from

Two reusable scaffolds live in `templates/` — read them before writing widget code from scratch:

- **`templates/prototype-widget.html`** — full-site recreation with per-rec toggles + tabbed page switcher (PDP A/B/C, PLP, Category landing, Homepage). Phone-wrap 294×588px outer, inner phone 420×840px scaled `0.7`, square corners, sticky popup wired to IntersectionObserver. Use this for the primary Step 6 deliverable. Replace `{{CLIENT_*}}` placeholders, plug in your six page-render functions, render via `mcp__visualize__show_widget`.

- **`templates/current-vs-suggested.html`** — element-level redesign comparison (single popup, banner, or module). One toggle at top: Current / Suggested. Desktop + Mobile views stacked, both update together. Use this when a single rec needs its own focused mockup (e.g. the email-popup redesign rec). The compound prototype is overkill for single-element comparisons.

Read `templates/README.md` for the full how-to-use guide.

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

- **Top of widget:** tab row for page selection (PDP A / PDP B / PLP / Cat landing / Homepage / etc.). For single-page prototypes (just one canvas), skip the tab row entirely.
- **Below tabs:** control panel listing the recs applicable to the current page (filtered by `applies_to_pages`), in **page-position order** — the rec whose targeted element sits highest on the page is rec #1 on that panel; the rec whose element sits lowest is the last. Future-build / Gamechanger recs (if any) are always placed at the END of the order, regardless of where their element sits on the page. Each rec is rendered as a **detailed card** (not a simple pill) — see 6.3.1 below.
- **Above the rec list:** a sticky summary bar showing `Recommendations · X / Y on` for the current tab. Top-right of the summary bar: `Defaults` button (restores the per-tab default-on set), `Reset all` button (turns every rec OFF).
- **Below controls:** the phone frame, ~420px wide (or `.desktop-wrap` if desktop target), with a fixed chrome and scrollable inner content area. Re-renders on every toggle change.

### 6.3.1 — Rec card spec (the rail)

Each rec card in the rail must include all of the following, in this order top-to-bottom:

- **Header row:**
  - `Number` — the rec's global index. Numbers do NOT renumber between page tabs — a rec keeps the same number across every page-tab it appears in.
  - `Title` — short, bold, 1 line. Prefix with `★ ` for Gamechanger / `future_build: true` recs.
  - `Future-build badge` (right-aligned) — a small "Future build" pill, only rendered for `future_build: true` recs.
  - `Toggle switch` (right-aligned) — small pill (30×18px). ON state uses `rec_toggle_on_color` from the style profile (default `#7c5cff`). Gamechanger / future-build cards use `gamechanger_toggle_on_color` (default `#d4a500`, gold) when ON, so they read as distinct bets.
- **Brief** — 1–2 sentence description of the change (`recommendation` field, verbatim).
- **Why line** — thin top divider, then `Why:` label followed by the heuristic + audience signal in one sentence (`why` field, verbatim).
- **"As seen on" line** — thin top divider, then `As seen on:` followed by comma-separated competitor citations. Each citation is the competitor domain, rendered as a clickable link (`target="_blank"`) to the specific URL where that pattern is visible. Source: the rec's `competitor_pattern_observed` URL(s) collected in Step 4b. For `differentiation` recs (no competitor pattern), write `Differentiation — no direct competitor pattern` and optionally a closest-analog citation.

The whole card is clickable — clicking anywhere on the card flips the toggle.

Visual style (override only if the user explicitly asks):
- Rec card: white background, 0.5px `#e0e0e0` border, 8px radius, 11px 13px padding.
- Rec card ON: light tint of `rec_toggle_on_color` for background, full color for border.
- Gamechanger card ON: light gold tint background, gold border.
- Future-build card: dashed border (so it reads as visually distinct in the rail even when OFF).
- Title: 13px 600 weight. Brief: 11.5px regular `#555`. Why line: 11px `#888`. "As seen on" line: 10.5px italic `#888`, links in `rec_toggle_on_color`.

Do NOT put impact metrics, expected lift, supporting stat, or ICE scores in the rec card itself — those live in the XLSX (Step 7) and in the per-rec text export (Step 9). The card is for quick scanning; the XLSX is the working document.

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

## Step 7 — Build implementation-tracker XLSX

Generate a companion spreadsheet at `outputs/<client-slug>/implementation-tracker.xlsx` so the client can manage rollout. Use the `xlsx` skill to build it.

### 7.1 Sheet structure

ONE sheet named "Recommendations" — one row per approved rec, in **rec-number order** (same numbering as the HTML prototype). Future-build / Gamechanger recs are at the end of the list.

Columns, in this exact order:

| # | Column | Type | Notes |
|---|---|---|---|
| 1 | **#** | integer | The rec number — matches the prototype rail. Freeze this column on the left. |
| 2 | **Recommendation** | string | A terse 1–5 word label. NOT the full one-sentence recommendation — that goes in `Details`. Examples: "Sticky ATC", "Trust badges", "Cadence visualization". |
| 3 | **Details** | string | The full `recommendation` sentence from the rec metadata. Wrap text. |
| 4 | **Category** | string | UI/UX · Strategic · Gamechanger. |
| 5 | **Page** | string | PDP · PLP · Cart · Homepage · Category · Other. Derived from `page_type_affected`. |
| 6 | **Impact** | integer 1–10 | From `ice.impact`. |
| 7 | **Confidence** | integer 1–10 | From `ice.confidence`. |
| 8 | **Ease** | integer 1–10 | From `ice.ease`. |
| 9 | **ICE Total** | integer | From `ice.total` (impact × confidence × ease). Apply a 3-color conditional-formatting scale: red (low) → yellow → green (high). |
| 10 | **Priority** | string | High / Medium / Low — derived from ICE Total quartiles within this deck. Top 25% = High; middle 50% = Medium; bottom 25% = Low. |
| 11 | **Risk** | string | "Ship" / "Test" / "Research-first" — directly from the feasibility classification in Step 4c. Color-coded cell: Ship = green tint, Test = amber tint, Research-first = grey tint. |
| 12 | **Implementation Status** | string | Dropdown with these values, in this order: `Not started` (default for every row), `Shared with client`, `Awaiting decision`, `Approved`, `In progress`, `Implemented`, `A/B testing`, `Won (kept)`, `Lost (rolled back)`, `Inconclusive`, `Dropped`. Apply data validation to enforce the dropdown. Color-code: Implemented / Won = green tint; In progress / A/B testing = amber tint; Dropped / Lost = grey tint; Not started = no fill. |
| 13 | **Implementation Date** | date | Blank by default. Format: `YYYY-MM-DD`. Apply date validation. |
| 14 | **Owner** | string | Who owns the implementation (e.g. "Client design team", "Client dev", "Agency"). Blank by default. |
| 15 | **Comment** | string | Free-text. For agency or client to note feedback, blockers, test results. Wrap text; column width ~60 chars. |

### 7.2 Sheet formatting

- Header row: bold, white text, dark fill matching `colors.primary` from `style-profile.json` (fall back to `#1a1a1a` if none).
- Freeze first row + first column.
- Body: `font-family: Inter, Arial, sans-serif`, 11pt.
- Auto-fit column widths with manual caps: Details = 45 chars, Comment = 60 chars (wrap text).
- Zebra striping on data rows (light grey alternate).
- ICE Total: 3-color scale conditional formatting.
- Risk and Implementation Status: cell-fill color coding per the values above.

### 7.3 Optional Summary sheet

If the deck has 10+ recs, add a second sheet named "Summary" with:

- Total rec count broken down by category (UI/UX / Strategic / Gamechanger)
- Distribution by Risk (Ship / Test / Research-first counts)
- Distribution by Page (counts per page type)
- Average ICE score
- A bar chart of ICE Total per rec (chart inside the sheet, not a separate image)

Keep the Summary sheet minimal — Recommendations is the working document; Summary is a one-glance health-check.

### 7.4 Quality gate before saving

- Row count = approved rec count, no extras, no blanks.
- Implementation Status defaults to "Not started" on every row.
- Implementation Date is blank on every row (the client fills it in over time).
- All ICE columns are integers, not strings.
- The Recommendation column is terse (max ~5 words). If you wrote the full sentence there by mistake, rewrite it to the short label and move the sentence to Details.
- File opens cleanly in Excel, Google Sheets, and Numbers.

---

## Step 8 — Hand-off

Deliver both files directly to the user. No .pptx is produced — the HTML prototype IS the deliverable, and the XLSX is the working document for the client to manage rollout.

1. Provide `computer://` links to both files:
   - `outputs/<client-slug>/prototype.html` — the interactive prototype
   - `outputs/<client-slug>/implementation-tracker.xlsx` — the implementation tracker
2. Tell the user how to share them: "Download both files from the links. The HTML opens in any browser on any device — attach to email or drop into Slack. The XLSX is the client's working document for tracking rollout status."
3. Offer a short client-facing email template (~5 lines) they can adapt:

   > Hi all,
   >
   > Given the [metric trend / context], we took another pass at the [page / site] to find areas to sharpen the communication, raise perceived value, and improve conversion.
   >
   > Attached are two files:
   >
   > - **The interactive prototype** (.html) — please download and open in your browser. It recreates your current site with toggleable recommendations in the left rail; flip each rec on/off to see the page update live. Multiple page tabs at the top cover the PDP / PLP / homepage / etc.
   > - **The implementation tracker** (.xlsx) — a working spreadsheet to manage rollout. Every rec is pre-filled with ICE scores, risk (Ship vs Test), and an Implementation Status dropdown. Update it as decisions are made; we'll review periodically together.
   >
   > A quick caveat: the prototype is directional — won't match production design 100%. It communicates intent, not final pixels.
   >
   > Let us know which ones are feasible to ship and if anything raises questions. Happy to jump on a call to walk through it together.

4. Offer to iterate: "Want to add, drop, or rephrase any recs? Adjust the prototype defaults? Recolor anything in the XLSX?"

### Final quality gate

Before declaring done:

**HTML prototype**

- The HTML file exists at the expected path and opens cleanly in a browser
- Every rec card has full metadata (number, title, brief, why, as seen on, toggle, future-build badge if applicable)
- Every toggle visibly changes the targeted element
- Every page tab applies its defaults correctly; counter is accurate
- No fabricated content — every product name, price, USP, review count is from the audit
- No icon-font CDN dependencies (inline SVG only — Tabler/FontAwesome CDNs have failed in the past)
- File size is reasonable (<500KB; if larger, you've over-embedded — trim)

**XLSX tracker**

- The XLSX file exists at the expected path and opens cleanly in Excel + Google Sheets
- Row count = approved rec count, in the same numbering as the prototype
- Implementation Status defaults to "Not started" everywhere
- Recommendation column is terse (1–5 words); full sentence is in Details
- Dropdown validation applies to Status; conditional formatting applies to ICE Total + Risk + Status

---

## Step 9 — Per-rec text export (on demand)

Independent deliverable the user can request at any point after Step 5 (rec confirmation). Produces the full per-rec copy — Recommendation, Why, Impact, Expected lift, Supporting stat, Implementation note, Heuristics, ICE — in the format the deck's left rail expects.

This step exists because clients frequently want the text content separately from the prototype: to paste into Notion, build their own slide deck, hand to a copywriter, ticket as Asana cards, or review offline.

### Two delivery modes — ask the user which they want

**Mode A — Direct to chat**

Format and paste the rec text into the chat response. Use this when:
- The user is reviewing recs interactively
- They want to copy-paste a single rec into another tool
- They asked for the text in chat ("send me the copy for slide N")

Format per rec:

```
**Rec N — [Title]**

**Recommendation**
[recommendation field — full prose]

**Why**
[why field — full prose]

**Impact**
- [impact_metrics bullet 1]
- [impact_metrics bullet 2]
- [impact_metrics bullet 3]

**Expected lift**: [expected_lift]
**Supporting stat**: [supporting_stat.stat] — [supporting_stat.source] ([supporting_stat.source_url])  ← omit this line if supporting_stat is null
**Implementation**: [implementation_note]  ← omit if null
**Heuristics**: [heuristics_cited joined by " · "]
**ICE**: [impact] / [confidence] / [ease] ([total]) · [feasibility_label]
**Competitor reference**: [competitor_url_for_reference]
```

If the user asks for multiple recs at once, render each as a separate block separated by `---`.

**Mode B — To a markdown file**

Write all recs to `outputs/<client-slug>/recommendations.md`. Use this when:
- The user wants the full set as a deliverable file
- They asked for a "file" / "document" / "export"
- The deck is the final form but they want a structured text version alongside

File structure:

```markdown
# [Client name] — CRO recommendations

[N] recommendations · Generated [date]

## Section 1 — UI/UX Recommendations

### Rec 1 — [Title]
[full per-rec block, same format as Mode A but without the surrounding bolding cues]

### Rec 2 — [Title]
...

## Section 2 — Strategic Recommendations

### Rec N — [Title]
...

## Sources
- [competitor URL 1]
- [competitor URL 2]
...
```

After writing, share `computer://` link to `recommendations.md`.

### How to choose

Default to asking the user explicitly: "Do you want the rec text in chat, or saved to a file you can open?" One short clarifying question saves wasted output.

If they ask for "all recs" / "the full list" / "a document," default to Mode B (file).
If they ask for "the copy for rec N" / "rec X text" / "send me," default to Mode A (chat).

### Source of truth

All text content comes from `outputs/<client-slug>/recommendations-validated.json`. Do not paraphrase or rewrite the fields — copy them verbatim. If a field is empty or null, omit the corresponding line entirely rather than inventing content.
