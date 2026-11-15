# NOFIRA — Small Signals Matter.

**Live website:** https://nofe88.github.io/nofira/  ·  **Repository:** https://github.com/nofe88/nofira

NOFIRA is an astronaut health and operational performance monitoring prototype built for the **NASA Space Apps Challenge 2026**
(challenge: *Create Health Monitoring Software for Astronauts on Space Missions*). Team Rolinf, Tabuk, Saudi Arabia.

> NOFIRA does not simply count mistakes. It identifies how small operational deviations accumulate over time.

**NOFIRA is an experimental decision-support research prototype. It is not a medical diagnostic system and does not replace flight surgeons or mission-control judgment.**

![Demo](docs/demo.png)

## The problem and the proposed solution
On demanding missions, small operational deviations (a repeated step, a slower task, an extra correction) look harmless one by one.
Seen together over time they can form a pattern worth a human look. NOFIRA compares each astronaut with their **own baseline**, detects accumulating
deviations, and produces **explainable alerts** for human review. It never diagnoses.

## Try it in one minute
1. Open the live site and press **TRY NOFIRA DEMO**.
2. Step through the four stages (or press Autoplay): normal operations → early deviations → accumulating deviations → explainable alert.
3. Open **How was this score calculated?** to inspect every weight, z-score and contribution.
4. Go to **Simulation Lab**, change sleep, errors, task or correction time and watch the index, charts, timeline and alert update.

The demo is deterministic and labeled **SIMULATED DEMONSTRATION DATA — NOT REAL ASTRONAUT MEASUREMENTS.**

## System architecture
Single static page (`index.html`): plain HTML, CSS and JavaScript with SVG charts. No framework, no server, no paid API. Works on GitHub Pages.

```
Operational data → Personal baseline → Deviation detection → Accumulation patterns → Explainable alert → Human review
 (simulated or      (days 1–14,        (z-scores, weights,    (repeated errors,       (index, reasons,       (flight surgeon /
  user-imported)     configurable)       Index 0–100)           rising times, recovery)  contributions)         mission control)
```
Settings and imported files are kept in the browser's local storage only; nothing is sent anywhere.

![How it works](docs/how-it-works.png)

## Data classification
| Class | Meaning |
|---|---|
| **REAL NASA DATA** | Only files you import yourself from OSD-569 / OSD-575 |
| **SIMULATED DATA** | Seeded or scripted demonstration telemetry (sleep, heart rate, errors, task and correction time, procedural deviations) |
| **DERIVED ANALYSIS** | Values computed by the NOFIRA engine (baselines, z-scores, index, patterns, alerts) |
| **USER-IMPORTED** | Operational CSV/TSV you load on the Engine page |

Every chart, card and export is labeled. The categories are never mixed silently.

## NASA datasets
| Accession | Study | Link |
|---|---|---|
| OSD-569 | SpaceX Inspiration4 Whole Blood Measurements | https://osdr.nasa.gov/bio/repo/data/studies/OSD-569 |
| OSD-575 | SpaceX Inspiration4 Blood Serum Metabolic Panel and Immune/Cardiac Cytokine Arrays | https://osdr.nasa.gov/bio/repo/data/studies/OSD-575 |

**This repository contains no NASA measurements.**

### How to import NASA data
1. Open the study page above and download a data file.
2. If it is Excel, export it to CSV or TSV.
3. Open **NASA Data**, pick OSD-569 or OSD-575, and upload the file.
4. NOFIRA reads the columns, detects numeric variables, units written in headers (e.g. `(g/dL)`), missing values, invalid numeric entries and time-like columns. Nothing is assumed in advance.
5. Choose a variable, a timepoint/group column and a filter value; chart the original values; export filtered CSV.

Invalid files (empty, header only, unreadable) show a clear error. Sparse biological samples are never interpolated or presented as continuous monitoring.

![NASA Data Explorer](docs/nasa-explorer.png)

## Operational Deviation Index (0–100)
1. **Baseline:** mean and SD of a selectable period (default mission days 1–14) per indicator. Fewer than 5 observations → indicator excluded.
2. **Current value:** mean of the last 3 days with data.
3. **z = direction × (current − baseline mean) ÷ SD** (direction −1 for sleep). SD has a floor (5% of the mean and a fixed minimum per indicator), so zero baselines never divide by zero.
4. **Normalization:** `n = clamp(z ÷ 3, 0, 1)`.
5. **Index = 100 × Σ(wᵢ × nᵢ) ÷ Σwᵢ**, summed over indicators that are available. Missing values are excluded, never treated as zero.

| Component | Default weight |
|---|---|
| Operational errors per day | 20 |
| Task completion time | 20 |
| Error correction time | 15 |
| Procedural deviations | 15 |
| Sleep duration (optional) | 10 |
| Fatigue indicator (optional) | 10 |
| Heart rate (optional) | 5 |
| Stress indicator (optional) | 5 |

Categories (configurable): **Normal** < 20 · **Observation** ≥ 20 · **Moderate deviation** ≥ 40 · **High deviation** ≥ 65.
Weights, thresholds and the baseline period are editable on the Engine page. The index is an experimental operational indicator,
not a medical risk score or probability of injury, and has not been clinically validated.

![Dashboard with score breakdown](docs/dashboard.png)

**Patterns detected:** isolated error · repeated minor errors (≥ 2 of 3 days) · gradually increasing task time · increasing correction time · repeated procedural deviations · accumulation across ≥ 3 indicators · recovery toward baseline.

## Import your own operational data
Engine page → *Import operational event data*. CSV/TSV columns: `astronaut_id, mission_day (1–30), task_time_min, errors, correction_min, procedural_deviations`, optional `sleep_h`. Several tasks per day are aggregated (time = mean, counts = sum). Missing values stay missing.

## Tests (executed)
12 engine tests run in the browser (Engine page → *Run engine tests*) and in Node. Results of the last run: **12 passed, 0 failed.**

| # | Test | Result |
|---|---|---|
| 1 | Stable operational baseline | PASS |
| 2 | One isolated error | PASS |
| 3 | Repeated minor errors | PASS |
| 4 | Gradually increasing task duration | PASS |
| 5 | Increasing correction duration | PASS |
| 6 | Recovery toward baseline | PASS |
| 7 | Missing observations | PASS |
| 8 | Insufficient baseline data | PASS |
| 9 | Zero baseline values | PASS |
| 10 | Invalid imported data | PASS |
| 11 | Determinism (identical inputs → identical results) | PASS |
| 12 | Changing input changes the index | PASS |

Also checked in a headless browser: 18 pages open without script errors; CSV and TSV import, empty and header-only file errors, filtered export, operational import (with skipped-row report), persistence and reset; no horizontal overflow at 390 px width.

![Engine tests](docs/engine-tests.png)

## Scientific methodology & limitations
1. The selected NASA datasets come from the Inspiration4 mission, a short-duration mission.
2. Their biological measurements exist at specific sampling timepoints.
3. They are not continuous real-time sensor streams.
4. Operational records used in the demonstration are simulated.
5. Biological and operational datasets are not assumed to share timepoints.
6. Correlation does not establish causation.
7. The deviation index is an experimental operational indicator, not a clinically validated medical score.
8. NOFIRA is a decision-support research prototype, not a medical diagnostic system.

Other limits: weights and thresholds are illustrative; imported data stays in the browser only; the 1–30 mission-day window is fixed.

## Run locally
Open `index.html` in a browser. To publish: Settings → Pages → Branch `main` → `/ (root)`.
