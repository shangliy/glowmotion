# Graph format — `scripts/layout.py`

`layout.py` (pure Python stdlib, zero deps) computes all geometry — layering,
row packing, boundary padding, orthogonal C2-safe routing — **and renders the
finished HTML** with the theme, the glow/trail/halo animation layer, icons,
legend, and your copy. You decide semantics and wording; the script writes the
file.

> Treat `layout.py` as a black box — do NOT read its source. This file is the
> complete contract. If something is unclear, run it on a tiny graph and look
> at the output.

```
python3 <skill-dir>/scripts/layout.py graph.json --render out.html  # deliverable
python3 <skill-dir>/scripts/layout.py graph.json                    # geometry JSON (inspection)
```

Write `graph.json` to a temp path (`$TMPDIR`/`mktemp`), never the output folder.

## Input — the semantic graph JSON you author

```json
{
  "mode": "flow | architecture",
  "theme": "midnight | neon | aurora",
  "title": "short header title",
  "titleHighlight": "capsule phrase (optional; renders as a colored pill after the title)",
  "subtitle": "one-line description under the title (optional)",
  "signature": "@byline (optional; top-right of the header)",
  "summary": [
    {"accent": "cyan|violet|rose", "title": "card title", "items": ["bullet", "bullet"]}
  ],
  "footer": "footer line (optional)",
  "icons": true,
  "effects": true,
  "nodes": [
    {
      "id": "A", "label": "verbatim label", "sublabel": "arch 2nd line, optional",
      "shape": "pill | decision",
      "type": "frontend|backend|database|cloud|security|bus|external",
      "tier": 0,
      "group": "GROUP_ID",
      "semStroke": "#3b82f6", "semDash": "6 3"
    }
  ],
  "edges": [{ "from": "A", "to": "B", "kind": "sync|async|main|static", "label": "optional" }],
  "groups": [{ "id": "VPC", "label": "AWS VPC", "kind": "region|subnet", "parent": "OUTER_ID?" }],
  "journeys": [{ "color": "#22d3ee", "hops": [["A","B"], ["B","C"]] }],
  "legendExtra": [{ "label": "verbatim legend entry", "stroke": "#f59e0b", "dash": "2 2" }]
}
```

## Authoring rules

- **Flow:** omit `tier`. Write `shape` **only** for pills (entry/exit) and
  decisions (branch nodes); omit it for steps — `"shape": "step"` is wasted
  output. Back/self edges are detected automatically and render as a
  `↻ label` annotation, never a drawn return path.
- **Architecture:** `tier` (0 = top row) is **all-or-nothing** — write it on
  every node or on none. Omit for ungrouped / single-top-level-group diagrams
  (the engine auto-layers by longest-path). Write it on every node when the
  diagram has >1 top-level group (auto-layering is group-blind and tears
  boundaries — C9 catches it) or a cross-cutting sink must sit at a specific
  row. Clean-boundary contract when you do set tiers: each tier belongs to one
  top-level group; a subgroup owns its tiers (no loose sibling on the same
  tier); a multi-tier group's tiers are consecutive; nesting ≤ 2 levels.
- **Edge kinds:** `sync` → animated solid; `async` → dotted, own keyframes,
  theme async color; `main` → 1.5px emphasis; `static` → plain line, no
  marker, no animation.
- **Journeys — always author 1–4.** Hops must be existing forward edges.
  Journeys are what animate: each becomes a glowing dot with two fading trail
  dots riding the exact connector path, and every node a journey touches gets
  a pulsing halo on a staggered delay. Omit `color` and the theme's accent
  cycle is applied (first journey matches the legend's "request in flight"
  dot). A diagram with zero journeys **fails the checker** (C10); more than ~8
  total hops-worth of dots also fails — dots mark informative paths, not every
  edge.
- **Copy:** write `title`/`subtitle` always; `titleHighlight` for the capsule
  when the title has a natural emphasis phrase; in architecture write a
  `summary` of exactly three cards. Keep Mermaid-sourced labels and
  `legendExtra` entries verbatim — the fidelity check compares exactly.
- **Knobs:** `icons` (default true) draws a small type glyph in each arch
  node's corner. `effects` (default true) is the whole premium layer — glow,
  trails, halos, sparkles, grain/vignette; set false for a minimal look.
  `semStroke`/`semDash` preserve a source classDef stroke variant; pair them
  with a matching `legendExtra` entry.

## Themes

| theme | canvas | vibe | finish |
|---|---|---|---|
| `midnight` | deep navy `#020617` | emerald flow, cyan dots — clean docs look | none |
| `neon` | pure black `#000` | lanshu green/purple/cyan/amber, high drama | grain + vignette |
| `aurora` | deep slate `#030712` | teal/violet borealis | vignette |

Unknown theme falls back to `midnight` with a stderr notice; invalid color/dash
tokens fall back to engine defaults the same way (fail-safe-to-render). A
structurally invalid graph (unknown edge endpoint, group parent cycle, missing
ids) fails fast with a readable list instead (fail-closed).

## Output

`--render out.html` writes the complete, self-contained deliverable: every
coordinate and path `d`, the theme style layer, the animation layer (dash flow
+ comet dots + halos), icons, legend (type swatches + "request in flight" +
`legendExtra`), summary cards, pause toggle, reduced-motion script, ARIA
title/desc wiring. Run `check_diagram.py` (and `check_fidelity.py` for Mermaid
input) on it; fix by editing the JSON and re-rendering.

Without `--render` the script prints the geometry JSON (nodes with x/y/w/h,
edges with `d`, group boxes, journey hop paths) — for inspection only.
