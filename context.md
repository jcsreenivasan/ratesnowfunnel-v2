# ratesnowfunnel — Project Context

## What this is
A single-file HTML/CSS/JS prototype of a multi-step mortgage rate comparison funnel for the brand **Rates,Now**. Target audience is general mortgage seekers (veterans are welcome but content is generic). No build tools, no frameworks, no backend.

## Files
| File | Description |
|------|-------------|
| `index.html` | The entire prototype — all HTML, CSS, JS in one file (~1260 lines) |
| `logo.png` | rates.now logo (copied from Downloads/ratesnlogo.png) |
| `ca.png` | Camouflage background image (light gray/white camo, fixed to body) |
| `Screenshots/` | Reference Bankrate funnel screenshots (source of structure/content) |

## Brand / Design
- **Colors:** Purple `#5B5BD6`, Dark purple `#1E1B4B`, Gold `#F59E0B`, White `#FFFFFF`
- **Background:** `ca.png` camo image, `background-attachment: fixed`, covers all pages
- **Panels:** `background: rgba(255,255,255,0.94)` + `backdrop-filter: blur(8px)` for frosted glass effect
- **Font:** Inter (Google Fonts CDN)

## Funnel Flow (14 steps)
| Step | ID | Content |
|------|----|---------|
| 1 | s1 | ZIP code |
| 2 | s2 | Property type (4 icon cards) |
| 3 | s3 | Property use (3 icon cards) |
| 4 | s4 | First-time buyer Yes/No |
| 5 | s5 | Homebuying stage (4 list options) |
| 6 | s6 | Purchase price (slider) |
| 7 | s7 | Down payment (3 list options) |
| 8 | s8 | Credit score (10-option grid) |
| 9 | s9 | Employment status (6-option grid) |
| 10 | s10 | Bankruptcy / foreclosure Yes/No |
| 11 | s11 | Annual income (slider) |
| 12 | s12 | Lender matches (3 cards + 3 hidden extras via Show more) |
| 13 | s13 | Contact info form |
| 14 | s14 | Thank you / confirmation |

## Key JS Patterns
```js
// Navigation
go(stepNumber)        // hide current, show target step
pick(el, stepId, next) // select option, auto-advance after 280ms delay
exitFunnel()          // return to landing page
beginFunnel()         // hide landing, show funnel at s1

// Progress bar
const progress = { 1:4, 2:15, ... 14:100 }  // step → % complete

// Lenders
toggleLender(card)    // toggle .sel class on lender card
showMoreQuotes()      // reveal 3 hidden extra lender cards in #lender-grid, hide #show-more-btn
```

## SVG Icon Library
All icons are defined once as `<symbol>` elements in a hidden `<svg>` at the top of `<body>`, then referenced with `<use href="#icon-name"/>`. Icon IDs:
- Property types: `icon-house`, `icon-townhome`, `icon-condo`, `icon-multifamily`
- Property use: `icon-primary`, `icon-secondhome`, `icon-investment`
- Yes/No: `icon-yes`, `icon-no`
- Value props: `icon-target`, `icon-rates`, `icon-shield`
- Trust/UI: `icon-sparkle`, `icon-lock`, `icon-users`
- Landing buttons: `icon-purchase`, `icon-refinance`
- Avatars: `icon-avatar1`, `icon-avatar2`, `icon-avatar3`
- Loan programs (unused in flow, kept in defs): `icon-firsttime`, `icon-veteran`, `icon-fha`, `icon-conventional`

## Sections on Landing Page
1. Hero (loan type selector → begins funnel)
2. Rate bar (today's 30yr fixed rate display)
3. Media strip (featured by WSJ, Forbes, etc.)
4. Value props grid (3 cards)
5. FAQ accordion
6. Reviews (Google/Zillow/X ratings + 3 review cards)
7. Footer

## Sections Below Each Funnel Step
- Reviews section (same as landing)
- FAQ accordion (condensed version)
- Footer

## Things Removed vs. Early Versions
- No trust bar ("Get matched… / Trusted & secure since 1976 / +400,000 people…")
- No Step 9 "Do you qualify for any special loan programs?" (removed per user request)
- No VA-specific / military-specific loan language in funnel steps
- No pre-selected options on any screen

## Option Selection Behavior
- `pick()` calls remove `.sel` from all siblings, add `.sel` to clicked element, then auto-advance after 280ms
- Lender cards use `toggleLender()` (toggle, not auto-advance) — user must click Continue
- No options are pre-selected on page load

## Responsive
- `@media (max-width: 640px)`: stacks hero buttons, value grid → 1 col, lender grid → 1 col, form row → 1 col
