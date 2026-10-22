# Field Deployment Brief — كيمت من جديد (Kemet Min Gedid)

**SolveThe17 · Track 1 · SDG 13 “Keeping Egypt’s Farms Alive” · prototype `app/` (v3) · 23 Oct 2026**

Scope: the software-only prototype in this repository, exactly as it behaves today. *(measured)* marks a number produced by a real build or request on 23 Oct 2026. *(assumption)* marks a planning input, not a result.

---

## 1. Egyptian operating conditions and how the build responds

### 1.1 Temperature and sunlight — the app runs on the farmer’s phone, in the field

| Condition | Effect | What the build does about it |
|---|---|---|
| 35–45 °C ambient, phone in direct sun | screen dims, touch accuracy drops, battery and thermal throttling | The decision is reached in **three taps** (preset → crop stage → read card). There is no map render, no video, no polling loop: **one** forecast request per action, result drawn once. Thresholds are printed as text as well as drawn as reference lines, so a washed-out chart is never the only carrier of meaning. |
| Glare on glossy screens | faint charts disappear | dark UI with high-contrast text; every number that matters (rain sum, probability, tmax, ETo, wind) is also written in the action sentences, not only plotted |
| Long field sessions | heat from a heavy JS bundle | 189 kB gzip paid **once** on first load, then served from the browser cache; each check afterwards costs one 7.6 kB API response |

### 1.2 Dust and hardware

There is **no field hardware** in this build, so device survival is not the issue. The honest caveat is the opposite one: the forecast is a **model value for the grid cell**, not a measurement inside a dusty, hot field with its own microclimate. The card says so on every screen (FAO-56 limitation + “verify root-zone moisture by hand”). Heat and dust can change actual moisture; the app never claims to have measured it.

### 1.3 Bandwidth and data cost *(measured)*

| Metric | Value | How measured |
|---|---|---|
| First load | **index.html + 671.72 kB JS (189.45 kB gzip) + 24.83 kB CSS (5.07 kB gzip)** — 2 746 modules | production `vite build` |
| One forecast check | **7.6 kB** (7-day daily + hourly JSON) | live request to `api.open-meteo.com` |
| Requests per check | 1 weather call (+1 optional AI call, only if the user entered a key) | code path |
| Round trip on the demo machine | 1 343 ms, HTTP 200 | app’s own timing display, headless Chrome run |
| Subsequent checks | ≈ **8 kB** each (bundle is cached) | size maths |
| Data per month at 4 checks | ≈ **0.03 MB** | 4 × 7.6 kB |

Consequence: usable on a weak 3G connection and cheap on a prepaid data plan; the 189 kB first load is the one real bandwidth cost, and it is **7× heavier** than a hand-written static equivalent (27 kB gzip) — a deliberate trade for the richer interface, and a candidate for trimming if field users complain.

### 1.4 Rural literacy and the Arabic interface

- Arabic is the primary language, with an **AR / EN / AR+EN** toggle so an extension agent and a farmer can look at the same screen together.
- Location entry never requires spelling: **more than 20 governorate and agricultural-centre presets** (Kafr El-Sheikh, Desouk, Damanhur, Aswan…), plus GPS, plus free text.
- The verdict is a full Arabic sentence with the numbers inside it — not an icon, not a colour, not an abbreviation.
- The crop stage is chosen from four named options with Kc shown in words (“القمح — مرحلة النمو المتوسطة”).
- Technical identifiers (`KEMET-72H`, branch codes, `LIVE_OPEN_METEO_API`) are kept away from the primary reading path and appear only in the evidence/audit sections, where a judge or an auditor looks.
- **Not yet tested with any farmer.** Whether the wording is understood in the field is an open question, not an achievement.

---

## 2. Known defects and risks in this build (declared, not hidden)

| # | Item | Impact | Mitigation before a pilot |
|---|---|---|---|
| 1 | **22 px horizontal overflow at 360 px width** (`scrollWidth 382` vs `clientWidth 360`, measured) | small phones get a sideways scroll — the target user’s most common screen | CSS fix on the widest card/table; re-measure at 320/360/390 |
| 2 | **Optional AI key is entered in the browser** | a key in memory is readable by injected scripts/extensions; the evidence JSON leaves the device to a provider | move the call behind a one-endpoint proxy, or ship without the live-LLM path |
| 3 | **Rule thresholds are unsourced** (5 mm / 60 % / 35 °C / 6.5 / 30 km/h) | a judge or an agronomist can challenge them; a farmer could act on an unvalidated number | agronomist review + a citation per threshold (see `docs/RULE-REGISTER.md` §6) |
| 4 | No external test runner | regressions are caught only by the 8 in-app assertions | extract the rule to a module and add `node --test` in CI |
| 5 | No offline cache | a dead connection = no data (the app offers DEMO fixtures, clearly labelled) | add a stored last-valid forecast with its timestamp, shown as `CACHED` |
| 6 | Open-Meteo free tier is non-commercial | cannot legally monetise as-is | paid licence (quote pending) before any revenue |

---

## 3. Pilot plan (what is actually next, in order)

| Phase | Duration | Activity | Success signal |
|---|---|---|---|
| P0 — agronomic review | 1–2 weeks | an agronomist reviews the five thresholds, the Kc table and the Arabic copy; each accepted number gets a citation, the rest are removed | signed register, or a documented decision to drop a check |
| P1 — usability test | 1 week | 10–15 farmers + 2 extension agents, Arabic only, own phones, one Delta and one Upper-Egypt village | ≥ 80 % complete a check unaided; the verdict is explained back in the farmer’s own words |
| P2 — closed pilot | 4 weeks | one village, weekly use, extension-agent visits; audit log read on the device | decisions recorded; reported usefulness; **no impact claim** beyond what was observed |
| P3 — costed operations | ongoing | commercial weather licence, server-side proxy for the AI call, monitoring | published cost per farmer per season |

## 4. Support and failure modes in the field

| Failure | What the farmer sees | What the team does |
|---|---|---|
| No connection | no live card; DEMO fixtures available and clearly labelled `DEMO / SIMULATION` | add a cached-forecast state with its original timestamp (defect #5) |
| Weather service down or rate-limited | error state, no fabricated card | re-run the demo build check; a pilot needs monitoring |
| AI provider error / no key | template explainer, labelled as such | nothing — the decision path never depended on the model |
| Wrong location picked | the chosen place is printed on the card and in the audit log | preset list is the default path, GPS is secondary |

**Bottom line for judges:** the prototype already produces a real forecast, a deterministic three-branch planning state, an evidence card that shows why it fired, and a device-local audit trail. What it does **not** have is agronomic sign-off, a field-tested interface, or measured impact — and the brief above says so explicitly rather than implying otherwise.
