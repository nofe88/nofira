# NOFIRA — Small Signals Matter.

An astronaut health and operational performance monitoring prototype, built for the **NASA Space Apps Challenge 2026**
(challenge: *Create Health Monitoring Software for Astronauts on Space Missions*). Team Rolinf, Tabuk, Saudi Arabia.

NOFIRA looks for **accumulating small operational deviations** (repeated errors, slower tasks, longer corrections) and
places them alongside available health indicators. It is a **research and decision-support prototype, not a medical
diagnostic system**. Correlation is not causation.

![Dashboard](docs/dashboard.png)
![Explainable alert](docs/alert.png)

## Run it
Single static file, no build step and no server needed.
- Open `index.html` in a browser, or
- Publish with GitHub Pages: Settings → Pages → Branch `main` → `/ (root)`.

## Features
Overview, astronaut profiles, health and performance views, analysis engine (configurable thresholds),
explainable alerts, mission timeline, mission control and astronaut views, interactive simulation lab with three
scenarios and before/after comparison, NASA Data Explorer (CSV/TSV upload), data sources page, CSV export,
printable report.

## Data provenance
| Class | Meaning |
|---|---|
| REAL NASA DATA | Only data you import yourself from OSD-569 / OSD-575 |
| SYNTHETIC DEMO | Seeded simulated telemetry (sleep, heart rate, fatigue, stress, errors, task and correction time) |
| DERIVED ANALYSIS | Values computed by NOFIRA's engine |

**This repository contains no NASA measurements.** To explore them, download a data file from
[OSD-569](https://osdr.nasa.gov/bio/repo/data/studies/OSD-569) or
[OSD-575](https://osdr.nasa.gov/bio/repo/data/studies/OSD-575) (Inspiration4), export to CSV/TSV if needed, and upload it
on the *NASA Data* page. Values, units and sampling timepoints stay as in the original file; nothing is imputed.
These samples are sparse biological timepoints from a short-duration mission, not continuous monitoring, and do not
contain operational errors, task times or sleep records.

## Operational Deviation Index (0–100)
1. Baseline: mean and SD of mission days 1–14 per metric (SD floor 5% of the mean).
2. Current value: mean of the last 3 days.
3. `z = direction × (current − baseline mean) / SD` (direction −1 for sleep).
4. `points = weight × clamp(z / 3, 0, 1)`.
5. Index = sum of points. Default levels: Low ≥ 20, Moderate ≥ 40, High ≥ 65 (configurable).

Weights: errors 25, task time 20, correction time 15, sleep 15, heart rate 10, fatigue 10, stress 5.
No random scores; every value is derived from the displayed inputs.

## Limitations
Thresholds and weights are illustrative and not clinically validated. Simulated data only, except what you import.
Uses plain HTML/JavaScript and SVG (no React, FastAPI or database). Browser storage keeps imported files locally.
