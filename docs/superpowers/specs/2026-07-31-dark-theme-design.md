# Dark Theme — Design Spec

## Goal
Convert the site from its current light theme to a permanent dark theme, using the existing brand palette (rose, gold, wine, black) — no light/dark toggle, no new hues introduced. Applies to both `index.html` and `client-resources.html`, which share an (almost) identical `:root` token block and consistently style components through those tokens.

## Approach
Token remap: redefine the *surface* and *text* CSS custom properties in `:root` to dark equivalents. Leave the brand accent tokens (`--rose`, `--gold`, `--wine`, `--black`, `--darkest`, and their `-light`/`-border` variants) unchanged, since they already read as dark, rich accent colors and don't need to shift. Patch the small number of hardcoded (non-variable) light colors by hand.

Rejected alternatives:
- A `.dark` toggle class/layer — unnecessary, no toggle is wanted, would add dead CSS.
- Manual per-rule rewrite of every color declaration — higher effort and higher risk of missed spots than remapping the shared tokens the components already consume.

## Token Remap

| Token | Current (light) | New (dark) | Role |
|---|---|---|---|
| `--white` | `#ffffff` | `#170F12` | primary surface (cards, nav, hero copy) |
| `--off-white` | `#FAFAF9` | `#1D1317` | alternating section surface |
| `--gray-light` | `#F2F0EF` | `#241A1E` | tertiary surface (footer, hero visual) |
| `--gray-mid` | `#E4E0DE` | `#3D2E35` | borders/dividers |
| `--ink` | `#1A1214` | `#F5EFF1` | primary text |
| `--ink-mid` | `#4A3840` | `#D8C7CD` | secondary emphasis text |
| `--ink-muted` | `#7A6870` | `#B3A0A8` | body/secondary text |
| `--ink-faint` | `#A89AA0` | `#8A7680` | tertiary/faint text |
| `--border` | `rgba(26,18,20,.1)` | `rgba(245,239,241,.12)` | hairline border |
| `--rose-pale` | `#FAF0F2` (light hover fill) | `rgba(217,141,155,.10)` | subtle rose tint fill (hover/panel bg) |
| `--gold-pale` | `#FAF6EE` (light hover fill) | `rgba(201,168,107,.10)` | subtle gold tint fill (hover/panel bg) |

Unchanged: `--rose #D98D9B`, `--rose-light #EFB4BE`, `--wine #241318`, `--gold #C9A86B`, `--gold-light #E9D6B0`, `--black #100B0D`, `--darkest #0A0608`, `--gold-border`, `--rose-border`.

`--rose-pale`/`--gold-pale` change role from "light cream background fill" to "dark translucent tint fill" — same brand hue, adapted so hover states and panels (`.btn-outline:hover`, `.btn-outline-gold:hover`, `.guarantee-row`, `.portal-card`) don't render as bright light patches on a dark page. Text/border colors used inside those panels (e.g. `--ink-muted`, `--gold-border`) already resolve correctly under the new dark tokens.

## Hardcoded exceptions to patch by hand
Both files contain a few colors written as literals instead of tokens, found via grep for `#fff`, `#000`, and `rgba(255,255,255,...)`:

- `nav { background: rgba(255,255,255,.96); }` → dark translucent (e.g. `rgba(10,6,8,.9)`), in both files.
- `nav.scrolled { box-shadow: 0 2px 24px rgba(26,18,20,.08); }` (index.html only) → darken/adjust so the scroll shadow still reads against a dark nav (e.g. `rgba(0,0,0,.35)`).
- `.founder-card::after` label strip: `background: rgba(255,255,255,.9)` with `border-top: 1px solid var(--gray-mid)` → dark translucent background so the "Founder" label strip isn't a light patch over the dark theme.
- `.cr-logo { background: #000; }` — already dark, no change needed.
- Remaining `#fff` / `rgba(255,255,255,...)` occurrences (button text, text on `--wine`/`--black`/`--rose` surfaces like `.q-author span`, `.cta-band-text`, `.about-stat-card p`, `.video-caption`, `.price-card.popular` text, `.product-cover span`, `.final-cta p`) are text-on-already-dark-surface and stay as-is — they're correct in both the old and new theme.

## Out of scope
- No toggle/switch between light and dark.
- No new colors beyond the existing rose/gold/wine/black/darkest family.
- No content or layout changes — colors only.
- Images (`hero-image.png`, `founder-photo.jpeg`, etc.) are not edited.

## Verification
After implementation, visually spot-check both pages (or a static preview) for: text contrast against new dark surfaces, the nav bar over page content on scroll, and the `.founder-card` label strip legibility.
