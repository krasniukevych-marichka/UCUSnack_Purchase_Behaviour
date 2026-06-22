# UCUSnack Purchase Behavior Analysis 📊

An econometric analysis of real sales data from UCUSnack — a student snack business operating across 15 dormitory locations at Ukrainian Catholic University (UCU).

## Overview

UCUSnack sells snacks through unmanned QR-code-based points of sale across two dormitory buildings (K1 and K2) and a student space (Студпростір). The digital payment system generates transactional data that enables data-driven decisions about restocking, product placement, and demand forecasting.

This project models the purchase frequency of chocolate bars (the most popular product category) using OLS regression, and tests whether location, day of the week, and weather conditions are significant drivers of demand.

## Research Questions

- Does **location** significantly predict purchase frequency, and which stands show systematically higher or lower demand?
- Are there **day-of-week** effects on chocolate bar purchases?
- Do **weather conditions** (precipitation, cloud cover) measurably affect demand?
- Does **customer gender** composition influence purchase frequency?

## Data

### Primary Dataset
Transactional data collected from the UCUSnack WooCommerce website covering **January 25 – April 4, 2026** (~1,024 observations after aggregation). Each row in the raw data represents a single item within an order and includes:

| Variable | Description |
|---|---|
| Order Number | Unique order identifier |
| Order Date | Timestamp (YYYY-MM-DD HH:MM) |
| Store Location | One of 15 location identifiers (e.g. `k1kvasiukiv`, `k2goldak`) |
| Item Name | Product purchased |
| Quantity | Units purchased |
| Item Cost | Price in UAH |

### Weather Data
Daily meteorological data for Lviv retrieved from the [Open-Meteo historical archive API](https://archive-api.open-meteo.com/v1/archive):
- `precipitation_sum` — total daily precipitation (mm)
- `cloudcover_mean` — mean daily cloud cover (%)

A binary variable `weather` was constructed: **bad** if precipitation > 0 mm or cloud cover > 70%, **good** otherwise.

### Gender Data
Customer gender was inferred from first names using a rule-based algorithm based on Ukrainian morphological rules (e.g. endings `-а`, `-я` → female; `-ій`, `-ло`, `-ан` → male), with manual review for unresolved cases. Approximately 3% of orders remain unclassified.

## Methodology

### Feature Engineering
- **purchase_count** — total units of chocolate bars sold per date × location
- **total_orders** — total unique orders at a location on a given day
- **female_count** — number of orders from female customers per date × location

### Model
OLS regression with **heteroskedasticity-robust standard errors (HC3)**:

```
purchase_count = β₀
  + Σ βᵢ · location_dummies     (14 locations, base: k1hnylytsi)
  + Σ γⱼ · day_of_week_dummies  (6 days, base: Saturday)
  + δ₁ · female_count
  + δ₂ · total_orders
  + δ₃ · good_weather
  + ε
```

### Diagnostics
- **Heteroskedasticity** — detected via Breusch-Pagan test → applied HC3 robust standard errors
- **Multicollinearity** — checked via VIF; all values within acceptable bounds
- **Autocorrelation** — Durbin-Watson statistic = 1.646, no evidence of serial autocorrelation

## Results

| Metric | Value |
|---|---|
| R² | 0.471 |
| Adjusted R² | 0.459 |

### Hypothesis Testing

| Hypothesis | Test | Result |
|---|---|---|
| H1: Day-of-week effect | Joint F-test (q=6) | F(6,1000) = 0.622, p = 0.713 — **not significant** |
| H2: Weather effect (good → lower demand) | One-sided t-test | p = 1.000 — **not rejected** (effect is opposite direction) |
| H3: Location effect | Joint F-test (q=14) | F(14,1000) = 7.371, p < 0.001 — **significant** ✓ |

### Key Findings
- **Location** is the strongest demand driver. Stands `k1pletenyh` (β = 0.765) and `k2goldak` (β = 0.750) significantly outperform the baseline; `k1gurby` (β = −0.428) underperforms.
- **Total orders** positively predict sweets demand (β = 0.240, p < 0.001) — roughly every 4 additional transactions yield one extra sweets purchase.
- **Day of week** has no significant effect — uniform restocking across weekdays is sufficient.
- **Good weather** is associated with *higher* (not lower) sweets demand, contrary to the initial hypothesis.
- **Female customer count** shows a weak positive association (p = 0.055), insufficient for gender-based inventory decisions at this stage.

## Business Implications

Based on the findings, higher sweets stock should be allocated to top-performing locations (`k1pletenyh`, `k1kvasiukiv`, `k2melnychenka`, `k2melnykiv`, `k2goldak`), while `k1gurby` may benefit from a reduced sweets assortment. Restocking frequency does not need to be differentiated by day of the week.
