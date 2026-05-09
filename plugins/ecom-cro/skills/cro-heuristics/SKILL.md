---
name: cro-heuristics
description: Provide the heuristic framework that grounds CRO recommendations in established UX/persuasion research. Use whenever generating, validating, or explaining ecom CRO recommendations. Surfaces relevant heuristics from Nielsen Norman's 10 Usability Heuristics, Cialdini's Principles of Persuasion, and standard ecom UX patterns (PDP, PLP, cart, checkout) — tuned to the specific audience. Each rec must cite at least one heuristic from this skill.
---

# CRO Heuristics Skill

This skill is a reference library. When called, return the relevant heuristics for the situation, then leave it to the calling agent (typically `ux-researcher` or `ideator`) to apply them.

**Critical rule:** Every recommendation must cite at least one heuristic name from below. Do not invent heuristics. If a recommendation doesn't fit any heuristic listed, say so explicitly and propose adding a documented external pattern with a public source URL.

---

## Layer 1 — Nielsen Norman 10 Usability Heuristics (universal floor)

1. **Visibility of system status** — keep users informed of what's happening (loading states, cart count, free-shipping progress bars, step indicators).
2. **Match between system and real world** — speak the user's language; avoid jargon, use familiar imagery.
3. **User control and freedom** — easy undo, clear back/close buttons, never trap users.
4. **Consistency and standards** — follow platform/category conventions (e.g. cart icon top-right, search top center).
5. **Error prevention** — eliminate error-prone conditions before they happen (smart defaults, input masks, format hints).
6. **Recognition rather than recall** — show options, don't force memory (recently viewed, breadcrumbs, persistent filters).
7. **Flexibility and efficiency of use** — accelerators for power users (saved addresses, one-tap reorder).
8. **Aesthetic and minimalist design** — every extra element competes for attention.
9. **Help users recognize, diagnose, and recover from errors** — clear error messages with the fix, inline.
10. **Help and documentation** — easy to find, task-oriented (not buried policies).

Cite by name and number, e.g. "NN/g #5 Error prevention".

---

## Layer 2 — Cialdini's Principles of Persuasion (motivation layer)

1. **Reciprocity** — give first (free samples, free shipping above threshold, gift cards, rewards).
2. **Commitment & Consistency** — small early yeses lead to big yes (newsletter signup → discount → purchase; build wishlist → checkout reminder).
3. **Social proof** — reviews, ratings, "X bought this", UGC, expert endorsements. Audience-specific format matters: numerical for trust-driven audiences, creator UGC for Gen Z, expert/editorial quotes for luxury.
4. **Authority** — credentials, awards, press logos, expert quotes, certifications.
5. **Liking** — relatable models, brand voice that matches audience, founder story.
6. **Scarcity** — limited stock, time-limited drops, exclusivity. **Use cautiously** — aggressive scarcity damages trust in luxury and older-female categories.
7. **Unity** — shared identity ("for parents like you", community language).

Cite as "Cialdini: Social proof", etc.

---

## Layer 3 — Ecom-specific UX patterns by page type

These are widely-documented patterns from the public ecom UX research community. Each agent applying them must judge audience fit before recommending.

### PLP (Product Listing Page)
- Filters: visible above the fold on mobile, sticky on scroll, multi-select, show applied filter chips
- Sort options: relevance, price asc/desc, newest, best-selling — labels matter (not "default")
- Product cards: primary image + name + price + ratings (star + count), hover/swipe alternate image
- "Quick view" or quick-add for low-consideration purchases; not for high-consideration
- Image consistency: same crop/background/aspect across cards
- Pagination: load-more for browsing audiences, paginated for comparison-heavy audiences
- Above-the-fold: avoid hero banners on PLP — push products up immediately

### PDP (Product Detail Page) — most important for CRO
- Hero image: large, zoomable, multiple angles, lifestyle + on-white
- Primary CTA (Add to Cart) above the fold on mobile, sticky on scroll
- Price clarity: list price + sale price + savings %, payment plan if applicable (Afterpay/Klarna for younger / mass; less prominent for luxury)
- Variant selectors: visual swatches for color, clear size selector, stock status per variant
- Trust signals near CTA: free shipping over $X, return window, warranty, secure checkout
- Reviews: average rating + count visible above fold, distribution chart, filterable, with photos
- Specs/materials: expandable sections, tabbed or accordion, never long scroll walls
- Cross-sell: "Complete the look" / "Frequently bought together" below CTA, NOT competing with it
- FAQ + sizing/fit info: addresses purchase obstacles for the specific category
- Recently viewed: bottom of page, helps comparison

### Cart drawer / mini-cart
- Slide-out, doesn't navigate away — preserves browsing flow
- Free-shipping progress bar (when applicable) — strong visual driver
- Suggested add-ons / bundles ("frequently bought with") — AOV lever
- Quantity adjust without leaving drawer
- Trust signals: secure checkout, guarantees
- Clear primary CTA "Checkout" — high contrast, full width, sticky bottom
- Avoid drawer-only cart for high-consideration purchases — full cart page may be better

### Homepage
- Hero: clear value prop in one line, audience-targeted imagery, single primary CTA
- Featured collections: 3–4 max, image-led, clear category labels
- Social proof: review carousel, press logos, UGC strip
- New arrivals / bestsellers
- Brand story: short, audience-aligned tone
- Email capture: secondary, not aggressive, with clear value

### Checkout (only if non-standard or Shopify Plus with Checkout Extensibility)
- Single-column layout
- Guest checkout option
- Real-time form validation
- Address autocomplete
- Multiple payment methods (incl. Apple Pay / Shop Pay / Klarna for relevant audiences)
- Order summary visible throughout (sticky on mobile)
- No surprise costs at the last step — show shipping/tax estimates earlier
- Trust signals at payment step

---

## Layer 4 — Audience-specific dial settings

Same heuristic, different execution. When the calling agent applies any heuristic above, it must tune by audience signal:

| Audience signal | Trust/social proof execution | Scarcity execution | Copy tone | Imagery | Payment options |
|---|---|---|---|---|---|
| Luxury, 45+ | Press logos, editorial quotes, "since 1998" | Avoid aggressive scarcity; "limited edition" OK | Calm, considered, confident | Lifestyle, refined, room context | Card primary; Klarna/Afterpay deemphasize |
| Young women streetwear/beauty | UGC, creator endorsements, review counts | Drops, "selling fast", time-limited | Friendly, fun, direct | Real customers, vibrant, on-body | Apple Pay, Shop Pay, BNPL prominent |
| Older male hobby/specialty | Expert reviews, technical specs, certifications | Inventory counts only when real | Authoritative, detailed, no fluff | Product detail, lifestyle in use | Card primary; PayPal welcomed |
| Parents | Other parents' reviews, safety badges | "Stock low" only when accurate | Reassuring, practical | Real families, age-appropriate | All major + BNPL |
| Wellness / health-conscious | Ingredient transparency, certifications, expert quotes | Avoid manufactured urgency | Calm, evidence-based, gentle | Natural, clean, real | Apple Pay, card |
| Tech-savvy / B2B-adjacent | Detailed specs, comparison tables, integrations | Specific availability dates | Direct, technical | Product UI, real environments | Card, invoice, NET-30 if relevant |

If the audience isn't on this list, infer the closest analog and state which heuristic dial settings you're applying and why.

---

## How to use this in a recommendation

Each recommendation cites:
- **Heuristic:** the specific named heuristic (e.g. "Cialdini: Authority" or "NN/g #5 Error prevention")
- **Audience tuning:** how it's been adapted for THIS audience
- **Optional public stat:** a citable percentage from a published source (Baymard public articles, Shopify research blog, Statista summary, etc.) — only cite real, public, linkable stats; do not fabricate

If there is no good citable stat, leave it out. Don't make up numbers.
