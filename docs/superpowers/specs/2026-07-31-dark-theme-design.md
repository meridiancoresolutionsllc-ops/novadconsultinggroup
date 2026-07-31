# Dark Theme — Design Spec

## Goal
Convert the site from its current light theme to a permanent dark theme, using the existing brand palette (rose, gold, wine, black) — no light/dark toggle, no new hues introduced. Applies to both `index.html` and `client-resources.html`, which share an (almost) identical `:root` token block and consistently style components through those tokens.

## Approach
Token remap: redefine the *surface* and *text* CSS custom properties in `:root` to dark equivalents. Leave the brand accent tokens (`--rose`, `--gold`, `--wine`, `--black`, `--darkest`, and their `-light`/`-border` variants) unchanged, since they already read as dark, rich accent colors and don't need to shift. Patch the small number of hardcoded (non-variable) light colors by hand.

**Source of the dark values:** commit `4b11e92` ("Refactor styles and update color scheme in index.html") is a previous dark version of this same site, from before it was restyled light. Its `:root` already defines `--rose`, `--rose-light`, `--wine`, `--gold`, `--gold-light`, `--black` at the exact same hex values used today, plus a proven dark surface/text scale (`--dark`, `--dark2`, `--dark3`, `--white` as off-white text, `--gray`, `--muted`). Rather than inventing new dark values, the remap below sources from that commit wherever a role lines up, instead of guessing new hex values from scratch.

Rejected alternatives:
- A `.dark` toggle class/layer — unnecessary, no toggle is wanted, would add dead CSS.
- Manual per-rule rewrite of every color declaration — higher effort and higher risk of missed spots than remapping the shared tokens the components already consume.
- Inventing a new dark palette from scratch — commit `4b11e92` already proves out a dark scale for this exact brand palette; reusing it is lower-risk than guessing new values.

## Token Remap

| Token | Current (light) | New (dark) | Source in `4b11e92` |
|---|---|---|---|
| `--white` (primary surface) | `#ffffff` | `#100B0D` | `--black` (body bg) |
| `--off-white` (alt surface) | `#FAFAF9` | `#181114` | `--dark` (`.results`/`.testimonials` section bg) |
| `--gray-light` (tertiary surface) | `#F2F0EF` | `#201619` | `--dark2` (card backgrounds) |
| `--gray-mid` (borders/dividers) | `#E4E0DE` | `#281C20` | `--dark3` |
| `--ink` (primary text) | `#1A1214` | `#FBF6F2` | `--white` (body text color) |
| `--ink-mid` (secondary emphasis) | `#4A3840` | `#DCD2D5` | interpolated between `--gray` and `--white` — no direct equivalent existed |
| `--ink-muted` (body/secondary text) | `#7A6870` | `#C7BBBE` | `--gray` (FAQ answers, resource lists) |
| `--ink-faint` (tertiary/faint text) | `#A89AA0` | `rgba(248,240,236,.64)` | `--muted` |
| `--border` (hairline border) | `rgba(26,18,20,.1)` | `rgba(248,240,236,.12)` | analogous hairline; token is unused today so no direct source |
| `--rose-pale` (hover/panel fill) | `#FAF0F2` (light cream) | `rgba(217,141,155,.10)` | matches the commit's actual hover-fill technique |
| `--gold-pale` (hover/panel fill) | `#FAF6EE` (light cream) | `rgba(201,168,107,.10)` | matches `.btn-outline-gold:hover { background: rgba(201,168,107,.09) }` |

Unchanged (already identical in `4b11e92`): `--rose #D98D9B`, `--rose-light #EFB4BE`, `--wine #241318`, `--gold #C9A86B`, `--gold-light #E9D6B0`, `--black #100B0D`, `--darkest #0A0608`, `--gold-border`, `--rose-border`.

`--rose-pale`/`--gold-pale` change role from "light cream background fill" to "dark translucent tint fill" — same brand hue, adapted so hover states and panels (`.btn-outline:hover`, `.btn-outline-gold:hover`, `.guarantee-row`, `.portal-card`) don't render as bright light patches on a dark page. Text/border colors used inside those panels (e.g. `--ink-muted`, `--gold-border`) already resolve correctly under the new dark tokens.

**Extra authentic touch:** `4b11e92` deliberately uses `#0A0608` (i.e. this repo's `--darkest`) for the marquee strip specifically to read darker than surrounding sections. Carry that over: `.marquee-wrap` switches from `background: var(--white)` to `background: var(--darkest)`, in addition to the token remap.

`client-resources.html` did not exist at commit `4b11e92`, so it has no dark version to source from directly — it gets the same remapped token set applied to its own (near-identical) `:root` block for consistency with `index.html`.

## Hardcoded exceptions to patch by hand
Both files contain a few colors written as literals instead of tokens, found via grep for `#fff`, `#000`, and `rgba(255,255,255,...)`:

- `nav { background: rgba(255,255,255,.96); }` → `rgba(16,11,13,.92)` (rgb of `--black`/new `--white`, matching `4b11e92`'s nav which used `rgba(16,11,13,.88)`), in both files.
- `nav.scrolled { box-shadow: 0 2px 24px rgba(26,18,20,.08); }` (index.html only) → darken/adjust so the scroll shadow still reads against a dark nav (e.g. `rgba(0,0,0,.35)`).
- `.founder-card::after` label strip: `background: rgba(255,255,255,.9)` with `border-top: 1px solid var(--gray-mid)` → `rgba(16,11,13,.85)` dark translucent background so the "Founder" label strip isn't a light patch over the dark theme.
- `.cr-logo { background: #000; }` — already dark, no change needed.
- Remaining `#fff` / `rgba(255,255,255,...)` occurrences (button text, text on `--wine`/`--black`/`--rose` surfaces like `.q-author span`, `.cta-band-text`, `.about-stat-card p`, `.video-caption`, `.price-card.popular` text, `.product-cover span`, `.final-cta p`) are text-on-already-dark-surface and stay as-is — they're correct in both the old and new theme.

## Out of scope
- No toggle/switch between light and dark.
- No new colors beyond the existing rose/gold/wine/black/darkest family.
- No content or layout changes — colors only.
- Images (`hero-image.png`, `founder-photo.jpeg`, etc.) are not edited.

## Verification
After implementation, visually spot-check both pages (or a static preview) for: text contrast against new dark surfaces, the nav bar over page content on scroll, and the `.founder-card` label strip legibility.
