---
name: daily-topline-forecasting
description: Forecast daily topline for grocery business across international markets by disaggregating monthly targets using historical patterns. Accounts for paydays, weekends, holidays, seasonality, and grocery-specific buying behaviors.
---

# Daily Topline Forecasting for Grocery Business

Disaggregate monthly topline targets into daily forecasts across international markets, accounting for grocery-specific patterns, calendar effects, and country-specific behaviors.

## Input Files

**1. Monthly Targets (FY2026)**
- File: `monthly_targets_26.csv`
- Columns:
  - `month` - Month reference (e.g., "2026-01", "Jan 2026")
  - `country` - Country code or name
  - `monthly_topline` - Planned monthly topline value

**2. Historical Actuals (2024-2025)**
- File: `actuals_24_25.csv`
- Columns:
  - `date` - Individual dates (YYYY-MM-DD format)
  - `country` - Country code or name
  - `daily_topline` - Actual daily topline value

## Workflow

**Step 1: Learn Historical Patterns**

```bash
python scripts/learn_patterns.py \
  --actuals actuals_24_25.csv \
  --output-dir results/
```

Analyzes 2024-2025 actuals to identify:
- Day-of-week effects (weekend grocery shopping spikes)
- Payday patterns (beginning/middle/end of month)
- Holiday effects by country
- Seasonal patterns (summer holidays, school vacations)
- Outlier detection and handling
- Country-specific buying behaviors

Outputs: `results/learned_patterns.json` with all identified patterns.

**Step 2: Generate Daily Forecasts**

```bash
python scripts/forecast_daily.py \
  --targets monthly_targets_26.csv \
  --patterns results/learned_patterns.json \
  --output-dir results/
```

Disaggregates monthly targets into daily forecasts using learned patterns.

Outputs:
- `results/daily_forecast_2026.csv` - Daily forecasts by country
- `results/forecast_summary.json` - Summary statistics and validation metrics
- `results/forecast_report.txt` - Human-readable summary

**Step 3: Visualize & Validate**

```bash
python scripts/visualize_forecast.py \
  --forecast results/daily_forecast_2026.csv \
  --actuals actuals_24_25.csv \
  --output-dir results/
```

Creates validation plots comparing patterns:
- Daily patterns vs historical
- Month-over-month consistency
- Country-specific distributions
- Holiday effect validation

Outputs plots in `results/plots/`

**Step 4: Export for Business Use**

```bash
python scripts/export_forecast.py \
  --forecast results/daily_forecast_2026.csv \
  --format excel  # or 'csv', 'powerbi'
  --output-dir results/exports/
```

Exports in business-ready formats with pivot tables and charts.

## Grocery-Specific Features

The model automatically accounts for:

**Calendar Effects:**
- **Weekends**: Higher grocery shopping (Friday-Saturday peaks)
- **Paydays**: Spikes at beginning and mid-month
- **Month-end**: Stock-up behavior before month rollover
- **Holidays**: Country-specific holiday shopping patterns
- **School calendars**: Back-to-school, vacation periods

**Seasonal Patterns:**
- **Summer holidays**: Reduced routine shopping, increased BBQ/outdoor
- **Winter holidays**: Holiday meal shopping spikes
- **Regional seasons**: Country-specific seasonal produce patterns

**Business Cycles:**
- **Promotional calendars**: Common retail promotion periods
- **Economic cycles**: Adapt to historical economic patterns
- **Supply chain**: Account for historical supply disruptions

**Outlier Handling:**
- Automatic detection of anomalies in historical data
- Smoothing of one-off events
- Preservation of recurring special events

## Country-Specific Modeling

Each country gets custom treatment for:
- National holidays and observances
- Cultural shopping patterns (e.g., weekly market days)
- Regional seasonal variations
- Currency and economic factors
- Local payday conventions

## Script Options

### learn_patterns.py
```bash
--actuals PATH              # Historical actuals CSV (required)
--output-dir PATH           # Output directory (default: results/)
--min-observations INT      # Min days per country (default: 365)
--outlier-threshold FLOAT   # Std devs for outlier detection (default: 3.0)
--holiday-window INT        # Days before/after holiday to consider (default: 3)
```

### forecast_daily.py
```bash
--targets PATH              # Monthly targets CSV (required)
--patterns PATH             # Learned patterns JSON (required)
--output-dir PATH           # Output directory (default: results/)
--smooth-weekends           # Apply weekend smoothing
--adjust-holidays           # Adjust for known holidays in 2026
--constrain-monthly         # Force daily sum = monthly target exactly
```

### visualize_forecast.py
```bash
--forecast PATH             # Daily forecast CSV (required)
--actuals PATH              # Historical actuals for comparison (required)
--output-dir PATH           # Output directory (default: results/)
--countries LIST            # Specific countries to plot (comma-separated)
```

### export_forecast.py
```bash
--forecast PATH             # Daily forecast CSV (required)
--format {excel,csv,powerbi} # Export format (default: excel)
--output-dir PATH           # Output directory (default: results/exports/)
--include-confidence        # Add confidence intervals
```

## Output Files

```
results/
├── learned_patterns.json          # Learned patterns from historical data
├── daily_forecast_2026.csv        # Daily forecasts by country
├── forecast_summary.json          # Summary statistics and validation
├── forecast_report.txt            # Human-readable findings
├── plots/
│   ├── country_COUNTRY_overview.png
│   ├── day_of_week_patterns.png
│   ├── monthly_distribution.png
│   ├── seasonal_decomposition_COUNTRY.png
│   ├── holiday_effects.png
│   └── forecast_validation.png
└── exports/
    ├── daily_forecast_2026.xlsx   # Excel with pivot tables
    ├── daily_forecast_2026.csv    # Raw CSV
    └── powerbi_ready.csv          # Power BI optimized format
```

## Validation Metrics

The forecast includes:
- **MAPE**: Mean Absolute Percentage Error vs historical patterns
- **Coverage**: Percentage of monthly target allocated
- **Consistency**: Day-of-week pattern alignment with historical
- **Holiday alignment**: Proper holiday effect application
- **Outlier flags**: Days with unusual patterns

## Dependencies

```
pandas>=2.0.0
numpy>=1.24.0
matplotlib>=3.7.0
seaborn>=0.12.0
statsmodels>=0.14.0
scipy>=1.10.0
holidays>=0.35  # For country-specific holidays
openpyxl>=3.1.0  # For Excel export
```

## Installation

```bash
pip install pandas numpy matplotlib seaborn statsmodels scipy holidays openpyxl
```

## Quick Start Example

```bash
# 1. Learn patterns from historical data
python scripts/learn_patterns.py \
  --actuals data/actuals_24_25.csv \
  --output-dir results/

# 2. Generate 2026 daily forecasts
python scripts/forecast_daily.py \
  --targets data/monthly_targets_26.csv \
  --patterns results/learned_patterns.json \
  --constrain-monthly \
  --output-dir results/

# 3. Validate and visualize
python scripts/visualize_forecast.py \
  --forecast results/daily_forecast_2026.csv \
  --actuals data/actuals_24_25.csv \
  --output-dir results/

# 4. Export for business use
python scripts/export_forecast.py \
  --forecast results/daily_forecast_2026.csv \
  --format excel \
  --include-confidence \
  --output-dir results/exports/
```

## References

See `references/` for:
- `methodology.md` - Detailed algorithm explanation
- `grocery_patterns.md` - Grocery-specific behavior patterns
- `country_configs.md` - Country-specific settings and holidays
- `validation_guide.md` - How to validate and adjust forecasts
