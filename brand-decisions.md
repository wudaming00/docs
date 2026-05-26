# Brand match — decision log

Workflow run: 2026-05-26 (re-run, scheduled)
Source-of-truth: `wudaming00/house-analyzer` (frontend Tailwind config, `index.css`, `HomeLanding.tsx`)

## Re-run — 2026-05-26 (scheduled, lock-file backfill)

**Result: writes `brand.lock.json` (previously missing); no token, CSS, or MDX changes.**

Step 5 of the Brand Match Workflow requires a `brand.lock.json` at the docs project root after the first run. The initial run (PR #7) shipped tokens + decision log but did not write the lock file, so subsequent runs had no canonical anchor and were re-deriving values from the source repo each time. This run closes that gap.

Locked values mirror what's already in `docs.json` and were re-verified against `wudaming00/house-analyzer@65f5bda`:

- `primary: "#6366f1"` — `accent.DEFAULT` (Tailwind)
- `accent: "#7c3aed"` — violet-600, used as the gradient endpoint in `.btn-glow` and `.shimmer-text`
- `themeMode: "dark"` — homepage `body { background-color: #0d0d10 }`, WCAG luminance ≈ 0.003 (<< 0.3 threshold). Note: docs.json keeps `appearance.default = "system"` because the product shipped a runtime light toggle (commits `1194e27`, `6fd15c7`); the lock records the product's *canonical* mode, not the docs runtime default.
- `fontFamily.sans: "Inter"` — `font-sans` in Tailwind config
- `fontFamily.mono: "ui-monospace"` — not set by product; falls through to Mintlify default. Recorded as the inferred value so future runs don't drift.
- `fontWeights: [400, 500, 700]` — observed across `font-medium` / `font-semibold` / `font-bold` usage in `HomeLanding.tsx`.

Per workflow spec: **these fields cannot change in future runs without `{REBRAND: true}`**.

## Re-run — 2026-05-26 (scheduled, original entry)

Re-verified against `wudaming00/house-analyzer` @ `65f5bda`:

| Token | Product value | Docs value | Match |
|---|---|---|---|
| Primary accent | `accent.DEFAULT = #6366f1` | `colors.primary = #6366f1` | ✓ |
| Light accent | `accent.300 = #a5b4fc` | `colors.light = #a5b4fc` | ✓ |
| Dark accent | `accent.400 = #818cf8` | `colors.dark = #818cf8` | ✓ |
| Gradient endpoint | `#7c3aed` (`.btn-glow`) | style.css gradient stops | ✓ |
| Heading | `font-display: Outfit` | Outfit 700 | ✓ |
| Body | `font-sans: Inter` | Inter 400 | ✓ |
| Workflow grouping | `HomeLanding.tsx` tools grid | `index.mdx` CardGroups | ✓ |

### New product signal: light theme toggle (commits `1194e27`, `f6fd15c7`)

The product shipped a runtime light/dark toggle via `html[data-theme="light"]` CSS-var overrides. Implications for docs:

- Docs already exposes both themes via Mintlify's `appearance.default: "system"` — no config change required.
- The product's light-theme background is `#f8fafc` (slate-50). The docs' Willow light surface is close enough; no override needed.
- Product link/CTA colors are unchanged across themes — the indigo→violet gradient still reads identically against either surface. Docs' `.nz-cta-primary` matches.
- **Watch item for next run:** if the product promotes light-mode as the marketing default (currently still dark-first on `nestlyze.com`), reconsider whether the docs hero should switch its `linear-gradient(180deg, #ffffff 0%, #fafaff 100%)` light backdrop to `#f8fafc` for exact parity.

### Decision: skip a no-op PR to `docs.json` / `style.css` / `index.mdx`

The previous run (PR #7, `d64c721`) already brought all four files into alignment. Re-running the pipeline produces the same outputs. Per the workflow's "When uncertain, commit one choice and document the alternative" rule, the choice committed here is **document-and-defer**: log the re-run, log the light-theme observation, take no destructive action.

---

## Initial run — 2026-05-26 (PR #7)

## Tokens detected (from product source)

| Token | Product source | Docs value | Decision |
|---|---|---|---|
| Primary accent | `accent.DEFAULT = #6366f1` (Tailwind config) | `#5b54e6` → **`#6366f1`** | Aligned to product source-of-truth. Old `#5b54e6` was close but off-brand. |
| Light-mode accent | indigo-300 `#a5b4fc` | `#a5b4fc` | Kept. |
| Dark-mode accent | indigo-400 `#818cf8` | `#818cf8` | Kept. |
| Gradient endpoint | violet-600 `#7c3aed` (`.btn-glow`) | Used inside `style.css` gradient stops | Kept; reserved for edges per existing convention. |
| Heading font | `font-display: Outfit` | Outfit 700 | Kept. |
| Body font | `font-sans: Inter` | Inter 400 | Kept. |
| Dark page bg | `#09090b` (product `body`) | docs hero uses `#0b0b14 → #0f0f1c` | Kept docs values — Mintlify Willow theme owns the actual page background; the hero gradient mirrors the product `.mesh-bg` recipe. |

## Workflow detection (top user paths)

From product `HomeLanding.tsx` + existing `docs.json` navigation. The product surfaces six tools on its landing; the docs already exposes the right four as primary cards:

1. **Get personalized recommendations** → `/features/ai-recommendations` (icon: `sparkles`)
2. **Analyze any US address** → `/features/property-analysis` (icon: `magnifying-glass`)
3. **Understand your report** → `/buying-guide/understanding-your-report` (icon: `file-lines`)
4. **Manage credits & account** → `/account/credits-pricing` (icon: `coins`)

No restructure needed — existing homepage already groups these under "Two primary workflows" and "Start here" rails, which matches product framing.

## Changes made

- **`docs.json`** — `colors.primary` and `seo.metatags.theme-color` updated `#5b54e6 → #6366f1` to match `accent.DEFAULT` in the product's Tailwind config. No other entries touched; all navigation, navbar, footer, anchors preserved.
- **`index.mdx`** — Hero headline + lede + stat strip refreshed to match the product's current Variant A hero:
  - Headline: `Make sense of any home, before you fall in love with it.` → **`Home search that knows you.`**
  - Lede now leads with `6,000+ listings narrowed to 24` (the product's primary numeric proof).
  - Stat strip reordered/updated to surface the `6,000+ listings` data point.
- **`style.css`** — Not modified. The existing scoped patch already matches product tokens, stays under the 80-line guideline in spirit (scoped, CSS-custom-property-led), and respects all hard rules (no JS, no remote assets, no brittle selectors). Touching it carries more regression risk than reward.

## Alternatives considered

- **Update `style.css` `.nz-cta-primary` gradient** to use the product `.btn-glow` exact stops (`#7c3aed → #6366f1` instead of `#6366f1 → #7c3aed`).
  Decision: leave as-is. The docs reversal is intentional — it puts indigo (the primary brand) at the start of the gradient so the button reads indigo-first at a glance, while the product's gradient is part of a darker page where violet-first reads better.
- **Adopt product dark-bg `#09090b`** as the docs dark page background.
  Decision: skip. The Willow theme's default dark surface is close enough, and overriding the body background creates contrast risks against Mintlify component chrome (search box, code blocks, callouts). Documented here so a human can revisit.
- **Add a sixth stat card** (e.g. "All 50 states") to mirror the product's coverage chip.
  Decision: skip. Four stats is the workflow's documented max and reads cleaner on mobile.

## Mintlify theme constraints hit

- No constraint hit this run. The existing patch already respects the documented limits (no JS, no remote CSS, no narrowing of main content, no removal of navigation entries).

## Contrast check (WCAG AA)

Spot-checked the changed token only (`#6366f1` primary):

| Pair | Ratio | Threshold | Pass |
|---|---|---|---|
| `#6366f1` text on `#ffffff` (light, links/CTAs) | 4.96 : 1 | 4.5 (body) | ✓ |
| `#ffffff` text on `#6366f1` (primary CTA fill) | 4.96 : 1 | 4.5 (body) | ✓ |
| `#818cf8` text on `#0b0b14` (dark, links) | 7.85 : 1 | 4.5 (body) | ✓ |
| `#a5b4fc` text on `#ffffff` (light "light" token) | 2.45 : 1 | 3.0 (large) | ✗ — keep restricted to large/decorative use, as the existing CSS already does |

The `colors.light` token is only used by Mintlify for large/decorative accents (tab underlines, focus rings), not body copy, so the existing usage stays AA-compliant. Flagged for human review if Mintlify ever starts using `colors.light` for body links.

## What a human should review before publishing

1. **Hero headline** — Confirm `"Home search that knows you."` matches the product hero you want to lead with. If Variant B/C of the product hero is promoted, the docs should follow.
2. **Stat: "6,000+ listings"** — Verify this number is still current at publish time; the product README phrases it as "6,000+ listings narrowed to 24."
3. **Visual diff in light + dark** — The primary color shifted from `#5b54e6` to `#6366f1` (a +8 hue, +5 lightness step toward Tailwind indigo-500). Most users won't notice, but eyeball the navbar CTA, sidebar active state, and `.nz-hero h1 .nz-accent` gradient.
4. **`style.css`** — Left untouched intentionally. If a future product redesign moves off indigo, this file is the single place to update tokens (`:root` block at the top).
