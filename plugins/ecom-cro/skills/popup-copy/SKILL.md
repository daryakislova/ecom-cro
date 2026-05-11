---
name: popup-copy
description: |
  Best-practice copy and design principles for email-capture popups and inline
  newsletter slots, calibrated for the audience the client describes. Use whenever
  generating, validating, or critiquing newsletter / email-capture surfaces in
  a CRO recommendation. The principles below are tuned for older, affluent buyers
  (luxury 45+) but the framework adapts — read the audience block first, then
  pick the matching rules.
---

# Email-capture popup copy & design principles

## How to use

When the client brief identifies an older or affluent audience (luxury, considered-purchase categories, 45+ skew), apply the "Luxury 45+" rules below. When the brief identifies a younger DTC audience, apply the "DTC 25–45" rules. When mixed, default to Luxury 45+ — its rules are more conservative and won't actively hurt the younger cohort, whereas the DTC rules can damage brand perception with the older cohort.

## The principal decision: should a popup exist at all?

Before designing one, ask which pattern fits the audience:

| Pattern | When it fits | Capture vs. interruption |
|---|---|---|
| **Modal popup** | DTC, mid-market, list-growth-driven priorities | Highest capture, highest interruption |
| **Slim sticky bar** (bottom of viewport, non-blocking) | Most cases — good middle ground | ~50–70% of modal capture, minimal interruption |
| **No popup, inline only** | Luxury tier where brand positioning matters more than list size (Sferra, Matouk, Frette pattern) | ~20–30% of modal capture, zero interruption |

For luxury 45+, the strongest direct competitors (Sferra, Matouk, Frette, Peacock Alley, Yves Delorme) do NOT run modals. If the client wants a popup anyway, redesign rather than remove — but flag the brand-positioning tradeoff in the rec text.

## Luxury 45+ rules

### Copy

1. **Lead with the specific offer.** "10% off your first order" beats "Sign up for exclusive offers." Specificity reads as honesty.

2. **Anchor with brand names the audience already trusts.** For a luxury linens retailer, naming Sferra / Matouk / Yves Delorme in the subline is stronger authority than any abstract claim about quality.

3. **Cap copy above the input at 20 words.** Older eyes scan, they don't read. Eyebrow + headline + one subline.

4. **Truthful claims only.** Don't promise a "Linen Care Guide" the brand doesn't publish, "VIP access" that doesn't exist, "early access" without an actual early-access program. Overpromise is the #1 trust killer for considered-purchase older buyers.

5. **Avoid the word "spam" even in a reassurance line.** It plants the negative concept. Affluent buyers at this tier know how email works; the Unsubscribe link in the email footer is sufficient. Bloomingdale's, Saks, Neiman Marcus all omit any "no spam" line.

6. **No exclamation marks. No emoji. No countdown timers.** Mature register.

7. **Gracious tone — "Welcome to [Brand Name]" works.** "Sign up and save" reads transactional. "Hey friend! 👋" reads juvenile.

### Visual

1. **Serif headline** (Times New Roman, Georgia, or the client's editorial serif). Reads as traditional, established, considered.

2. **Sans-serif body and form** — usually system-ui or the client's UI sans.

3. **Minimum 16px body, 18–24px headline.** Contrast sensitivity drops measurably after 45 — small thin gray-on-cream is functionally hard to read.

4. **Two-column desktop modal**, image on the left, content on the right. Image is lifestyle/product, not a decorative pattern.

5. **Single navy / dark-tone CTA**, full-width, labeled with the verb ("SUBSCRIBE" or "JOIN") not an arrow icon. Older users routinely don't recognize iconographic CTAs.

6. **Visible X close button**, top-right of the modal, minimum 32px tap target on mobile.

### Mobile

1. **Centered modal, NOT bottom sheet.** This is the single most important mobile choice for this audience. Bottom sheets create dismissal uncertainty ("how do I close this?"). Centered modals match the desktop pattern, which matters for trust-driven older users.

2. **Same content stack as desktop**, just scaled — eyebrow, serif headline, one-line subline, email input, button. Image strip slim at top.

3. **Width ~220px in a 270–390px viewport.** Don't go full-bleed.

### Trigger

1. **Delayed**: 15–30 seconds, OR 2+ pageviews, OR exit-intent. Never instant.

2. **Suppress on cart and checkout.** Never interrupt a buyer in the funnel.

3. **Suppress for known subscribers** (cookie / email match).

4. **Do NOT reappear after dismissal.** Baymard's qualitative work ranked reappearing popups the #1 most-hated UX pattern for 55+.

### Form mechanics

- **One field only**: email. Adding name, phone, or preferences cuts capture by 30–60%.
- **Visible label OR good placeholder** — don't rely on disappearing placeholders alone for accessibility.
- **No confirmshame dismiss copy** ("No thanks, I hate linens"). Backfires hard for this audience — Baymard interviews ranked it among the most brand-damaging patterns.

## DTC 25–45 rules (briefly)

For younger audiences, looser rules apply: clever copy, emoji, casual register, gamified popups (wheel-of-fortune, scratch-to-reveal), exit-intent-only triggers, persistent badge / mailbox-icon patterns (Brooklinen-style) all work. Trigger can be earlier (5–10s). Bottom sheets are fine on mobile. The "no spam" trust line is acceptable. None of these should bleed into Luxury 45+ work.

## What to put in the rec

When writing the slide for an email-popup rec, the rec must include:

- **The exact copy** (eyebrow, headline, subline) as a string, with word count noted (e.g. "16 words above the input").
- **Layout dimensions** — desktop modal size, mobile modal size, image strip height.
- **Trigger logic** — time threshold, suppression rules.
- **Mobile pattern explicitly named** as centered modal (NOT bottom sheet).
- **What is NOT being promised** — for this audience, an explicit "Do NOT claim X, Y, Z" line in the rec text prevents downstream copy drift.
- **Heuristics cited**: NN/g #8 Aesthetic & minimalist · NN/g #2 Match between system and real world · Cialdini Authority. (Avoid citing Reciprocity for luxury — it's the wrong lever.)

## Reference

Built and validated during the Fig Linens & Home v0.3 engagement. See `outputs/fig-linens-and-home/recommendations-validated.json` slide 14 for the canonical rec template.
