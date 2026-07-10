# glowmotion

Premium animated technical diagrams as **single self-contained HTML+SVG files**
— a Claude skill.

glowmotion is the merge of two earlier experiments:

- **[dashmotion](https://github.com/csthink/dashmotion)** contributed the
  engineering core: a deterministic pure-stdlib layout engine (longest-path
  layering, orthogonal box-safe routing, nested boundaries), the
  one-self-contained-HTML output contract, and mechanized verification
  (`check_diagram.py` C1–C10 structural checks, `check_fidelity.py` Mermaid
  label fidelity).
- **lanshu-animated-architecture-diagram** contributed the look: glowing comet
  dots with fading trails, pulsing module highlights, film grain + vignette
  finish, icon glyphs, the title highlight capsule and signature slot — all
  reimplemented here as vector SVG/CSS/SMIL instead of pre-rendered GIF
  frames, so the premium look costs kilobytes, scales losslessly, can be
  paused, and respects `prefers-reduced-motion`.

## What you get

- **Two modes** — `flow` (flowcharts, state machines) and `architecture`
  (systems, topologies, nested boundaries), plus Mermaid input
  (`flowchart`/`graph`, `stateDiagram-v2`) with verbatim-label fidelity
  checking.
- **Three themes** — `midnight` (deep navy / emerald), `neon` (black canvas,
  lanshu green/purple/cyan/amber, grain + vignette), `aurora` (teal/violet).
- **A real animation layer** — flowing dashed connectors, per-journey glow
  dots with trails, staggered pulsing halos on the nodes a request touches.
- **Determinism + verification** — the engine owns every coordinate; the
  bundled checkers are the delivery gate (including C10: a diagram whose
  animation layer went missing fails).

## Install

glowmotion is a [Claude skill](https://code.claude.com/docs/en/skills) — install
it once and Claude Code picks it automatically whenever you ask for an animated
diagram.

**With the skills CLI (recommended):**

```bash
npx skills add shangliy/glowmotion       # this project only (.claude/skills/)
npx skills add -g shangliy/glowmotion    # global, all projects (~/.claude/skills/)
```

**Manually (clone + symlink):**

```bash
git clone https://github.com/shangliy/glowmotion.git
ln -s "$(pwd)/glowmotion" ~/.claude/skills/glowmotion
```

Start a new Claude Code session and ask for e.g. *"an animated architecture
diagram of our checkout system, neon theme"* — or invoke it explicitly with
`/glowmotion`. To upgrade later: `npx skills update` (CLI install) or
`git pull` (manual install). To remove: delete the symlink/skill folder.

## Usage

```bash
# author a semantic graph JSON (see references/graph-format.md), then:
python3 scripts/layout.py graph.json --render diagram.html
python3 scripts/check_diagram.py diagram.html          # must print 0 violations
python3 scripts/check_fidelity.py src.mmd diagram.html # mermaid input only
```

Demo outputs live in `examples/` (`arch-neon.html`, `flow-aurora.html`,
`mermaid-midnight.html`) — open them in any browser.

## Requirements

`python3` (stdlib only). No pip installs, no fonts to install, no network.
