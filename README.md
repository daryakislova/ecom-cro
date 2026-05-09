# Ecom CRO Plugin

End-to-end CRO recommendation deck builder for ecom clients. From client brief to a ready-for-Claude-Design output package, in one slash command.

## What it does

You run `/cro-deck` and the workflow walks Claude through:

0. **Brand intake** — once per session, you provide your own visual style (drop an existing deck, drop a brand guide, or answer a few quick questions). The plugin ships brand-neutral; every team brings their own look.
1. **Context** — parses the client brief, extracts audience / products / competitors / blockers, asks four clarifying questions
2. **Research** — picks audience-tuned UX heuristics from the library + audits the client website (PDPs, PLPs, homepage, cart drawer)
3. **Ideation** — generates ICE-scored, hypothesis-formatted recommendations with expected lift ranges and full metadata
4. **Self-validation** — three independent passes: audience fit, competitor scan (advisory not gating), Ship/Test/Research-first classification
5. **Confirmation** — you approve / edit the rec list
6. **Screenshots** — captures pristine iPhone 16 Pro screenshots of current state + competitor references
7. **Deck brief** — writes `deck-brief.md`, `claude-design-prompt.txt`, `metadata.json`, and a `screenshots/` folder ready to hand off to Claude Design

Output: a complete folder you attach to a new Claude Design session. Claude Design produces the final `.pptx` styled in your brand.

## Folder structure (after install)

```
ecom-cro/
├── .claude-plugin/plugin.json     ← manifest
├── commands/cro-deck.md           ← /cro-deck — the entire workflow
└── skills/
    └── cro-heuristics/SKILL.md    ← reference library: NN/g + Cialdini + ecom UX patterns
```

Three files. The slash command owns the workflow end-to-end; the heuristic library is a separate file because it's a reference you'll edit independently (adding new audience signals, new patterns) without touching the workflow logic. **No brand assets ship with the plugin** — every user supplies their own at runtime.

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

Brand style is captured per session at runtime — no plugin file to edit.

## Known limitations

- Mobile-only screenshots by default. Web is supported but desktop framing differs.
- Standard Shopify checkout audit is skipped by default. Flag in Step 1 ("client is on Shopify Plus with Checkout Extensibility") to include it.
- Geofenced or auth-walled competitor sites cannot be auto-screenshotted — the workflow falls back to asking you to drop screenshots manually.
- Expected lift ranges are heuristic-based unless a public stat is cited. Don't treat them as forecasts.
