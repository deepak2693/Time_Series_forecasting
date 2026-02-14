# Daily Topline Forecasting for Grocery Business

Transform monthly topline targets into daily forecasts across international markets using historical patterns. Specifically designed for grocery retail with built-in intelligence for weekends, paydays, holidays, and seasonality.

## What This Does

**Input:**
- Monthly topline targets for FY2026 (12 rows per country)
- 2 years of daily actuals (2024-2025)

**Output:**
- Daily topline forecasts for all of FY2026
- Pattern analysis reports
- Validation visualizations
- Business-ready Excel exports

**Key Features:**
- ✅ Grocery-specific patterns (weekend shopping, payday effects)
- ✅ Country-specific holidays and calendars
- ✅ Seasonal intelligence (summer lulls, holiday peaks)
- ✅ Outlier detection and handling
- ✅ Automatic pattern learning from history
- ✅ Monthly constraint (daily totals = monthly targets)

## Quick Start

### 1. Installation

```bash
# Install dependencies
pip install pandas numpy matplotlib seaborn statsmodels scipy holidays openpyxl

# Or use requirements file
pip install -r requirements.txt
```

### 2. Prepare Your Data

**File 1: actuals_24_25.csv**
```csv
date,country,daily_topline
2024-01-01,US,125000
2024-01-02,US,98000
2024-01-03,US,142000
...
```

**File 2: monthly_targets_26.csv**
```csv
month,country,monthly_topline
2026-01,US,3500000
2026-02,US,3200000
2026-03,US,3600000
...
```

### 3. Run the Workflow

```bash
# Step 1: Learn patterns from historical data
python scripts/learn_patterns.py \
  --actuals data/actuals_24_25.csv \
  --output-dir results/

# Step 2: Generate daily forecasts
python scripts/forecast_daily.py \
  --targets data/monthly_targets_26.csv \
  --patterns results/learned_patterns.json \
  --constrain-monthly \
  --output-dir results/

# Step 3: Visualize and validate
python scripts/visualize_forecast.py \
  --forecast results/daily_forecast_2026.csv \
  --actuals data/actuals_24_25.csv \
  --output-dir results/

# Step 4: Export for business use
python scripts/export_forecast.py \
  --forecast results/daily_forecast_2026.csv \
  --format excel \
  --output-dir results/exports/
```

### 4. Review Results

Check these files:
- `results/pattern_summary.txt` - What patterns were learned
- `results/forecast_report.txt` - Forecast summary
- `results/plots/` - Validation charts
- `results/exports/daily_forecast_2026.xlsx` - Business-ready export

## Understanding the Output

### Daily Forecast File

Each row represents one day:

```csv
date,country,daily_forecast,day_of_week,day_of_month,month,is_weekend
2026-01-01,US,112500.50,Thursday,1,January,False
2026-01-02,US,98234.25,Friday,2,January,False
2026-01-03,US,156789.75,Saturday,3,January,True
```

### Pattern File

Contains learned patterns:

```json
{
  "US": {
    "day_of_week": {
      "Monday": {"index": 0.98, "mean": 115000},
      "Saturday": {"index": 1.35, "mean": 158000},
      ...
    },
    "day_of_month": {
      "early": {"index": 1.08},
      "mid": {"index": 1.05},
      "late": {"index": 0.95}
    },
    ...
  }
}
```

## How It Works

1. **Pattern Learning**: Analyzes 2024-2025 actuals to identify:
   - Weekend shopping patterns (Friday-Saturday peaks)
   - Payday effects (beginning/mid-month spikes)
   - Holiday shopping behaviors
   - Seasonal variations

2. **Forecast Generation**: Applies learned patterns to distribute monthly targets:
   ```
   Daily forecast = Base value × Day-of-week × Day-of-month × Seasonality × Holidays
   ```

3. **Monthly Constraint**: Scales forecasts to ensure:
   ```
   Sum(daily forecasts for month) = Monthly target (exactly)
   ```

## Validation Checklist

Before using forecasts:

- [ ] Check monthly totals match targets
- [ ] Verify weekend lift is realistic (15-35%)
- [ ] Review day-of-week patterns vs. historical
- [ ] Inspect holiday effects
- [ ] Look for unrealistic spikes/valleys
- [ ] Compare seasonal patterns

See `references/validation_guide.md` for detailed validation steps.

## Grocery-Specific Intelligence

### Weekend Shopping
- Friday: 120-140% of daily average
- Saturday: Peak day (130-150%)
- Sunday: Variable by region (80-120%)

### Payday Patterns
- Early month (1-10): +5-15% lift
- Mid-month (11-20): +0-10% lift
- Late month (21-31): -5-10% dip

### Holidays
- Pre-holiday: +20-50% (shopping spike)
- Holiday day: -50-80% (stores closed/limited)
- Post-holiday: -10-20% (recovery)

### Seasonal Patterns
- November-December: Holiday peak (+15-35%)
- January: Post-holiday dip (-5-10%)
- Summer (July-Aug): Vacation impact (-5-15%)

## Advanced Usage

### Custom Outlier Detection

```bash
python scripts/learn_patterns.py \
  --actuals data/actuals_24_25.csv \
  --outlier-threshold 2.5 \
  --output-dir results/
```

### Disable Weekend Smoothing

```bash
python scripts/forecast_daily.py \
  --targets data/monthly_targets_26.csv \
  --patterns results/learned_patterns.json \
  --no-smooth-weekends \
  --output-dir results/
```

### Specific Countries Only

```bash
python scripts/visualize_forecast.py \
  --forecast results/daily_forecast_2026.csv \
  --actuals data/actuals_24_25.csv \
  --countries "US,UK,DE" \
  --output-dir results/
```

### Power BI Export

```bash
python scripts/export_forecast.py \
  --forecast results/daily_forecast_2026.csv \
  --format powerbi \
  --output-dir results/exports/
```

## File Structure

```
daily_topline_forecasting/
├── SKILL.md                          # Skill documentation
├── README.md                         # This file
├── requirements.txt                  # Python dependencies
├── scripts/
│   ├── learn_patterns.py            # Pattern learning
│   ├── forecast_daily.py            # Forecast generation
│   ├── visualize_forecast.py        # Visualization
│   └── export_forecast.py           # Business exports
├── references/
│   ├── methodology.md               # Algorithm details
│   ├── grocery_patterns.md          # Grocery behavior patterns
│   └── validation_guide.md          # Validation checklist
├── data/                            # Your input data (not included)
│   ├── actuals_24_25.csv
│   └── monthly_targets_26.csv
└── results/                         # Output directory
    ├── learned_patterns.json
    ├── daily_forecast_2026.csv
    ├── pattern_summary.txt
    ├── forecast_report.txt
    ├── plots/
    └── exports/
```

## Troubleshooting

### "No patterns for country X"
**Cause**: Country name mismatch between actuals and targets
**Fix**: Ensure country names are identical (case-sensitive)

### "Monthly totals don't match"
**Cause**: `--constrain-monthly` flag not used
**Fix**: Add flag or check for missing dates

### "Weekend lift too high"
**Cause**: Extreme historical outliers or data quality
**Fix**: Adjust `--outlier-threshold` or enable `--smooth-weekends`

### "No holiday effects"
**Cause**: Country code not recognized
**Fix**: Check country mapping in scripts, add manual mapping if needed

## Support & Documentation

- **Methodology**: See `references/methodology.md`
- **Grocery Patterns**: See `references/grocery_patterns.md`
- **Validation**: See `references/validation_guide.md`
- **Examples**: See output files in `results/`

## Limitations

This forecasting system:
- Assumes 2024-2025 patterns will repeat in 2026
- Cannot predict one-off events or disruptions
- Requires 1+ year of clean historical data per country
- Does not model promotions, pricing, or competition explicitly
- Works at country level (not regional/store)

For best results:
- Use 2+ years of historical data
- Validate forecasts before business use
- Monitor accuracy and update patterns quarterly
- Adjust for known future events manually

## License

Internal use only. Not for distribution.

## Version

v1.0 - Initial release for FY2026 planning
