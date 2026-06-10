# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VA mortgage rate comparison funnel — **variation v2** (forked from `ratesnowfunnel`). Single-page application — all HTML, CSS, and JavaScript lives in one file: `index.html`. No build system.

## Deployment

```bash
/tmp/vercel-local/node_modules/.bin/vercel --prod --cwd /Users/sreenivasanjayachandrabalaji/Desktop/claudework/ratesnowfunnel-v2
```

Global `npm install -g vercel` is blocked by macOS permissions, so vercel is installed locally. If `/tmp/vercel-local` is missing, reinstall:

```bash
npm install --prefix /tmp/vercel-local vercel --cache /tmp/npm-cache
```

Live URL: **https://ratesnowfunnel-v2.vercel.app**

GitHub: **https://github.com/jcsreenivasan/ratesnowfunnel-v2**

Original project (do not modify): `ratesnowfunnel/` → https://ratesnowfunnel.vercel.app

## Architecture

`index.html` has three logical sections in order:
1. **`<style>`** — all CSS (~550 lines)
2. **`<body>`** — landing page + funnel HTML
3. **`<script>`** — all JS at the bottom (~130 lines)

### Landing page (`id="landing"`)
Shown first. Contains header (VArateslogo.png), hero headline, **property type selector cards** (4 cards: Single Family Home, Townhome, Condominium, Multi-Family Home), VA national average rate bar, proof strip (3 highlights), reviews section, FAQ, footer.

Clicking a property type card sets `propertyType` and calls `beginFunnel()`, which hides the landing and activates the funnel.

### Funnel (`id="funnel"`)
Hidden until a property type card is clicked. Contains progress bar + step cards. **No refi flow** — purchase only.

## Funnel Step System

All step cards are `<div class="step-card hidden" id="sN">`. Only the active card has `hidden` removed.

### Navigation

```javascript
go(step)                  // step = number (→ 's'+step) or string ('s12b')
pick(el, stepId, next)    // auto-advances after 280ms
selectPropertyType(el, type)  // stores propertyType, calls beginFunnel()
```

`curStep` is either a number or a string (for named steps like `'s12b'`, `'sload'`).

### Flow (purchase only)

Landing → s1 → s3 → rs4 → rs5 → rs6 → rs7 → rs8 → s12b → s8 → s9 → s10 → s11 → s12 → sload → s13 → s14 → sotp → s15

> Notes:
> - s2 removed — property type captured on the landing page.
> - Steps after s1 mirror the V1 VA refinance flow (rs4–rs8, then s12b for late payments, then credit onward).
> - s4, s5, s6, s7 (first-time buyer, homebuying stage, purchase price, down payment) removed.

| ID | Question |
|----|----------|
| **landing** | Property type (4 cards: Single Family, Townhome, Condo, Multi-Family) |
| s1 | Where is the property? (ZIP code) |
| s3 | Property use (Primary / Second Home / Investment) |
| rs4 | Estimated property value slider ($80K–$2M) |
| rs5 | Remaining mortgage balance slider (0–100%, shows $ via `syncRemainingBalance`) |
| rs6 | Current interest rate slider (0–12%, via `syncInterestRate`) |
| rs7 | Second mortgage (Yes/No) |
| rs8 | Additional cash slider ($0–$100K) |
| s12b | Any 30 day late payments in last 12 months? (No / 1 time / 2+) |
| s8 | Credit score slider (560–850, shows rating via `syncCreditScore`) |
| s9 | Employment status (opt-grid) |
| s10 | Branch of service (opt-icon 3-col grid) |
| s11 | Bankruptcy/foreclosure (Yes/No) |
| s12 | Annual income slider ($0–$500K) → Continue goes to sload |
| sload | Animated lender-matching loader |
| s13 | Lender matches (select up to 2) |
| s14 | Contact form (name, email, phone) |
| sotp | OTP phone verification |
| s15 | Thank you |

### Progress bar

```javascript
const progress = { 1:4, 3:11, 'rs4':18, 'rs5':25, 'rs6':32, 'rs7':38, 'rs8':44, 's12b':51, 8:58, 9:64, 10:69, 11:74, 12:80, 'sload':85, 13:89, 14:93, 'sotp':97, 15:100 };
```

## Key Element IDs

| ID | Purpose |
|----|---------|
| `prop-val` / `prop-range` | Property value input/slider (rs4) — also read by `syncRemainingBalance` |
| `rb-range` / `rb-pct-label` / `rb-dollar-val` | Remaining balance slider/display (rs5) |
| `ir-range` / `ir-val` | Interest rate slider/input (rs6) |
| `cash-range` / `cash-val` | Additional cash slider/input (rs8) |
| `cs-range` / `cs-display` / `cs-rating` | Credit score (s8) |
| `income-range` / `income-val` | Annual income (s12) |
| `pbar` / `ppct` | Progress bar fill and percent label |

## CSS Patterns

- **Property type cards (landing)**: `.prop-type-grid` (4-col → 2-col on ≤860px) > `.prop-type-card` > `.ptc-icon` / `.ptc-name` / `.ptc-desc` / `.ptc-cta` (absolute bottom bar). Hover: lifts, purple border, CTA bar fills purple.
- **Dollar input**: `.inp-dollar` wrapper with `.inp-dollar-sym` (`$`) on the left
- **Percentage input**: `.inp-pct` wrapper with `.inp-pct-sym` (`%`) on the right, `.inp` gets `padding-right:34px`
- **Slider labels**: `.slider-wrap` > `input[type=range]` + `.slider-labels`
- **Option styles**: `.opts-yn` / `.opt-yn` (Yes/No), `.opts-list` / `.opt-list` (text list), `.opts-grid` / `.opt-grid` (grid), `.opts-icon` / `.opt-icon-card` (icon cards)
- **Color tokens**: `--primary` (#5349DB), `--ink`, `--slate`, `--green`, `--purple`, `--lborder`
- **Breakpoints**: ≤860px (tablet), ≤640px (mobile), ≤390px (small phones)

## Assets

| File | Usage |
|------|-------|
| `VArateslogo.png` | Header logo |
| `ca.png` | Body background texture (repeating) |
| `veterans-united-logo.png` | Lender card (s13) |
| `better-mortgage-logo.svg` | Lender card (s13) |
| `rocket-logo.png` | Lender card (s13) |
