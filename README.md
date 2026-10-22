# كيمت من جديد · Kemet Min Gedid

**Weather-to-action planning workflow with a decision audit trail — for Egyptian smallholders.**

SolveThe17 Hackathon (IEEE EUI Student Branch) · Track 1 · SDG 13 — *“Keeping Egypt’s Farms Alive”*

| | |
|---|---|
| **Live demo** | https://walnut-diner-m3as.here.now/ |
| **Prototype** | `app/` — React 18 + Vite + Tailwind single-page application |
| **Required documentation** | `docs/ARCHITECTURE.md`, `docs/BOM-EGP.md`, `docs/FIELD-DEPLOYMENT-BRIEF.md` |
| **Rule specification** | `docs/RULE-REGISTER.md` |

---

## What the app does

1. The user picks an Egyptian governorate / agricultural centre (over 20 built-in presets, plus free-text location, plus GPS) and the crop stage.
2. The app retrieves a **real 7-day forecast from Open-Meteo** — daily max/min temperature, precipitation sum and probability, maximum wind, and FAO-56 reference evapotranspiration (ETo), plus hourly detail.
3. A **deterministic rule** (`KEMET-72H`, see `docs/RULE-REGISTER.md`) reduces those numbers to one of three planning states over a **72-hour window**:
   - **Check A — Rain hold:** cumulative rain ≥ 5 mm **or** max rain probability ≥ 60 % → hold and verify soil moisture / field drainage before irrigating.
   - **Check B — Heat, evaporative demand, wind:** otherwise, max temperature ≥ 35 °C **or** peak ETo ≥ 6.5 mm/day **or** wind ≥ 30 km/h → verify root-zone moisture and avoid foliar spraying in high wind.
   - **Check C — Routine window:** none of the above → standard moisture inspection.
4. An **evidence card** shows the exact 72-hour metrics, which thresholds fired, the data provenance (`LIVE_OPEN_METEO_API` vs `DEMO_SIMULATION_FIXTURE`), the crop coefficient used, and a bilingual FAO-56 limitation note.
5. An optional **explanation layer** turns the evidence card into plain Egyptian Arabic. With no key it uses a constrained template explainer; with a user-supplied Gemini/OpenAI key it calls the model under a prompt that forbids inventing numbers, overriding the rule, or prescribing water volumes.
6. Every decision is written to a **local audit log** (`localStorage`), which is the point of the product: a prompt without a record of what the farmer did is just a notification.

### What the app deliberately does not do

It does not prescribe irrigation volumes (m³/feddan), predict crop damage, or replace a field inspection. Every card carries that limitation in Arabic and English, in line with FAO-56 (ETc = Kc × ETo is an *indicative* reference value, not a field measurement).

---

## Run it

```bash
cd app
npm install          # Node 20+; on npm 12 you may need: npm install-scripts approve esbuild@0.21.5 && npm rebuild esbuild
npm run dev          # http://localhost:5173
npm run build        # production bundle in app/dist
```

The build output is fully static (HTML + JS + CSS) — deploy by copying `app/dist` to any HTTPS static host. No server, no database, no account.

### Measured build (23 Oct 2026)

| Artifact | Size |
|---|---|
| `dist/assets/index-*.js` | 671.72 kB — **189.45 kB gzip** |
| `dist/assets/index-*.css` | 24.83 kB — 5.07 kB gzip |
| Modules transformed | 2 746 |
| Forecast API response (daily + hourly, 7 days) | **7.6 kB** (daily-only: 0.85 kB) |

---

## Repository layout

```
app/          the prototype (React + Vite + Tailwind source, no build output committed)
docs/         the three required submission documents + the rule register
README.md     this file
LICENSE       MIT
```

## Known gaps (stated up front, not hidden)

1. **The rule thresholds are indicative and unsourced.** 5 mm / 60 % / 35 °C / 6.5 mm-day⁻¹ / 30 km/h are engineering choices, not published agronomic thresholds. They need a review and sign-off by an agricultural specialist before any claim of agronomic validity. See `docs/RULE-REGISTER.md` §6.
2. **The optional AI key is entered in the browser** and the request goes from the browser to the provider. That is acceptable for a demo (the user pays with their own key, no project secret is shipped) but it is not production-grade: a real pilot needs a server-side proxy.
3. **Mobile overflow at 360 px:** measured `scrollWidth 382` vs `clientWidth 360` — 22 px of horizontal overflow on narrow phones.
4. **No external test runner.** The 8 verification tests run inside the app on mount (they assert the rule branches, the LIVE/DEMO isolation and the unit-economics constants). There is no CI and no headless test command yet.
5. **No field validation.** No farmer has used this build; no comprehension test, no impact measurement.

## Data sources and licences

| Source | Use | Licence / policy |
|---|---|---|
| [Open-Meteo](https://open-meteo.com/) forecast API | the only weather input | CC BY 4.0; **free tier is non-commercial / evaluation only** — a commercial licence is required before charging farmers, price not published (quote pending) |
| [Google Gemini API](https://ai.google.dev/) / [OpenAI API](https://platform.openai.com/) | optional explanation layer, only with a user-supplied key | provider terms, billed to the key owner |
| Google Fonts | typography | SIL OFL; system-font fallback if blocked |

## Licence

MIT — see [`LICENSE`](LICENSE). Weather data remains under the providers’ licences above.
