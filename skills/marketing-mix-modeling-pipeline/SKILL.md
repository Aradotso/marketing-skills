---
name: marketing-mix-modeling-pipeline
description: End-to-end Marketing Mix Modeling pipeline with adstock, saturation, OLS regression, and budget optimization
triggers:
  - "build a marketing mix model"
  - "calculate channel ROI with adstock"
  - "optimize marketing budget allocation"
  - "apply saturation curves to media spend"
  - "marketing mix modeling pipeline"
  - "measure incremental revenue by channel"
  - "adstock transformation for advertising"
  - "budget optimizer for marketing channels"
---

# Marketing Mix Modeling Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project provides an end-to-end Marketing Mix Modeling (MMM) pipeline in Python. It transforms raw weekly marketing spend into channel-level revenue contributions using adstock (carryover) and saturation (diminishing returns) transformations, fits an OLS regression, and optimizes budget allocation to maximize revenue.

## What It Does

- **Adstock transformation**: models the carryover effect of advertising (geometric decay)
- **Hill saturation**: models diminishing returns as spend increases
- **OLS regression**: decomposes revenue into channel contributions + baseline
- **Budget optimization**: reallocates spend across channels to maximize predicted revenue
- **Privacy-safe**: uses aggregate historical data, no user-level tracking

## Installation

```bash
# Clone the repository
git clone https://github.com/francescaetnom-wq/Marketing-Mix-Modeling-End-to-End-Pipeline.git
cd Marketing-Mix-Modeling-End-to-End-Pipeline

# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

**Key dependencies** (add to `requirements.txt` if missing):
```
pandas>=1.5.0
numpy>=1.23.0
statsmodels>=0.14.0
scipy>=1.10.0
jupyter>=1.0.0
matplotlib>=3.6.0
seaborn>=0.12.0
```

## Project Structure

```
mmm_project/
├── data/
│   ├── dt_simulated_weekly.csv      # raw input (Robyn demo data)
│   ├── dt_transformed.csv           # with adstock + saturation
│   ├── coefficients.csv             # model coefficients
│   └── optimized_budget.csv         # optimization results
├── notebooks/
│   ├── 01_exploration.ipynb
│   ├── 02_transformations.ipynb
│   ├── 03_model.ipynb
│   └── 04_optimizer.ipynb
├── src/
│   ├── adstock.py                   # adstock transformation
│   ├── saturation.py                # Hill saturation
│   ├── model.py                     # OLS wrapper
│   └── optimizer.py                 # budget allocation
└── requirements.txt
```

## Core Transformations

### 1. Geometric Adstock

Models carryover effect: today's exposure includes decayed contribution from previous periods.

**Formula**: `X_t = x_t + θ × X_{t-1}` where θ ∈ [0,1] is decay rate.

```python
# src/adstock.py
import numpy as np
import pandas as pd

def geometric_adstock(x, theta):
    """
    Apply geometric adstock transformation.
    
    Args:
        x: array-like, raw media spend
        theta: float [0,1], decay rate (0.5 = 50% carryover)
    
    Returns:
        array of same shape with adstock applied
    """
    x = np.array(x, dtype=float)
    adstocked = np.zeros_like(x)
    adstocked[0] = x[0]
    
    for t in range(1, len(x)):
        adstocked[t] = x[t] + theta * adstocked[t-1]
    
    return adstocked

# Usage
spend = [100, 0, 0, 100, 0]
adstocked = geometric_adstock(spend, theta=0.5)
# [100.0, 50.0, 25.0, 112.5, 56.25]
```

**Applying to multiple channels**:
```python
import pandas as pd

df = pd.read_csv('data/dt_simulated_weekly.csv')

# Channel columns
channels = ['tv_S', 'ooh_S', 'print_S', 'facebook_S', 'search_S']

# Apply adstock with channel-specific decay rates
adstock_params = {
    'tv_S': 0.5,
    'ooh_S': 0.4,
    'print_S': 0.3,
    'facebook_S': 0.2,
    'search_S': 0.1
}

for channel in channels:
    theta = adstock_params[channel]
    df[f'{channel}_adstocked'] = geometric_adstock(df[channel].values, theta)
```

### 2. Hill Saturation

Models diminishing returns: incremental revenue decreases as spend increases.

**Formula**: `y = α × x^s / (K^s + x^s)` where:
- α = maximum response
- K = half-saturation point
- s = slope (steepness)

```python
# src/saturation.py
import numpy as np

def hill_saturation(x, alpha, K, s):
    """
    Apply Hill saturation curve.
    
    Args:
        x: array-like, adstocked media signal
        alpha: float, maximum response (scale)
        K: float, half-saturation point
        s: float, slope/steepness (typically 1-3)
    
    Returns:
        saturated signal
    """
    x = np.array(x, dtype=float)
    return alpha * np.power(x, s) / (np.power(K, s) + np.power(x, s))

# Usage
adstocked = [0, 50, 100, 200, 500]
saturated = hill_saturation(adstocked, alpha=1.0, K=100, s=2)
```

**Combining adstock + saturation** (order matters!):
```python
# CORRECT: adstock first, then saturation
for channel in channels:
    # Step 1: adstock
    adstocked = geometric_adstock(df[channel].values, theta=0.5)
    
    # Step 2: saturation on adstocked values
    saturated = hill_saturation(adstocked, alpha=1.0, K=100, s=2)
    
    df[f'{channel}_transformed'] = saturated
```

## OLS Regression Model

Fit a regression: `revenue = intercept + Σ(β_i × channel_i_transformed) + controls`

```python
# src/model.py or in notebook
import statsmodels.api as sm
import pandas as pd

def fit_mmm(df, channel_cols, control_cols, target='revenue'):
    """
    Fit OLS regression for MMM.
    
    Args:
        df: DataFrame with transformed channels
        channel_cols: list of transformed channel column names
        control_cols: list of control variables (competitor_sales, events)
        target: target variable (revenue)
    
    Returns:
        statsmodels RegressionResults object
    """
    # Build feature matrix
    X = df[channel_cols + control_cols].copy()
    
    # Add intercept
    X = sm.add_constant(X)
    
    # Target
    y = df[target]
    
    # Fit OLS
    model = sm.OLS(y, X).fit()
    
    return model

# Example usage
transformed_channels = [
    'tv_S_transformed',
    'ooh_S_transformed', 
    'print_S_transformed',
    'facebook_S_transformed',
    'search_S_transformed'
]

controls = ['competitor_sales_B']

model = fit_mmm(df, transformed_channels, controls, target='revenue')

print(model.summary())
print(f"\nR²: {model.rsquared:.3f}")

# Extract coefficients
coef_df = pd.DataFrame({
    'channel': model.params.index,
    'coefficient': model.params.values,
    'p_value': model.pvalues.values
})
```

## Channel Contribution & ROI

```python
def calculate_contributions(df, model, channel_cols):
    """
    Calculate each channel's total contribution to revenue.
    
    Returns DataFrame with channel, contribution, avg_spend, proxy_roi
    """
    contributions = []
    
    for col in channel_cols:
        coef = model.params[col]
        total_contrib = (df[col] * coef).sum()
        
        # Map back to original spend column
        raw_col = col.replace('_transformed', '')
        avg_spend = df[raw_col].mean()
        
        contributions.append({
            'channel': raw_col,
            'coefficient': coef,
            'total_contribution': total_contrib,
            'avg_weekly_spend': avg_spend,
            'proxy_roi': total_contrib / df[raw_col].sum() if df[raw_col].sum() > 0 else 0
        })
    
    return pd.DataFrame(contributions)

contrib_df = calculate_contributions(df, model, transformed_channels)
print(contrib_df.sort_values('proxy_roi', ascending=False))
```

## Budget Optimization

Reallocate spend to maximize predicted revenue.

```python
# src/optimizer.py
from scipy.optimize import minimize
import numpy as np

def optimize_budget(model, channel_cols, total_budget, adstock_params, saturation_params):
    """
    Find optimal budget allocation across channels.
    
    Args:
        model: fitted statsmodels model
        channel_cols: list of transformed channel names
        total_budget: total weekly budget to allocate
        adstock_params: dict {channel: theta}
        saturation_params: dict {channel: {'alpha': ..., 'K': ..., 's': ...}}
    
    Returns:
        dict with optimized budget per channel
    """
    n_channels = len(channel_cols)
    
    def objective(budget_allocation):
        """Negative revenue (we minimize)"""
        revenue = model.params['const']  # intercept
        
        for i, col in enumerate(channel_cols):
            raw_col = col.replace('_transformed', '')
            spend = budget_allocation[i]
            
            # Apply transformations
            theta = adstock_params[raw_col]
            adstocked = spend  # simplified single-period
            
            sat = saturation_params[raw_col]
            saturated = sat['alpha'] * (adstocked ** sat['s']) / \
                       (sat['K'] ** sat['s'] + adstocked ** sat['s'])
            
            revenue += model.params[col] * saturated
        
        return -revenue  # minimize negative = maximize revenue
    
    # Constraints
    constraints = [
        {'type': 'eq', 'fun': lambda x: np.sum(x) - total_budget}  # sum = budget
    ]
    
    # Bounds: each channel >= 0
    bounds = [(0, total_budget) for _ in range(n_channels)]
    
    # Initial guess: equal split
    x0 = np.full(n_channels, total_budget / n_channels)
    
    # Optimize
    result = minimize(
        objective,
        x0,
        method='SLSQP',
        bounds=bounds,
        constraints=constraints,
        options={'maxiter': 1000}
    )
    
    if not result.success:
        print(f"Warning: optimization did not converge: {result.message}")
    
    # Build result dict
    optimal = {}
    for i, col in enumerate(channel_cols):
        raw_col = col.replace('_transformed', '')
        optimal[raw_col] = result.x[i]
    
    return optimal, -result.fun  # return allocation and predicted revenue

# Example usage
total_weekly_budget = 100000

adstock_params = {
    'tv_S': 0.5,
    'ooh_S': 0.4,
    'print_S': 0.3,
    'facebook_S': 0.2,
    'search_S': 0.1
}

saturation_params = {
    'tv_S': {'alpha': 1.0, 'K': 5000, 's': 2},
    'ooh_S': {'alpha': 1.0, 'K': 3000, 's': 2},
    'print_S': {'alpha': 1.0, 'K': 2000, 's': 2},
    'facebook_S': {'alpha': 1.0, 'K': 4000, 's': 2},
    'search_S': {'alpha': 1.0, 'K': 3500, 's': 2}
}

optimal_budget, predicted_revenue = optimize_budget(
    model, 
    transformed_channels, 
    total_weekly_budget,
    adstock_params,
    saturation_params
)

print("Optimized budget allocation:")
for channel, amount in optimal_budget.items():
    print(f"  {channel}: ${amount:,.0f}")
print(f"\nPredicted revenue: ${predicted_revenue:,.0f}")
```

## Full Workflow Example

```python
import pandas as pd
import numpy as np
import statsmodels.api as sm
from src.adstock import geometric_adstock
from src.saturation import hill_saturation

# 1. Load data
df = pd.read_csv('data/dt_simulated_weekly.csv', parse_dates=['DATE'])

# Clean events column (Robyn encodes "no event" as string "na")
df['events'] = df['events'].replace('na', np.nan)

# 2. Apply transformations
channels = ['tv_S', 'ooh_S', 'print_S', 'facebook_S', 'search_S']

adstock_params = {
    'tv_S': 0.5, 'ooh_S': 0.4, 'print_S': 0.3,
    'facebook_S': 0.2, 'search_S': 0.1
}

saturation_params = {
    'tv_S': {'alpha': 1.0, 'K': 5000, 's': 2},
    'ooh_S': {'alpha': 1.0, 'K': 3000, 's': 2},
    'print_S': {'alpha': 1.0, 'K': 2000, 's': 2},
    'facebook_S': {'alpha': 1.0, 'K': 4000, 's': 2},
    'search_S': {'alpha': 1.0, 'K': 3500, 's': 2}
}

for channel in channels:
    # Adstock
    adstocked = geometric_adstock(df[channel].values, adstock_params[channel])
    
    # Saturation
    sat = saturation_params[channel]
    saturated = hill_saturation(adstocked, sat['alpha'], sat['K'], sat['s'])
    
    df[f'{channel}_transformed'] = saturated

# 3. Fit model
transformed_cols = [f'{c}_transformed' for c in channels]
control_cols = ['competitor_sales_B']

X = df[transformed_cols + control_cols].copy()
X = sm.add_constant(X)
y = df['revenue']

model = sm.OLS(y, X).fit()
print(model.summary())

# 4. Calculate contributions
contrib = []
for col in transformed_cols:
    raw_col = col.replace('_transformed', '')
    total_contrib = (df[col] * model.params[col]).sum()
    total_spend = df[raw_col].sum()
    
    contrib.append({
        'channel': raw_col,
        'contribution': total_contrib,
        'roi': total_contrib / total_spend if total_spend > 0 else 0
    })

contrib_df = pd.DataFrame(contrib)
print("\nChannel ROI:")
print(contrib_df.sort_values('roi', ascending=False))

# 5. Save outputs
df.to_csv('data/dt_transformed.csv', index=False)
contrib_df.to_csv('data/coefficients.csv', index=False)
```

## Common Patterns

### Creating Dummy Variables for Events

```python
# If events is categorical
df['event_dummy'] = df['events'].notna().astype(int)

# Or one-hot encode specific events
event_dummies = pd.get_dummies(df['events'], prefix='event', dummy_na=False)
df = pd.concat([df, event_dummies], axis=1)
```

### Handling Seasonality

```python
# Add month or week-of-year dummies
df['month'] = df['DATE'].dt.month
month_dummies = pd.get_dummies(df['month'], prefix='month', drop_first=True)
df = pd.concat([df, month_dummies], axis=1)

# Include in controls
control_cols += list(month_dummies.columns)
```

### Residual Diagnostics

```python
# Check model assumptions
import matplotlib.pyplot as plt
import seaborn as sns

residuals = model.resid
fitted = model.fittedvalues

# Residuals vs fitted
plt.figure(figsize=(10, 5))
plt.subplot(1, 2, 1)
plt.scatter(fitted, residuals, alpha=0.5)
plt.axhline(0, color='red', linestyle='--')
plt.xlabel('Fitted')
plt.ylabel('Residuals')
plt.title('Residuals vs Fitted')

# Q-Q plot
plt.subplot(1, 2, 2)
sm.qqplot(residuals, line='s', ax=plt.gca())
plt.title('Q-Q Plot')
plt.tight_layout()
plt.show()
```

## Troubleshooting

### Issue: Model R² is very low

**Solution**: Check transformation parameters. Adstock decay and saturation K should be calibrated to your data scale.

```python
# Explore different theta values
for theta in [0.3, 0.5, 0.7]:
    adstocked = geometric_adstock(df['tv_S'].values, theta)
    print(f"theta={theta}, mean={adstocked.mean():.0f}, max={adstocked.max():.0f}")
```

### Issue: Optimization returns equal budgets

**Solution**: Saturation curves may be too flat. Lower K (half-saturation point) or increase s (steepness).

```python
# More aggressive saturation
saturation_params['tv_S']['K'] = 2000  # lower = earlier saturation
saturation_params['tv_S']['s'] = 3     # higher = steeper
```

### Issue: Negative coefficients

**Solution**: Check for multicollinearity or incorrectly applied transformations.

```python
# Check correlation
corr_matrix = df[transformed_cols].corr()
print(corr_matrix)

# Drop highly correlated channels (|corr| > 0.8)
```

### Issue: "Adstock then saturation" vs. "Saturation then adstock"

**Answer**: Always **adstock first**, then saturation. Saturation is non-linear and level-dependent; applying it before adstock would undervalue channels with strong carryover.

## Exporting for Power BI

```python
# Create aggregated metrics table
summary = pd.DataFrame({
    'metric': ['R²', 'Total Revenue', 'Model Predicted', 'Residual Std'],
    'value': [
        model.rsquared,
        df['revenue'].sum(),
        model.fittedvalues.sum(),
        np.std(model.resid)
    ]
})

# Channel-level table
channel_summary = contrib_df.copy()
channel_summary['avg_weekly_spend'] = [df[c].mean() for c in channels]

# Time series for dashboard
timeseries = df[['DATE', 'revenue']].copy()
timeseries['predicted'] = model.fittedvalues

# Export
summary.to_csv('data/powerbi/model_summary.csv', index=False)
channel_summary.to_csv('data/powerbi/channel_metrics.csv', index=False)
timeseries.to_csv('data/powerbi/timeseries.csv', index=False)
```

## References

- **Robyn** (Meta's MMM): [github.com/facebookexperimental/Robyn](https://github.com/facebookexperimental/Robyn)
- **PyMC Marketing**: Bayesian MMM alternative with uncertainty quantification
- **Adstock literature**: Koyck (1954), Clarke (1976)
