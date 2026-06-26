---
name: marketing-mix-modeling-mmm-pipeline
description: End-to-end Marketing Mix Modeling pipeline with adstock, saturation, OLS regression, and budget optimization
triggers:
  - "build a marketing mix model"
  - "analyze marketing channel effectiveness"
  - "optimize marketing budget allocation"
  - "calculate marketing ROI by channel"
  - "implement adstock transformation"
  - "apply saturation curves to marketing spend"
  - "decompose sales into marketing contributions"
  - "run MMM pipeline"
---

# Marketing Mix Modeling Pipeline Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill provides expertise in using the Marketing Mix Modeling (MMM) end-to-end pipeline for analyzing marketing channel effectiveness, calculating ROI, and optimizing budget allocation using Python.

## What This Project Does

Marketing Mix Modeling decomposes historical sales/revenue into contributions from each marketing channel (TV, search, social, print, OOH) plus baseline organic demand. This pipeline:

- Applies **geometric adstock** transformations to model advertising carryover effects
- Applies **Hill saturation** curves to model diminishing returns
- Fits **OLS regression** to estimate channel contributions
- Optimizes budget allocation across channels using `scipy`
- Exports results for Power BI dashboard visualization

**Key insight**: Adstock must be applied *before* saturation because saturation is level-dependent and saturating raw spend would understate carryover effects.

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

**Core dependencies**:
- pandas, numpy
- statsmodels (OLS regression)
- scipy (optimization)
- jupyter (notebooks)

## Project Structure

```
data/
  dt_simulated_weekly.csv      # Raw weekly marketing spend + revenue
  dt_transformed.csv           # After adstock + saturation
  coefficients.csv             # Model coefficients & ROI
  optimized_budget.csv         # Optimization results

src/
  adstock.py                   # Geometric adstock function
  saturation.py                # Hill saturation function
  model.py                     # OLS regression wrapper
  optimizer.py                 # Budget allocation optimizer

notebooks/
  01_exploration.ipynb         # EDA
  02_transformations.ipynb     # Apply adstock + saturation
  03_model.ipynb              # Fit OLS model
  04_optimizer.ipynb          # Budget optimization
```

## Key Components

### 1. Geometric Adstock Transformation

Models advertising carryover effect where this week's exposure includes decayed impact from previous weeks.

```python
# src/adstock.py
import numpy as np
import pandas as pd

def geometric_adstock(x, theta):
    """
    Apply geometric adstock transformation.
    
    Parameters:
    -----------
    x : array-like
        Raw media spend or impressions (time series)
    theta : float
        Decay rate (0-1). Higher = longer carryover
        e.g., 0.5 means 50% of last week's effect carries over
    
    Returns:
    --------
    array : Adstocked series
    """
    x = np.array(x)
    adstocked = np.zeros_like(x, dtype=float)
    adstocked[0] = x[0]
    
    for t in range(1, len(x)):
        adstocked[t] = x[t] + theta * adstocked[t-1]
    
    return adstocked

# Usage example
import pandas as pd

df = pd.read_csv('data/dt_simulated_weekly.csv')
df['DATE'] = pd.to_datetime(df['DATE'])

# Apply to TV spend with 60% carryover
df['tv_adstocked'] = geometric_adstock(df['tv_S'], theta=0.6)

# Apply to all channels
channels = ['tv_S', 'ooh_S', 'print_S', 'facebook_S', 'search_S']
theta_values = {'tv_S': 0.6, 'ooh_S': 0.5, 'print_S': 0.4, 
                'facebook_S': 0.3, 'search_S': 0.2}

for channel in channels:
    col_name = f"{channel.replace('_S', '')}_adstocked"
    df[col_name] = geometric_adstock(df[channel], theta_values[channel])
```

### 2. Hill Saturation Transformation

Models diminishing returns as spend increases.

```python
# src/saturation.py
import numpy as np

def hill_saturation(x, alpha, gamma):
    """
    Apply Hill saturation curve.
    
    Parameters:
    -----------
    x : array-like
        Adstocked media variable
    alpha : float
        Half-saturation point (inflection point of the curve)
    gamma : float
        Shape parameter (>0). Higher = steeper curve
        
    Returns:
    --------
    array : Saturated series (0-1 scale)
    """
    x = np.array(x)
    return x**gamma / (alpha**gamma + x**gamma)

# Usage example
# Apply saturation AFTER adstock
alpha_values = {'tv': 0.5, 'ooh': 0.6, 'print': 0.7,
                'facebook': 0.4, 'search': 0.3}
gamma = 0.5  # Same shape for all channels

for channel in ['tv', 'ooh', 'print', 'facebook', 'search']:
    adstock_col = f"{channel}_adstocked"
    sat_col = f"{channel}_sat"
    df[sat_col] = hill_saturation(
        df[adstock_col], 
        alpha=alpha_values[channel],
        gamma=gamma
    )
```

### 3. OLS Regression Model

Fits revenue decomposition model.

```python
# src/model.py or in notebook
import statsmodels.api as sm
import pandas as pd

# Load transformed data
df = pd.read_csv('data/dt_transformed.csv')
df['DATE'] = pd.to_datetime(df['DATE'])

# Clean events column (Robyn data uses string "na" instead of null)
df['events'] = df['events'].replace('na', None)

# Define model variables
media_vars = ['tv_sat', 'ooh_sat', 'print_sat', 'facebook_sat', 'search_sat']
control_vars = ['competitor_sales_B', 'events']

# Create dummy for events
df = pd.get_dummies(df, columns=['events'], drop_first=True, dtype=float)
event_dummies = [col for col in df.columns if col.startswith('events_')]

# Prepare regression
X_vars = media_vars + ['competitor_sales_B'] + event_dummies
X = df[X_vars].fillna(0)
X = sm.add_constant(X)  # Add intercept
y = df['revenue']

# Fit OLS
model = sm.OLS(y, X).fit()

print(model.summary())
print(f"\nR-squared: {model.rsquared:.3f}")

# Extract coefficients for media channels only
coef_df = pd.DataFrame({
    'channel': media_vars,
    'coefficient': [model.params[var] for var in media_vars],
    'p_value': [model.pvalues[var] for var in media_vars]
})

# Calculate proxy ROI (contribution per unit spend)
# Need to link back to average spend per channel
raw_channels = ['tv_S', 'ooh_S', 'print_S', 'facebook_S', 'search_S']
avg_spend = df[raw_channels].mean()

coef_df['avg_weekly_spend'] = avg_spend.values
coef_df['avg_contribution'] = coef_df['coefficient'] * df[media_vars].mean().values
coef_df['proxy_roi'] = coef_df['avg_contribution'] / coef_df['avg_weekly_spend']

coef_df.to_csv('data/coefficients.csv', index=False)
print("\n", coef_df)
```

### 4. Budget Optimization

Maximize revenue under fixed total budget constraint.

```python
# src/optimizer.py
from scipy.optimize import minimize
import numpy as np
import pandas as pd

def optimize_budget(model, df, media_vars, raw_channels, total_budget):
    """
    Optimize budget allocation across channels.
    
    Parameters:
    -----------
    model : statsmodels OLS result
        Fitted regression model
    df : DataFrame
        Original data with transformations
    media_vars : list
        Saturated media variable names
    raw_channels : list
        Raw spend column names
    total_budget : float
        Fixed total weekly budget
    
    Returns:
    --------
    dict : Optimized allocation
    """
    # Get transformation parameters (need to store these from step 2)
    theta_values = {'tv_S': 0.6, 'ooh_S': 0.5, 'print_S': 0.4,
                    'facebook_S': 0.3, 'search_S': 0.2}
    alpha_values = {'tv': 0.5, 'ooh': 0.6, 'print': 0.7,
                    'facebook': 0.4, 'search': 0.3}
    gamma = 0.5
    
    # Get model coefficients
    coefs = {var: model.params[var] for var in media_vars}
    
    def predict_revenue(budget_allocation):
        """Predict revenue for given budget allocation"""
        # budget_allocation is array of spend values per channel
        revenue = model.params['const']
        
        for i, raw_ch in enumerate(raw_channels):
            spend = budget_allocation[i]
            channel_name = raw_ch.replace('_S', '')
            
            # Apply adstock (simplified - using only current spend)
            # In practice, need full time series context
            adstocked = spend  # Simplified
            
            # Apply saturation
            alpha = alpha_values[channel_name]
            saturated = adstocked**gamma / (alpha**gamma + adstocked**gamma)
            
            # Get contribution
            sat_var = f"{channel_name}_sat"
            revenue += coefs[sat_var] * saturated
        
        return revenue
    
    # Objective: maximize revenue = minimize negative revenue
    def objective(budget_allocation):
        return -predict_revenue(budget_allocation)
    
    # Constraint: sum of budgets = total_budget
    constraints = {'type': 'eq', 'fun': lambda x: np.sum(x) - total_budget}
    
    # Bounds: each channel >= 0
    bounds = [(0, total_budget) for _ in raw_channels]
    
    # Initial guess: equal allocation
    x0 = np.array([total_budget / len(raw_channels)] * len(raw_channels))
    
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
        print(f"Warning: optimization did not converge - {result.message}")
    
    # Format results
    current_avg = df[raw_channels].mean().values
    optimized = result.x
    
    results_df = pd.DataFrame({
        'channel': [ch.replace('_S', '') for ch in raw_channels],
        'current_budget': current_avg,
        'optimized_budget': optimized,
        'change_pct': ((optimized - current_avg) / current_avg * 100)
    })
    
    current_revenue = predict_revenue(current_avg)
    optimized_revenue = predict_revenue(optimized)
    uplift_pct = (optimized_revenue - current_revenue) / current_revenue * 100
    
    print(f"Current revenue: ${current_revenue:,.0f}")
    print(f"Optimized revenue: ${optimized_revenue:,.0f}")
    print(f"Uplift: +{uplift_pct:.1f}%\n")
    print(results_df)
    
    results_df.to_csv('data/optimized_budget.csv', index=False)
    return results_df

# Usage
total_weekly_budget = df[raw_channels].sum(axis=1).mean()
opt_results = optimize_budget(
    model=model,
    df=df,
    media_vars=media_vars,
    raw_channels=raw_channels,
    total_budget=total_weekly_budget
)
```

## Complete Workflow

### Step 1: Load and Clean Data

```python
import pandas as pd
import numpy as np

df = pd.read_csv('data/dt_simulated_weekly.csv')
df['DATE'] = pd.to_datetime(df['DATE'])
df['events'] = df['events'].replace('na', None)

# Verify data
print(df.head())
print(f"Date range: {df['DATE'].min()} to {df['DATE'].max()}")
print(f"Rows: {len(df)}")
```

### Step 2: Apply Transformations

```python
from src.adstock import geometric_adstock
from src.saturation import hill_saturation

# Define channels and parameters
channels = ['tv_S', 'ooh_S', 'print_S', 'facebook_S', 'search_S']
theta_map = {'tv_S': 0.6, 'ooh_S': 0.5, 'print_S': 0.4,
             'facebook_S': 0.3, 'search_S': 0.2}
alpha_map = {'tv': 0.5, 'ooh': 0.6, 'print': 0.7,
             'facebook': 0.4, 'search': 0.3}
gamma = 0.5

# Apply transformations (order matters: adstock first!)
for raw_channel in channels:
    channel_name = raw_channel.replace('_S', '')
    
    # Adstock
    adstock_col = f"{channel_name}_adstocked"
    df[adstock_col] = geometric_adstock(df[raw_channel], theta_map[raw_channel])
    
    # Saturation
    sat_col = f"{channel_name}_sat"
    df[sat_col] = hill_saturation(df[adstock_col], alpha_map[channel_name], gamma)

# Save
df.to_csv('data/dt_transformed.csv', index=False)
```

### Step 3: Fit Model

```python
import statsmodels.api as sm

# Prepare variables
media_vars = ['tv_sat', 'ooh_sat', 'print_sat', 'facebook_sat', 'search_sat']
df_model = pd.get_dummies(df, columns=['events'], drop_first=True, dtype=float)
event_cols = [c for c in df_model.columns if c.startswith('events_')]

X = df_model[media_vars + ['competitor_sales_B'] + event_cols].fillna(0)
X = sm.add_constant(X)
y = df_model['revenue']

model = sm.OLS(y, X).fit()
print(model.summary())
```

### Step 4: Optimize Budget

```python
from src.optimizer import optimize_budget

total_budget = df[channels].sum(axis=1).mean()
results = optimize_budget(model, df, media_vars, channels, total_budget)
```

## Configuration

**Adstock Parameters (theta)**:
- 0.0 = no carryover
- 0.5 = moderate carryover
- 0.9 = strong carryover
- Typical range: 0.2-0.7

**Saturation Parameters**:
- `alpha`: half-saturation point (scale-dependent, use percentiles of spend)
- `gamma`: shape parameter (0.3-0.7 typical, <1 = diminishing returns)

## Common Patterns

### Calculate Channel Contributions

```python
# Decompose revenue into channel contributions
contributions = {}
for var in media_vars:
    contributions[var] = model.params[var] * df[var]

contrib_df = pd.DataFrame(contributions)
contrib_df['baseline'] = model.params['const']
contrib_df['total_predicted'] = model.predict(X)
contrib_df['actual_revenue'] = y.values

# Weekly stack
print(contrib_df.head())

# Average contribution per channel
avg_contrib = contrib_df[media_vars].mean()
print("\nAverage weekly contribution:\n", avg_contrib)
```

### Export for Power BI

```python
# Time series for line chart
powerbi_ts = df[['DATE', 'revenue']].copy()
powerbi_ts['predicted'] = model.predict(X)
powerbi_ts.to_csv('data/powerbi/timeseries.csv', index=False)

# Channel contributions
powerbi_contrib = avg_contrib.reset_index()
powerbi_contrib.columns = ['channel', 'contribution']
powerbi_contrib.to_csv('data/powerbi/contributions.csv', index=False)

# Optimization results already saved in optimize_budget()
```

### Scenario Analysis

```python
# What-if: increase TV budget by 20%
scenario_budget = df[channels].mean().values.copy()
tv_index = channels.index('tv_S')
scenario_budget[tv_index] *= 1.2

# Redistribute from other channels to keep total constant
deficit = scenario_budget.sum() - total_budget
for i in range(len(scenario_budget)):
    if i != tv_index:
        scenario_budget[i] -= deficit / (len(channels) - 1)

from src.optimizer import predict_revenue  # Need to expose this
scenario_revenue = predict_revenue(scenario_budget)
print(f"TV +20% scenario revenue: ${scenario_revenue:,.0f}")
```

## Troubleshooting

**Issue**: Model R² is low (<0.7)
- Check for multicollinearity between channels (VIF)
- Verify transformations applied correctly
- Add more control variables (seasonality, promotions)
- Try different adstock/saturation parameters

**Issue**: Negative coefficients for media channels
- Usually indicates overspending/saturation
- Check for correlation with baseline/events
- Verify correct order: adstock → saturation
- May need interaction terms or non-linear baseline

**Issue**: Optimization fails to converge
- Simplify constraints (wider bounds)
- Use better initial guess (current allocation)
- Try different optimization method (`method='trust-constr'`)
- Check if objective function is smooth

**Issue**: String "na" not treated as null in events column
```python
# Robyn data quirk - clean explicitly
df['events'] = df['events'].replace('na', None).replace('NA', None)
```

**Issue**: DateTime parsing errors
```python
df['DATE'] = pd.to_datetime(df['DATE'], format='%Y-%m-%d', errors='coerce')
```

## Advanced: Bayesian MMM Migration

For uncertainty quantification, migrate to PyMC:

```python
# Future enhancement (not in current repo)
import pymc as pm
import pytensor.tensor as pt

with pm.Model() as mmm:
    # Priors on coefficients
    beta_tv = pm.HalfNormal('beta_tv', sigma=10)
    # ... other channels
    
    # Likelihood
    mu = beta_tv * df['tv_sat'] + ...  # + other channels
    sigma = pm.HalfNormal('sigma', sigma=5)
    revenue = pm.Normal('revenue', mu=mu, sigma=sigma, observed=df['revenue'])
    
    # Sample
    trace = pm.sample(2000, tune=1000)
```

This skill enables AI agents to help developers implement Marketing Mix Models, from data preparation through optimization and visualization.
