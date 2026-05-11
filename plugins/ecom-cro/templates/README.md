# Widget templates

Reusable HTML prototypes for `/cro-deck`. Both are designed for `mcp__visualize__show_widget` (live rendering inside Cowork chat) but also work standalone if saved as `.html` files.

## When to use which

| Template | When to use |
|---|---|
| `prototype-widget.html` | The primary deliverable — full site recreation with **per-rec toggles** so the client can flip each fix on/off and watch the design compound. One widget covers PDP variants + PLP + category landing + homepage. |
| `current-vs-suggested.html` | Element-level redesigns (popups, single components, individual modules). One **Current/Suggested toggle** at the top, both desktop + mobile views shown side by side or stacked. |

## prototype-widget.html

Phone-wrap pattern: outer `.phone-wrap` 294×588px, inner `.phone` 420×840px scaled `0.7` via `transform: scale()`, square corners (`border-radius: 0`). Tabs across the top switch between client pages (PDP A/B/C, PLP, category landing, homepage). Pill controls below tabs toggle individual recs on/off — each rec keyed by ID (`r3`, `r4`, etc.) that maps to `recommendations-validated.json` slide numbers. Hash `applicableRecs()` shows only the recs that affect the current page.

Mobile-only by default. If the client needs desktop framing, swap `.phone-wrap` for a `.desktop-wrap` and adjust inner dimensions.

The visible chrome includes browser top-bar (`figlinensandhome.com`-style URL), announcement strip, sticky header with serif wordmark, and a sticky bottom popup. The popup uses IntersectionObserver to appear when the Delivery section is reached and hide when the Footer is reached — this is the standard luxury-PDP sticky-ATC pattern.

Each rec toggle is a single boolean in `state`. The render function rebuilds page HTML from scratch each time a toggle flips — no diffing, simpler than React.

## current-vs-suggested.html

Two views stacked vertically: **Desktop** (16:10 frame with brand chrome + dimmed page + popup centered) and **Mobile** (phone frame with same content scaled down). Toggle pills at top switch between Current and Suggested state — both views update together.

Use this when the rec is about ONE element (a popup, a banner, a single module). The compound prototype-widget pattern is overkill for single-element comparisons.

Pattern principles baked in:
- Centered modal on mobile, NOT bottom sheet — bottom sheets create dismissal uncertainty for older users.
- Serif headline, sans-serif body — matches luxury-tier convention.
- Single CTA, full-width.
- Trust line optional — see `skills/popup-copy/SKILL.md` for when to include and when to skip.

## How agents should use these

1. Read the template file from disk to get the working HTML.
2. Replace placeholders (CLIENT_BRAND, brand colors, product names, rec list) with values from the current session's `client-context.json` + `recommendations-validated.json`.
3. Pass the populated HTML to `mcp__visualize__show_widget` as `widget_code`.
4. If the user wants the widget saved to disk for sharing, also write it to `outputs/<client-slug>/prototype.html`.

Both templates use only CSS variables + system-ui font + Times New Roman serif. No external dependencies.
