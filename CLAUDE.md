# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VA mortgage rate comparison funnel for **varates.now / rates.now**. Single-page application — all HTML, CSS, and JavaScript lives in one file: `index.html`. No build system.

## Deployment

```bash
/tmp/vercel-local/node_modules/.bin/vercel --prod
```

Global `npm install -g vercel` is blocked by macOS permissions, so vercel is installed locally. If `/tmp/vercel-local` is missing, reinstall:

```bash
npm install --prefix /tmp/vercel-local vercel --cache /tmp/npm-cache
```

Live URL: **https://ratesnowfunnel.vercel.app**

## Architecture

`index.html` has three logical sections in order:
1. **`<style>`** — all CSS (~500 lines)
2. **`<body>`** — landing page + funnel HTML
3. **`<script>`** — all JS at the bottom (~120 lines)

### Landing page (`id="landing"`)
Shown first. Contains header (VArateslogo.png), hero headline, two CTA buttons (Buy a Home / Refinance), VA national average rate bar, proof strip (3 highlights), reviews section, FAQ, footer.

### Funnel (`id="funnel"`)
Hidden until a CTA button is clicked. Contains progress bar + step cards.

## Funnel Step System

All step cards are `<div class="step-card hidden" id="sN">` (buy flow) or `id="rsN"` (refi-specific). Only the active card has `hidden` removed.

### Navigation

```javascript
go(step)          // step = number (→ 's'+step) or string ('rs4')
pick(el, stepId, next)    // auto-advances after 280ms
pickPropertyUse(el)       // flow-aware: go('rs4') or go(4)
```

`curStep` is either a number (buy flow) or string (refi flow). The `go()` function normalizes both.

### Flow type

```javascript
let flowType = 'buy';   // or 'refi'
beginFunnel('buy')      // triggered by Buy a Home button
beginFunnel('refi')     // triggered by Refinance button
```

### Buy flow step order
s1 → s2 → s3 → s4 → s5 → s6 → s7 → s8 → s9 → s10 → s11 → s12 → s13 → s14 → s15

| ID | Question |
|----|----------|
| s1 | ZIP code |
| s2 | Property type |
| s3 | Property use (calls `pickPropertyUse`) |
| s4 | First-time homebuyer (Yes/No) |
| s5 | Homebuying stage (opt-list) |
| s6 | Purchase price slider ($100K–$5M) |
| s7 | Down payment slider (0–100%, shows $ via `syncDownPayment`) |
| s8 | Credit score slider (560–850, shows rating via `syncCreditScore`) |
| s9 | Employment status (opt-grid) |
| s10 | Branch of service (opt-icon 3-col grid) |
| s11 | Bankruptcy/foreclosure (Yes/No) |
| s12 | Annual income slider ($0–$500K) |
| s13 | Lender matches (select up to 3) |
| s14 | Contact form (name, email, phone) |
| s15 | Thank you |

### Refinance flow step order
s1 → s2 → s3 → **rs4 → rs5 → rs6 → rs7 → rs8 → rs9** → s8 → s9 → s10 → s11 → s12 → s13 → s14 → s15

Refi skips: s4 (first-time buyer), s5 (homebuying stage), s6 (purchase price), s7 (down payment).

| ID | Question |
|----|----------|
| rs4 | Estimated property value slider ($80K–$2M, id: `prop-val` / `prop-range`) |
| rs5 | Remaining mortgage balance slider (0–100%, dollar display via `syncRemainingBalance`) |
| rs6 | Current interest rate slider (0–12%, pct input via `syncInterestRate`) |
| rs7 | Second mortgage (Yes/No) |
| rs8 | Additional cash slider ($0–$100K, id: `cash-val` / `cash-range`) |
| rs9 | Late mortgage payments (No / 1 Late / 2+) |

The s8 back button is flow-aware: `go(flowType === 'refi' ? 'rs9' : 7)`.

### Progress bars

Two progress maps, selected by `flowType`:
```javascript
const progress = { 1:4, 2:15, ... 15:100 };          // buy
const refiProgress = { 's1':4, 's2':10, ... 's15':100 }; // refi (keyed by string ID)
```

## Key Element IDs

| ID | Purpose |
|----|---------|
| `prop-val` | Property value text input (rs4) — also read by `syncRemainingBalance` |
| `rb-range` / `rb-pct-label` / `rb-dollar-val` | Remaining balance slider/display (rs5) |
| `ir-range` / `ir-val` | Interest rate slider/input (rs6) |
| `cash-range` / `cash-val` | Additional cash slider/input (rs8) |
| `price-val` / `price-range` | Purchase price (s6) |
| `dp-range` / `dp-pct-label` / `dp-dollar-val` | Down payment (s7) |
| `cs-range` / `cs-display` / `cs-rating` | Credit score (s8) |
| `income-range` / `income-val` | Annual income (s12) |
| `pbar` / `ppct` | Progress bar fill and percent label |

## CSS Patterns

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
