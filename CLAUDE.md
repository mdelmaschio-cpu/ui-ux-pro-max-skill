# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Antigravity Kit** is an AI-powered design intelligence toolkit providing searchable databases of UI styles, color palettes, font pairings, chart types, and UX guidelines. It functions as an installable skill/workflow for AI coding assistants (Claude Code, Windsurf, Cursor, and 15+ others).

**Key stats:**
- 67 UI styles, 161 color palettes, 57 font pairings, 161 product types
- 99 UX guidelines, 25 chart types, 1,924 Google Fonts indexed
- 161 reasoning rules mapping product types → design systems
- 16 tech stacks, 18 supported AI platforms
- ~6,500 total searchable CSV rows

**npm package:** `uipro-cli` (v2.5.0) — binary: `uipro`

---

## Search Command

```bash
python3 src/ui-ux-pro-max/scripts/search.py "<query>" [options]
```

### Domain search

```bash
python3 src/ui-ux-pro-max/scripts/search.py "<query>" --domain <domain> [-n <max_results>]
```

| Domain | Description |
|---|---|
| `product` | Product type recommendations (SaaS, e-commerce, portfolio) |
| `style` | UI styles (glassmorphism, minimalism, brutalism) + AI prompts and CSS keywords |
| `typography` | Font pairings with Google Fonts imports |
| `color` | Color palettes by product type |
| `landing` | Page structure and CTA strategies |
| `chart` | Chart types and library recommendations |
| `ux` | Best practices and anti-patterns |
| `icons` | Icon library reference |
| `react` | React-specific performance guidelines |
| `web` | Web/app interface guidelines |
| `google-fonts` | Full Google Fonts catalog (1,924 fonts) |

Omit `--domain` for auto-detection based on query keywords.

### Stack search

```bash
python3 src/ui-ux-pro-max/scripts/search.py "<query>" --stack <stack>
```

Available stacks: `html-tailwind` (default), `react`, `nextjs`, `vue`, `svelte`, `astro`, `nuxtjs`, `nuxt-ui`, `swiftui`, `react-native`, `flutter`, `shadcn`, `jetpack-compose`, `threejs`, `angular`, `laravel`

### Design system generation

```bash
# Generate a full design system recommendation
python3 src/ui-ux-pro-max/scripts/search.py "<project description>" --design-system

# With project name and markdown output
python3 src/ui-ux-pro-max/scripts/search.py "SaaS dashboard" --design-system \
  --project-name "MyApp" --format markdown

# Persist to design-system/MASTER.md (Master + Overrides pattern)
python3 src/ui-ux-pro-max/scripts/search.py "SaaS dashboard" --design-system --persist

# Create a page-specific override
python3 src/ui-ux-pro-max/scripts/search.py "landing page" --design-system \
  --persist --page landing

# Custom output directory
python3 src/ui-ux-pro-max/scripts/search.py "fintech app" --design-system \
  --persist --output-dir ./my-project
```

### All flags reference

| Flag | Short | Default | Description |
|---|---|---|---|
| `--domain` | `-d` | auto | Search domain |
| `--stack` | `-s` | `html-tailwind` | Tech stack |
| `--max-results` | `-n` | `3` | Max results returned |
| `--json` | | false | Output as JSON |
| `--design-system` | `-ds` | false | Generate complete design system |
| `--project-name` | `-p` | | Project name for design system output |
| `--format` | `-f` | `ascii` | Output format: `ascii` or `markdown` |
| `--persist` | | false | Save to `design-system/MASTER.md` |
| `--page` | | | Create page-specific override in `design-system/pages/` |
| `--output-dir` | `-o` | `.` | Output directory for persisted files |

---

## Architecture

```
src/ui-ux-pro-max/                    # Source of Truth — edit here only
├── data/                             # Canonical CSV databases
│   ├── styles.csv                    # 67 UI styles with CSS keywords + AI prompts
│   ├── colors.csv                    # 161 product-type color palettes
│   ├── typography.csv                # 57 font pairings with Google Fonts URLs
│   ├── products.csv                  # 161 product types with design recommendations
│   ├── ui-reasoning.csv              # 161 reasoning rules (product → design system)
│   ├── ux-guidelines.csv             # 99 UX best practices + anti-patterns
│   ├── charts.csv                    # 25 chart types with library recommendations
│   ├── landing.csv                   # Landing page patterns and CTA strategies
│   ├── icons.csv                     # Icon library reference
│   ├── google-fonts.csv              # 1,924 indexed Google Fonts
│   ├── react-performance.csv         # React-specific performance guidelines
│   ├── app-interface.csv             # Web interface guidelines
│   ├── design.csv                    # Design reference data
│   └── stacks/                       # Stack-specific guidelines (16 files)
│       ├── html-tailwind.csv, react.csv, nextjs.csv, vue.csv
│       ├── svelte.csv, astro.csv, nuxtjs.csv, nuxt-ui.csv
│       ├── swiftui.csv, react-native.csv, flutter.csv
│       ├── shadcn.csv, jetpack-compose.csv
│       └── threejs.csv, angular.csv, laravel.csv
├── scripts/
│   ├── search.py                     # CLI entry point (all flags, domain routing)
│   ├── core.py                       # BM25 + regex hybrid search engine
│   └── design_system.py             # Design system generator with reasoning rules
└── templates/
    ├── base/
    │   ├── skill-content.md          # Common SKILL.md content (all platforms)
    │   └── quick-reference.md        # Quick reference section (Claude only)
    └── platforms/                    # 18 platform configs (*.json)
        ├── claude.json, cursor.json, windsurf.json, copilot.json
        ├── kiro.json, roocode.json, kilocode.json, augment.json
        ├── agent.json, codebuddy.json, codex.json, continue.json
        ├── droid.json, gemini.json, opencode.json, qoder.json
        └── trae.json, warp.json

cli/                                  # CLI installer (uipro-cli on npm)
├── src/
│   ├── index.ts                      # CLI entry point (Commander.js)
│   ├── commands/
│   │   ├── init.ts                   # Install command (platform detection + template gen)
│   │   ├── versions.ts               # List available versions
│   │   ├── update.ts                 # Update to latest
│   │   └── uninstall.ts              # Remove skill from platform
│   └── utils/
│       ├── detect.js                 # AI platform auto-detection
│       ├── template.js               # Template rendering engine
│       ├── extract.js                # ZIP extraction utilities
│       ├── github.js                 # GitHub release API integration
│       └── logger.js                 # Logging utilities
├── assets/                           # Bundled assets (~564KB, synced from src/)
│   ├── data/                         # Copy of src/ui-ux-pro-max/data/
│   ├── scripts/                      # Copy of src/ui-ux-pro-max/scripts/
│   └── templates/                    # Copy of src/ui-ux-pro-max/templates/
└── package.json                      # version 2.5.0, deps: commander chalk ora prompts

.claude/skills/                       # Claude Code skills (local development)
├── ui-ux-pro-max/                    # Primary skill (symlinks to src/)
├── banner-design/                    # Banner generation skill
├── brand/                            # Brand identity skill
├── design/                           # Logo, icons, CIP skill
├── design-system/                    # Design system generation skill
├── slides/                           # HTML presentation skill
└── ui-styling/                       # UI styling with shadcn/ui skill

.claude-plugin/                       # Claude Marketplace publishing
├── plugin.json                       # Marketplace metadata (name, version, skill ref)
└── marketplace.json                  # Marketplace ID + owner

.factory/skills/ui-ux-pro-max/       # Droid (Factory) skill (symlinks to src/)
.shared/ui-ux-pro-max/               # Symlink to src/ui-ux-pro-max/
```

The search engine uses BM25 token ranking combined with regex scoring. Domain auto-detection fires when `--domain` is omitted. The design system generator aggregates results across multiple domains using the 161 reasoning rules in `ui-reasoning.csv`.

---

## Sync Rules

**Source of Truth:** `src/ui-ux-pro-max/` — always edit here, never in copies.

| What changed | Action required |
|---|---|
| `data/*.csv` or `data/stacks/*.csv` | None — symlinks propagate changes automatically |
| `scripts/*.py` | None — symlinks propagate changes automatically |
| `templates/base/*.md` or `templates/platforms/*.json` | None — symlinks propagate changes automatically |
| CLI publishing | Run sync command below before `npm publish` |

**CLI asset sync (before publishing):**
```bash
cp -r src/ui-ux-pro-max/data/* cli/assets/data/
cp -r src/ui-ux-pro-max/scripts/* cli/assets/scripts/
cp -r src/ui-ux-pro-max/templates/* cli/assets/templates/
```

**Reference folders** (`.claude/`, `.factory/`, `.shared/`) — no manual sync needed. The CLI generates these from templates at `uipro init` time.

---

## CLI Development

The CLI is built with TypeScript and Bun.

```bash
cd cli

# Install dependencies
bun install

# Build
bun run build

# Test locally
./dist/index.js init

# Link globally for testing
npm link
uipro init
```

---

## Prerequisites

- **Python 3.x** — no external dependencies (stdlib only)
- **Bun** — for CLI development and build
- **Node.js** — for `npm publish` of `uipro-cli`

---

## CI/CD

`.github/workflows/` contains three workflows:

| File | Trigger | Purpose |
|---|---|---|
| `claude.yml` | `@claude` mention in issues/PRs | Claude Code AI actions via `anthropics/claude-code-action@v1` |
| `claude-code-review.yml` | PR opened/updated | Automated code review |
| `python-package-conda.yml` | Push/PR | Python data file CI |

The Claude workflow requires permissions: `contents:read`, `pull-requests:read`, `issues:read`, `id-token:write`, `actions:read`.

---

## Git Workflow

Never push directly to `main`. Always:

1. Create a branch: `git checkout -b feat/...` or `fix/...`
2. Commit changes with descriptive messages
3. Push branch: `git push -u origin <branch>`
4. Open a PR — `gh pr create` if `gh` CLI is available

Branch naming conventions:
- `feat/<description>` — new features or data additions
- `fix/<description>` — bug fixes
- `chore/<description>` — maintenance (sync, version bumps)
