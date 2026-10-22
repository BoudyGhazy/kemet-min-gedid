# Bill of Materials (EGP) — Kemet Min Gedid · prototype `app/`

**Prepared:** 23 October 2026 · **currency:** Egyptian Pound (EGP) · **conversion used:** 1 USD = 55 EGP (rounded *budgeting* rate, not a live Central Bank quote — refresh before committing money)

Labels: **VERIFIED** = published price/limit read on the vendor page · **CALCULATED** = arithmetic from verified/assumed inputs · **PLANNING ASSUMPTION** = our own budget input · **QUOTE PENDING** = no published price exists and **must not be budgeted as 0**.

---

## 1. Hardware BOM

**None.** The prototype has no sensor, microcontroller, enclosure or field device. It is a static web application running in the farmer's existing browser.

| User equipment | In project cost? | Treatment |
|---|---|---|
| Smartphone with a modern browser | No | Bring-your-own-device **assumption** — to be validated with the target group; a phone-less or data-less farmer cannot be served without an SMS/USSD channel that does not exist yet |
| Mobile data | No | Existing package. Measured cost of one full check: **7.6 kB** forecast response + up to 189 kB of JS on the *first* load only (cached afterwards by the browser) |
| Team laptops | No | Existing equipment / in-kind |

No Cairo electronics-market sourcing, no import, no customs: there are no parts to buy.

---

## 2. Software and services BOM (the stack actually used)

| # | Item | Type | USD | EGP | Constraint |
|---|---|---|---|---|---|
| 1 | GitHub public repository (submission requirement) | VERIFIED | $0 | **0** | public repo, MIT licence |
| 2 | Static HTTPS hosting (any static host) | VERIFIED | $0 | **0** | the build is 3 files: `index.html`, one JS bundle, one CSS file |
| 3 | Open-Meteo Forecast API — free / open-access tier | VERIFIED | $0 | **0** | **evaluation / non-commercial only**; published limits 600 calls/min, 10 000/day, 300 000/month; no uptime guarantee; CC BY 4.0 attribution required (present in the app footer) |
| 4 | Open-Meteo **commercial** licence | **QUOTE PENDING** | not published | **QUOTE PENDING** | required before charging farmers. The pricing page (checked 22 Oct 2026) lists API Standard 1M calls/mo, Professional 5M/mo, Enterprise 50M+ — **no price is shown**; subscription is via Stripe checkout, volume quotes via `info@open-meteo.com`. Do not enter 0 in a real budget. |
| 5 | Google Fonts (Tajawal) | VERIFIED | $0 | **0** | only external asset; system-font fallback if blocked |
| 6 | Optional custom domain | PLANNING ASSUMPTION | ~$15/yr | **≈825 / yr** | allowance only; the demo runs on the free host subdomain |
| 7 | Optional LLM explanation — Gemini 2.5 Flash or OpenAI gpt-4o-mini | CALCULATED | — | **≈0.15 EGP per explanation** | **the user supplies their own key** — the project pays nothing, the user pays their provider. The figure is the app's own modelled estimate (~1 180 input / 285 output tokens); the per-token price must be re-checked on the provider's current price page before budgeting. With no key the template explainer costs 0. |
| 8 | SMS / USSD gateway (only if a feature-phone channel is added) | **QUOTE PENDING** | — | **QUOTE PENDING** | not part of this build; Egyptian aggregator pricing must be quoted |
| 9 | Developer time | PLANNING ASSUMPTION | — | **in-kind** | student team. The app itself models a professional-rebuild scenario at 16 000–126 000 EGP (16–180 h at 500–1 000 EGP/h) — a *model*, not a quote. |

---

## 3. Totals

| Scenario | Cash cost | Covers |
|---|---|---|
| Hackathon demo as built | **0 EGP** | free static host, free weather tier, no domain, no key |
| Demo with a branded domain | **825 EGP / yr** | domain allowance only |
| Pilot-ready | **QUOTE PENDING** + hosting | commercial weather licence; optionally a small proxy server for the AI call |
| Pilot with SMS channel | **QUOTE PENDING** | per-message gateway pricing |

## 4. Recurring / maintenance

| Item | Cost | Note |
|---|---|---|
| Static hosting | 0 EGP | free tier |
| Weather + geocoding | 0 EGP on free tier | evaluation-only; production needs the paid licence (quote pending) |
| Monitoring | 0 EGP in this build | manual: rebuild + load the page before each demo |
| Domain | 825 EGP/yr (assumption) | optional |

## 5. What is *not* priced here, on purpose

Agronomist review time, field pilot logistics, farmer incentives, and translation/voice-over for a rural-literacy variant. All four are real costs of the next stage and none of them has a quote — inventing a number would be worse than leaving the line open.
