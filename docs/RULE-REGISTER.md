# Rule Register — `KEMET-72H`

Every prompt the farmer sees must be traceable to an entry in this file. If a rule is not written here, it is not shipped.

**Status of this register: thresholds are INDICATIVE and NOT yet signed off by an agricultural specialist.** That is stated here, in the UI on every card, and in the README. Treat the numbers below as engineering defaults pending agronomic review — not as published agronomic thresholds.

---

## 1. Identity

| Field | Value |
|---|---|
| Rule ID | `KEMET-72H` |
| Implemented in | `app/src/App.tsx` → `evaluateDeterministicPlanningRule()` |
| Window | first 3 days (72 hours) of the 7-day Open-Meteo forecast |
| Unit | mm, mm/day, °C, km/h, % |
| Inputs | `temperature_2m_max[]`, `temperature_2m_min[]`, `precipitation_sum[]`, `precipitation_probability_max[]`, `wind_speed_10m_max[]`, `et0_fao_evapotranspiration[]` + a crop stage (Kc) |
| Minimum data | at least 3 daily values per array; otherwise the function returns `null` and no card is produced |
| Output | one of three planning states (A / B / C) + the exact metrics that produced it |

---

## 2. Evaluation procedure (exact)

1. Slice days 0–2 of every daily array.
2. Compute: `tMax72` (max), `tMin72` (min), `rainSum72` (sum, 2 dp), `rainProbMax72` (max), `windMax72` (max, 1 dp), `et0Max72` (max), `et0Mean72` (mean).
3. Compute indicative crop demand: `ETc = Kc × ETo` (peak and mean). **Indicative only** — see §6.
4. **Check A — Rain hold** if `rainSum72 ≥ 5.0 mm` **OR** `rainProbMax72 ≥ 60 %`.
   Output: *HOLD / verify soil moisture and field drainage before irrigating* + the numbers that triggered it.
5. Else **Check B — Heat, evaporative demand, wind** if `tMax72 ≥ 35.0 °C` **OR** `et0Max72 ≥ 6.5 mm/day` **OR** `windMax72 ≥ 30 km/h`.
   Output: verify root-zone moisture; avoid foliar spraying in high wind.
6. Else **Check C — Routine 72 h window**: standard moisture inspection, flexible field operations.
7. Attach provenance (`LIVE_OPEN_METEO_API` / `DEMO_SIMULATION_FIXTURE`), the bilingual FAO-56 limitation, and the action steps to the evidence card.

A is evaluated before B: rain and drainage dominate, because irrigating into a forecast rain event is the most expensive mistake in the window.

---

## 3. Crop coefficients (indicative, FAO-56 mid-season shape)

| Crop stage | Kc | Note shown to the user |
|---|---|---|
| Wheat — mid-season canopy | 1.15 | mid-season Kc reference; actual field water need requires local soil-moisture and salinity check |
| Maize — mid-season tasseling | 1.20 | high sensitivity to heat and ETo spikes |
| Tomato / vegetables — mid-season fruiting | 1.10 | avoid foliar spraying during high wind (> 30 km/h) |
| Berseem (Egyptian clover) — active cut cycle | 1.05 | indicative Kc after canopy recovery |

These are **mid-season reference values in the FAO-56 family**, not a validated local calibration. A pilot must replace them with locally validated values per governorate and season.

---

## 4. Explicit non-uses

`KEMET-72H` must **not** be used to:

- prescribe an irrigation volume in m³/feddan, or any water quantity;
- predict crop damage, yield loss or income;
- replace a physical inspection of root-zone moisture, salinity or drainage;
- ignore the local canal rotation (مناوبة الترعة);
- rank locations or crops by “risk”;
- certify food safety or pesticide compliance.

Every evidence card prints the first four of these in Arabic and English.

---

## 5. Verification (executed)

The prototype runs 8 assertions on mount (visible in the *Live Verification Suite* tab). Those relevant to this rule:

| Test | Asserts |
|---|---|
| TEST-04 / TEST-05 | A synthetic hot/dry 72 h window yields Check B; a mild window yields Check C — i.e. the branch selection matches the thresholds above |
| TEST-06 | All DEMO fixtures carry `DEMO_SIMULATION_FIXTURE` and cannot pollute live state |
| TEST-07 | Every card carries the bilingual FAO-56 limitation and makes zero m³/feddan claims |
| TEST-08 | The workflow executes with the AI layer disabled (the rule never depends on the model) |

Gap: these assertions live inside the app and run in the browser on mount. There is **no external test runner and no CI** yet; the plan is to extract the rule function into a plain module and add a headless `node --test` suite.

---

## 6. Open issues before this rule can be called agronomic

1. **Threshold provenance.** 5 mm / 60 % / 35 °C / 6.5 mm·day⁻¹ / 30 km/h are engineering defaults. No published source is cited in the app for any of them. Until a specialist signs off (with a citation per threshold), the UI wording must stay “planning check”, never “risk” or “warning”.
2. **Kc values** are mid-season *reference* values; local calibration per crop, governorate and season is required.
3. **Rain probability field.** Open-Meteo’s `precipitation_probability_max` is a model probability for the grid cell, not an on-farm measurement.
4. **The 72-hour window** was chosen for planning relevance; it is not derived from any crop-specific study.
5. **No field validation** has been done: no farmer has used this build.

## 7. Change control

Any change to a threshold, to the branch order, or to the farmer-facing copy requires: a new version identifier (`KEMET-72H v1.1`), an entry in this register with the reasoning and a citation, a corresponding assertion in the verification suite, and — for anything that carries agronomic meaning — documented review by an agricultural specialist. Adding an irrigation-volume rule without such a review is out of scope for this project.
