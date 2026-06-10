# ratesnowfunnel

VA mortgage rate comparison funnel for **varates.now / rates.now**.
Live: **https://ratesnowfunnel.vercel.app**

---

## Overview

Single-page application — all HTML, CSS, and JavaScript live in one file: `index.html`. No build system, no dependencies, no package.json.

---

## Project Structure

```
ratesnowfunnel/
├── index.html                  # Entire app — HTML + CSS + JS
├── VArateslogo.png             # Header logo
├── ca.png                      # Body background texture (repeating camo)
├── american-flag.png           # Flag icon used in VA rate bar
├── veterans-united-logo.png    # Lender card logo (s13)
├── better-mortgage-logo.svg    # Lender card logo (s13)
├── rocket-logo.png             # Lender card logo (s13)
└── README.md
```

`index.html` has three logical sections in order:
1. `<style>` — all CSS (~530 lines)
2. `<body>` — SVG icon library + landing page + funnel HTML
3. `<script>` — all JS at the bottom (~160 lines)

---

## Running Locally

No build step needed. Just open `index.html` in a browser:

```bash
open index.html
```

Or serve it with any static server:

```bash
npx serve .
# or
python3 -m http.server 8080
```

---

## Deployment (Vercel)

Vercel CLI is installed locally at `/tmp/vercel-local`. If it's missing, reinstall:

```bash
npm install --prefix /tmp/vercel-local vercel --cache /tmp/npm-cache
```

Deploy to production:

```bash
/tmp/vercel-local/node_modules/.bin/vercel --prod
```

---

## Architecture

### Landing Page (`id="landing"`)

Shown on first load. Sections in order:
- **Header** — `VArateslogo.png` logo (clicking returns to landing from funnel)
- **Hero** — headline, two CTA buttons (Buy a Home / Refinance)
- **VA Rate Bar** — today's rate display with American flag icon
- **Proof strip** — 3 trust highlights
- **Reviews** — platform badges + 6 review cards
- **FAQ** — accordion (6 items)
- **Footer** — legal disclaimer + links

### Funnel (`id="funnel"`)

Hidden until a CTA button is clicked. Contains:
- Fixed progress bar (top of page)
- Step cards (only one visible at a time)
- Reviews section (repeated below steps)
- FAQ section (condensed)
- Footer

---

## Funnel Step System

All step cards: `<div class="step-card hidden" id="sN">` (buy) or `id="rsN"` (refi). Only the active card has `hidden` removed.

### Buy Flow
```
s1 → s2 → s3 → s4 → s5 → s6 → s7 → s8 → s9 → s10 → s11 → s12 → s13 → s14 → sotp → s15
```

| ID | Question |
|----|----------|
| s1 | ZIP code |
| s2 | Property type (icon grid) |
| s3 | Property use → `pickPropertyUse()` |
| s4 | First-time homebuyer (Yes/No) |
| s5 | Homebuying stage (list) |
| s6 | Purchase price slider ($100K–$5M) |
| s7 | Down payment slider (0–100%) |
| s8 | Credit score slider (560–850) |
| s9 | Employment status (grid) |
| s10 | Branch of service (icon grid, 7 options incl. "I Did Not Serve") |
| s11 | Bankruptcy/foreclosure (Yes/No) |
| s12 | Annual income slider ($0–$500K) |
| s13 | Lender matches (select up to 3) |
| s14 | Contact form + SMS consent checkboxes |
| sotp | 4-digit phone verification |
| s15 | Thank you + selected lender logos |

### Refi Flow
```
s1 → s2 → s3 → rs4 → rs5 → rs6 → rs7 → rs8 → rs9 → s8 → s9 → s10 → s11 → s12 → s13 → s14 → sotp → s15
```

| ID | Question |
|----|----------|
| rs4 | Estimated property value slider ($100K–$5M) |
| rs5 | Remaining mortgage balance (0–100%, shows $) |
| rs6 | Current interest rate slider (0–12%) |
| rs7 | Second mortgage (Yes/No) |
| rs8 | Additional cash slider ($0–$250K) |
| rs9 | Late mortgage payments (No / 1 Late / 2+) |

---

## Key JavaScript Functions

```js
beginFunnel(type)         // 'buy' or 'refi' — hides landing, shows funnel at s1
exitFunnel()              // returns to landing page
go(step)                  // step = number (→ 's'+step) or string ('rs4', 'sotp')
pick(el, stepId, next)    // selects option, auto-advances after 280ms
pickPropertyUse(el)       // flow-aware: go('rs4') for refi, go(4) for buy
continueWithOffers(count) // 'all' selects all visible lenders; 1 selects first only
buildTyLenders()          // renders selected lender logos on the thank you page
toggleLender(card)        // toggles .sel on a lender card (manual selection)
showMoreQuotes()          // reveals 3 hidden lender cards in s13
syncSlider(rangeId, displayId)  // syncs range input value → formatted text input
syncDownPayment()         // computes down payment $ from % and purchase price
syncRemainingBalance()    // computes remaining balance $ from % and property value
syncInterestRate()        // syncs interest rate slider → text input
syncCreditScore()         // updates score display + rating label (Fair/Good/etc.)
toggleFaq(btn)            // accordion open/close
```

### Navigation State

```js
let curStep = 1;        // current step — number (buy) or string (refi / sotp)
let flowType = 'buy';   // 'buy' or 'refi'
```

### Progress Bar Maps

```js
// Buy flow — keyed by step number or string
const progress = { 1:4, 2:15, ..., 14:92, 'sotp':96, 15:100 };

// Refi flow — keyed by string ID
const refiProgress = { 's1':4, 's2':10, ..., 's14':96, 'sotp':98, 's15':100 };
```

---

## CSS Patterns & Design Tokens

### Color tokens (`:root`)
| Token | Value | Usage |
|-------|-------|-------|
| `--primary` | `#5349DB` | Primary purple |
| `--primary-dk` | `#3d33c4` | Hover state |
| `--primary-lt` | `#ece9ff` | Light purple bg |
| `--ink` | `#0f0e1a` | Body text |
| `--slate` | `#6b6b8a` | Secondary text |
| `--green` | `#16a25b` | Positive/savings |
| `--lborder` | `rgba(83,73,219,0.12)` | Card borders |
| `--purple` | `#5B5BD6` | Funnel UI |
| `--purple-dark` | `#1E1B4B` | Buttons |
| `--purple-bg` | `#EBEBFB` | Selected state bg |

### Input wrappers
```html
<!-- Dollar prefix -->
<div class="inp-dollar">
  <span class="inp-dollar-sym">$</span>
  <input class="inp" type="text">
</div>

<!-- Percentage suffix -->
<div class="inp-pct">
  <input class="inp" type="text">
  <span class="inp-pct-sym">%</span>
</div>
```

### Option types
| Class | Usage |
|-------|-------|
| `.opts-yn` / `.opt-yn` | Yes/No two-column |
| `.opts-list` / `.opt-list` | Full-width stacked list |
| `.opts-grid` / `.opt-grid` | Two-column grid |
| `.opts-icon` / `.opt-icon-card` | Icon cards (2-col default, add `.cols3` for 3-col) |

Selected state: `.sel` class added by `pick()` or `continueWithOffers()`.

### Breakpoints
| Breakpoint | Target |
|-----------|--------|
| `≤860px` | Tablet |
| `≤640px` | Mobile |
| `≤390px` | Small phones |

---

## SVG Icon Library

All icons are `<symbol>` elements in a hidden `<svg>` at the top of `<body>`, referenced with `<use href="#icon-name">`.

| ID | Description |
|----|-------------|
| `icon-house` | Single family home |
| `icon-townhome` | Townhome |
| `icon-condo` | Condominium |
| `icon-multifamily` | Multi-family |
| `icon-primary` | Primary residence |
| `icon-secondhome` | Second home |
| `icon-investment` | Investment/rental |
| `icon-yes` | Check circle |
| `icon-no` | X circle |
| `icon-target` | Matching (target) |
| `icon-rates` | Rates (tag) |
| `icon-shield` | Security shield |
| `icon-sparkle` | Thank you star |
| `icon-lock` | Security lock |
| `icon-users` | Users |
| `icon-purchase` | Purchase CTA |
| `icon-refinance` | Refinance CTA |
| `icon-avatar1/2/3` | Review avatars |

---

## Key Element IDs

| ID | Step | Purpose |
|----|------|---------|
| `landing` | — | Landing page wrapper |
| `funnel` | — | Funnel wrapper (toggled with `.active`) |
| `pbar` / `ppct` | — | Progress bar fill + percent label |
| `zip` | s1 | ZIP code input |
| `price-val` / `price-range` | s6 | Purchase price |
| `dp-range` / `dp-pct-label` / `dp-dollar-val` | s7 | Down payment |
| `cs-range` / `cs-display` / `cs-rating` | s8 | Credit score |
| `income-range` / `income-val` | s12 | Annual income |
| `lender-grid` | s13 | Lender cards container |
| `show-more-btn` | s13 | Show more quotes link |
| `sms-consent-1` / `sms-consent-2` | s14 | SMS consent checkboxes |
| `otp-group` | sotp | OTP digit inputs container |
| `ty-lenders` | s15 | Dynamically rendered lender logos |
| `prop-val` / `prop-range` | rs4 | Property value |
| `rb-range` / `rb-pct-label` / `rb-dollar-val` | rs5 | Remaining balance |
| `ir-range` / `ir-val` | rs6 | Interest rate |
| `cash-range` / `cash-val` | rs8 | Additional cash |

---

## Notes for Developers

- **No backend** — the OTP screen is UI-only; wire up a real SMS provider (Twilio, etc.)
- **Lender data** — hardcoded in s13; replace with a dynamic API call for real lender matching
- **Form submission** — s14 contact form has no submission handler; integrate your lead capture endpoint in the `go('sotp')` call or the OTP verify step
- **Phone formatting** — no mask applied to the phone input; add one if needed
- **Consent storage** — SMS checkbox state (`#sms-consent-1`, `#sms-consent-2`) is not persisted; capture values before navigating away from s14
