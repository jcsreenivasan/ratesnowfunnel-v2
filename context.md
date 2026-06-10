# ratesnowfunnel-v2 — Project Context

## What this is
A single-file HTML/CSS/JS lead-generation funnel for the brand **VArates.now**, targeting U.S. military veterans seeking a **VA Cash-Out Refinance**. No build tools, no framework, no backend. Everything lives in `index.html`.

## Repository & Deployment
- **GitHub:** https://github.com/jcsreenivasan/ratesnowfunnel-v2
- **Active branch:** `experiment1-homepage` (CRO experiment variant)
- **Live Vercel URL:** https://ratesnowfunnel-v2-ratesnowfunnel-ca.vercel.app
- **Vercel project:** `ratesnowfunnel-v2-ratesnowfunnel-cashout-v2` (team: `jcsreenivasans-projects`)
- **Deploy command** (manual — Vercel does NOT auto-deploy from GitHub push):
  ```bash
  /tmp/vercel-local/node_modules/.bin/vercel --prod
  ```
  Run from: `/Users/sreenivasanjayachandrabalaji/Desktop/claudework/ratesnowfunnel-v4-cro/ratesnowfunnel-v2-ratesnowfunnel-cashout-v2`

  If `/tmp/vercel-local` is missing, reinstall:
  ```bash
  mkdir -p /tmp/gh_install && curl -sL https://github.com/cli/cli/releases/download/v2.92.0/gh_2.92.0_macOS_arm64.zip -o /tmp/gh_install/gh.zip && unzip -q -o /tmp/gh_install/gh.zip -d /tmp/gh_install/
  npm install --prefix /tmp/vercel-local vercel --cache /tmp/npm-cache
  ```

  The `gh` CLI (for git push auth) also lives in `/tmp` and may need reinstalling:
  ```bash
  mkdir -p /tmp/gh_install && curl -sL https://github.com/cli/cli/releases/download/v2.92.0/gh_2.92.0_macOS_arm64.zip -o /tmp/gh_install/gh.zip && unzip -q -o /tmp/gh_install/gh.zip -d /tmp/gh_install/
  ```

## File Structure
| File | Description |
|------|-------------|
| `index.html` | Entire app — all HTML, CSS, JS (~1300 lines) |
| `VArateslogo.png` | Header logo |
| `ca.png` | Camouflage background texture (body background) |
| `veterana.png` | Hero image (desktop) |
| `veterana2.png` | Hero image (desktop alternate) |
| `veterana-mobile.png` | Hero image (mobile, used via `<picture>`) |
| `american-flag.png` | Small flag icon in rate bar |
| `veterans-united-logo.png` | Lender logo (s13) |
| `better-mortgage-logo.svg` | Lender logo (s13) |
| `rocket-logo.png` | Lender logo (s13) |
| `strong-home-mortgage-logo.webp` | Lender logo (s13) |
| `shm-logo.png` | Lender logo |
| `lender3-logo.svg` | Lender logo |
| `farmers-bank-logo.svg` | Lender logo |
| `armed-forces-logo.svg` | Military branding |
| `.vercel/project.json` | Vercel project ID + org ID |
| `CLAUDE.md` | Claude Code guidance file |
| `context.md` | This file |

## `index.html` Architecture
Three logical sections in order:
1. `<style>` — all CSS (~700 lines)
2. `<body>` — SVG icon library, then landing page (`#landing`), then funnel (`#funnel`)
3. `<script>` — all JS at the bottom (~130 lines)

## Landing Page (`id="landing"`)
Shown on load. Contains:
1. **Fixed header** — VArates logo (centered desktop, left-aligned mobile), height 64px desktop / 56px mobile
2. **Hero section** (`.hero`) — full-bleed banner image with overlay, eyebrow badge, h1, subtext
3. **Property type selector** (`.prop-type-grid`) — 4 cards (Single Family, Townhome, Condo, Multi-Family); clicking any begins the funnel
4. **VA Rate Bar** — "Today's VA 30-Yr Fixed Rate: 5.70% vs national avg 6.39%"
5. **Q&A chat bubbles** — 3 cards with common VA Cash-Out questions
6. **FAQ accordion**
7. **Reviews section** — Google / Zillow / X ratings + 3 review cards
8. **Article section** — "VA IRRRL vs. Cash-Out Refinance" educational content
9. **Footer**

## Funnel (`id="funnel"`)
Hidden until a property type card is clicked (`beginFunnel()`). Steps:

| Step ID | Question |
|---------|----------|
| s1 | ZIP code |
| s3 | Property use (Primary / Second Home / Investment) |
| rs4 | Estimated property value (slider $80K–$2M) |
| rs5 | Remaining mortgage balance (slider, 0–100%) |
| rs6 | Current interest rate (slider 0–12%) |
| rs7 | Second mortgage? (Yes/No) |
| rs8 | Additional cash needed (slider $0–$100K) |
| s12b | Any 30-day late payments in last 12 months? |
| s8 | Credit score (slider 560–850) |
| s9 | Employment status (grid) |
| s10 | Branch of service (icon grid) |
| s11 | Bankruptcy/foreclosure? (Yes/No) |
| s12 | Annual income (slider $0–$500K) |
| sload | Animated lender-matching loader |
| s13 | Lender matches — "You have 8 mortgage offers that are ready for review!" |
| s14 | Contact form (name, phone — email removed) |
| sotp | OTP phone verification (UI-only, no real SMS) |
| s15 | Thank you page |

Flow: Landing → s1 → s3 → rs4 → rs5 → rs6 → rs7 → rs8 → s12b → s8 → s9 → s10 → s11 → s12 → sload → s13 → s14 → sotp → s15

## Key JS Functions
```js
go(step)                        // navigate to step (number → 's'+n, or string for named steps)
pick(el, stepId, next)          // select option, auto-advance after 280ms
selectPropertyType(el, type)    // store propertyType, call beginFunnel()
beginFunnel()                   // hide landing, activate funnel, go to s1
showLanding() / exitFunnel()    // return to landing page
syncSlider() / syncCreditScore() / syncRemainingBalance() / syncInterestRate() // slider display
cancelLoader()                  // cancel sload screen
continueWithOffers()            // advance from s13
buildTyLenders()                // build thank-you lender results
```

## CSS Design Tokens
```css
--primary:    #5349DB   (main purple)
--purple:     #5B5BD6
--purple-dark:#1E1B4B
--primary-lt: #ece9ff   (light purple tint)
--green:      #16a25b
--green-lt:   #d6f4e6
--gold:       #F59E0B
--ink:        #0f0e1a   (near-black text)
--slate:      #6b6b8a   (secondary text)
--radius:     20px
```

## Responsive Breakpoints
| Breakpoint | Applies to |
|------------|------------|
| `≤860px` | Tablet — prop-type-grid 2-col, reviews 2-col |
| `≤640px` | Mobile — full mobile layout (see below) |
| `≤390px` | Small phones — hero h1 26px, lender grid 1-col |

## Mobile-Specific Layout Notes (≤640px)
- **Header:** 56px tall, logo left-aligned
- **#landing:** `padding-top: 56px` (matches mobile header height)
- **Hero section:** `padding: 0 18px 28px` (no top padding — image sits flush below header)
- **Hero banner image:** Full-bleed edge-to-edge (`width: calc(100%+36px); margin: 0 -18px 24px`), no border-radius, height 163px, black overlay retained, h1 and subtext in white over the image
- **s13 heading:** Font-size overridden to 22px (`!important`) to fit "You have 8 mortgage offers / that are ready for review!" in 2 lines

## CRO Changes Made on `experiment1-homepage` (June 2026)
1. Mobile hero: full-bleed image (edge-to-edge, no border-radius, no shadow)
2. Mobile hero: text (eyebrow, h1, subtext) overlaid on image with black overlay; h1 + subtext in white
3. Mobile hero: removed gap between fixed header and image (padding-top set to match header height)
4. Mobile hero: image height reduced 35% (250px → 163px)
5. s13 heading: font-size reduced to 22px on mobile so the 2-line heading fits without overflow
