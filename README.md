# Ecom CRO Plugin

End-to-end CRO recommendation builder for ecom clients. From client brief to an interactive HTML prototype your client can play with — plus optional .pptx handoff package.

## What it does

You run `/cro-deck` and the workflow walks Claude through:

0. **Brand intake** — once per session, you provide your own visual style (drop an existing deck, drop a brand guide, or answer a few quick questions). The plugin ships brand-neutral; every team brings their own look.
1. **Context** — parses the client brief, extracts audience / products / competitors / blockers, asks four clarifying questions
2. **Research** — picks audience-tuned UX heuristics from the library + audits the client website (PDPs, PLPs, homepage, cart drawer)
3. **Ideation** — generates ICE-scored, hypothesis-formatted recommendations with expected lift ranges and full metadata
4. **Self-validation** — three independent passes: audience fit, competitor scan (advisory not gating), Ship/Test/Research-first classification
5. **Confirmation** — you approve / edit the rec list
6. **Interactive HTML prototype** — builds a near-1:1 recreation of the client's key pages (PDP, PLP, category landing, homepage, footer) with toggleable rec controls so you can flip each fix on/off and watch the design compound toward the Suggested state. **DOM-first**: opens the live site in a browser tab, extracts real structure and computed styles via `read_page` + `javascript_tool`, and recreates each page from that data. Phone screenshots are a fallback only — used when Cloudflare/anti-bot blocks the site, the page is geofenced or auth-walled, or responsive behavior can't be reproduced from DOM inspection. The plugin always tries the live site first.
7. **Deck brief + handoff package** — writes `prototype.html`, `prototype-rec-map.json`, `deck-brief.md`, `claude-design-prompt.txt`, `metadata.json` — share the prototype URL directly with the client OR hand the package to Claude Design for an optional .pptx.
8. **Hand-off** — Path A (recommended): share `prototype.html` directly with the client. Path B (optional): generate `.pptx` via Claude Design.
9. **Per-rec text export (on demand)** — at any point after Step 5 the user can ask for the rec text (Recommendation / Why / Impact / Expected lift / Heuristics / ICE) either pasted into chat (single rec, slide-format) or saved to `recommendations.md` (full set). Source: `recommendations-validated.json`, verbatim.

Primary output: an interactive HTML prototype the client can interact with directly. Optional output: static .pptx via Claude Design hand-off (less reliable; recommended only if the client specifically expects a deck). Either path can be paired with the per-rec text export.

## Folder structure (after install)

```
ecom-cro/
├── .claude-plugin/plugin.json     ← manifest
├── commands/cro-deck.md           ← /cro-deck — the entire workflow
├── templates/
│   ├── README.md                  ← when to use which template
│   ├── prototype-widget.html      ← full-site recreation with per-rec toggles
│   └── current-vs-suggested.html  ← element-level redesign comparison
└── skills/
    ├── cro-heuristics/SKILL.md    ← reference library: NN/g + Cialdini + ecom UX patterns
    └── popup-copy/SKILL.md        ← luxury 45+ email-popup copy & design principles
```

The slash command owns the workflow end-to-end. The reference libraries (`cro-heuristics`, `popup-copy`) are separate files because they're references you'll edit independently — adding new audience signals, new patterns, new copy principles — without touching the workflow logic. The `templates/` folder holds two reusable HTML widget patterns the `/cro-deck` workflow plugs into. **No brand assets ship with the plugin** — every user supplies their own at runtime.

## Install

In Cowork desktop:

1. Open Settings → Plugins
2. Click **Add marketplace** → **GitHub**
3. Enter `daryakislova/ecom-cro` and click Sync
4. Find the `ecom-cro` plugin → click Install

The marketplace lives at `https://github.com/daryakislova/ecom-cro`.

For Claude Code CLI install:

```bash
claude plugin marketplace add daryakislova/ecom-cro
claude plugin install ecom-cro@ecom-cro-marketplace
```

## Use

In any chat session: `/cro-deck`

You'll be asked first for your brand style (Step 0), then for the client brief and four clarifying questions before any work starts.

## Customization

Want to update something across all future decks?

- **Workflow** (steps, validators, mix rules, slide layout, brief format) → edit `commands/cro-deck.md`
- **Heuristic library** (add audience signals, new patterns) → edit `skills/cro-heuristics/SKILL.md`
- **Popup / newsletter copy rules** (audience-tuned copy + design principles) → edit `skills/popup-copy/SKILL.md`
- **Widget templates** (prototype scaffold, current-vs-suggested scaffold) → edit `templates/*.html`

Brand style is captured per session at runtime — no plugin file to edit.

## Known limitations

- Mobile-only screenshots by default. Web is supported but desktop framing differs.
- Standard Shopify checkout audit is skipped by default. Flag in Step 1 ("client is on Shopify Plus with Checkout Extensibility") to include it.
- Geofenced or auth-walled competitor sites cannot be auto-screenshotted — the workflow falls back to asking you to drop screenshots manually.
- Expected lift ranges are heuristic-based unless a public stat is cited. Don't treat them as forecasts.
