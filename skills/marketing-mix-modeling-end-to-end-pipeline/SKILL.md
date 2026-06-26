---
name: marketing-mix-modeling-end-to-end-pipeline
description: Build end-to-end Marketing Mix Models with adstock, saturation, OLS regression, and budget optimization in Python
triggers:
  - "build a marketing mix model"
  - "implement MMM with adstock and saturation"
  - "optimize marketing budget allocation"
  - "calculate channel ROI for marketing spend"
  - "apply geometric adstock transformation"
  - "run marketing attribution analysis"
  - "create budget optimizer for marketing channels"
  - "decompose revenue by marketing channel"
---

# Marketing Mix Modeling End-to-End Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project provides a complete Marketing Mix Modeling (MMM) pipeline in Python that transforms raw marketing spend data into actionable channel insights and optimized budget allocations. It includes geometric adstock transformation, Hill saturation curves, OLS regression modeling, and constrained budget optimization.

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

**Key Dependencies:**
- `pandas` — data manipulation
- `numpy` — numerical operations
- `statsmodels` — OLS regression
- `scipy` — optimization algorithms
- `matplotlib` / `seaborn` — visualization

## Project Structure

```
mmm_project/
├── data/
│   ├── dt_simulated_weekly.csv      # Input: weekly spend & revenue
│   ├── dt_transformed.csv           # Output: with adstock/saturation
│   ├── coefficients.csv             # Output: model coefficients
│   └── optimized_budget.csv         # Output: budget recommendations
├── src/
│   ├── adstock.py                   # Geometric adstock
│   ├── saturation.py                # Hill saturation
│   ├── model.py                     # OLS wrapper
│   └── optimizer.py                 # Budget allocation
└── notebooks/
    ├── 01_exploration.ipynb
    ├── 02_transformations.ipynb
    ├── 03_model.ipynb
    └── 04_optimizer.ipynb
```

## Core Transformations

### 1. Geometric Adstock

Models the carryover effect of advertising — this week's exposure includes decayed contributions from previous weeks.

**Implementation (`src/adstock.py`):**

```python
import numpy as np
import pandas as pd

def geometric_adstock(x, theta, L_max=8):
    """
    Apply geometric adstock transformation.
    
    Parameters:
    -----------
    x : array-like
        Raw media spend or impressions
    theta : float
        Decay rate (0 to 1). Higher = longer carryover
    L_max : int
        Maximum lag window
        
    Returns:
    --------
    array : Adstocked signal
    """
    x = np.array(x)
    adstocked = np.zeros_like(x, dtype=float)
    
    for t in range(len(x)):
        for lag in range(min(t + 1, L_max)):
            adstocked[t] += x[t - lag] * (theta ** lag)
    
    return adstocked

# Example usage
spend = [100, 150, 120, 90, 110]
adstocked_spend = geometric_adstock(spend, theta=0.5, L_max=4)
# adstocked_spend accounts for carryover from previous weeks
```

**Apply to DataFrame:**

```python
import pandas as pd

df = pd.read_csv('data/dt_simulated_weekly.csv')

# Define adstock parameters per channel
adstock_params = {
    'tv_S': 0.7,      # High carryover
    'search_S': 0.3,   # Low carryover
    'facebook_S': 0.5,
    'print_S': 0.6,
    'ooh_S': 0.4
}

# Apply adstock to each channel
for channel, theta in adstock_params.items():
    df[f'{channel}_adstocked'] = geometric_adstock(
        df[channel].values, 
        theta=theta, 
        L_max=8
    )
```

### 2. Hill Saturation

Models diminishing returns — the first dollar spent is more effective than the millionth.

**Implementation (`src/saturation.py`):**

```python
import numpy as np

def hill_saturation(x, alpha, gamma):
    """
    Apply Hill saturation transformation.
    
    Parameters:
    -----------
    x : array-like
        Input signal (usually adstocked spend)
    alpha : float
        Half-saturation point (inflection)
    gamma : float
        Shape parameter (> 1 for S-curve)
        
    Returns:
    --------
    array : Saturated signal
    """
    x = np.array(x)
    return (x ** gamma) / (alpha ** gamma + x ** gamma)

# Example usage
adstocked = np.array([50, 100, 150, 200, 250])
saturated = hill_saturation(adstocked, alpha=100, gamma=2.0)
# saturated shows diminishing marginal returns
```

**Apply After Adstock (Order Matters!):**

```python
# Saturation parameters per channel
saturation_params = {
    'tv_S': {'alpha': 150, 'gamma': 2.0},
    'search_S': {'alpha': 80, 'gamma': 1.8},
    'facebook_S': {'alpha': 100, 'gamma': 1.9},
    'print_S': {'alpha': 120, 'gamma': 2.1},
    'ooh_S': {'alpha': 90, 'gamma': 2.0}
}

# Apply saturation to adstocked columns
for channel, params in saturation_params.items():
    adstocked_col = f'{channel}_adstocked'
    df[f'{channel}_transformed'] = hill_saturation(
        df[adstocked_col].values,
        alpha=params['alpha'],
        gamma=params['gamma']
    )

# Save transformed data
df.to_csv('data/dt_transformed.csv', index=False)
```

## OLS Regression Model

### Training the Model

**Implementation (`src/model.py`):**

```python
import pandas as pd
import statsmodels.api as sm

def fit_mmm_model(df, transformed_channels, controls, target='revenue'):
    """
    Fit OLS regression for MMM.
    
    Parameters:
    -----------
    df : DataFrame
        Input data with transformed channels
    transformed_channels : list
        Column names of transformed media variables
    controls : list
        Control variables (e.g., competitor_sales_B, events)
    target : str
        Revenue or sales column
        
    Returns:
    --------
    model : statsmodels RegressionResults
    """
    # Prepare feature matrix
    X = df[transformed_channels + controls].copy()
    
    # Handle categorical controls (one-hot encode events)
    if 'events' in controls:
        X = pd.get_dummies(X, columns=['events'], drop_first=True)
    
    # Add constant
    X = sm.add_constant(X)
    
    # Target variable
    y = df[target]
    
    # Fit OLS
    model = sm.OLS(y, X).fit()
    
    return model

# Example usage
df = pd.read_csv('data/dt_transformed.csv')

transformed_channels = [
    'tv_S_transformed',
    'search_S_transformed',
    'facebook_S_transformed',
    'print_S_transformed',
    'ooh_S_transformed'
]

controls = ['competitor_sales_B', 'events']

model = fit_mmm_model(df, transformed_channels, controls, target='revenue')
print(model.summary())
print(f"\nR-squared: {model.rsquared:.3f}")
```

### Extract Coefficients & ROI

```python
import pandas as pd

def extract_coefficients(model, channel_names):
    """Extract coefficients and calculate proxy ROI."""
    coefs = model.params[channel_names]
    pvalues = model.pvalues[channel_names]
    
    results = pd.DataFrame({
        'channel': channel_names,
        'coefficient': coefs.values,
        'p_value': pvalues.values
    })
    
    return results

# Get coefficients
channel_names = [col for col in model.params.index if '_transformed' in col]
coef_df = extract_coefficients(model, channel_names)

# Calculate proxy ROI (revenue per unit spend)
# Match transformed columns to original spend columns
spend_mapping = {
    'tv_S_transformed': 'tv_S',
    'search_S_transformed': 'search_S',
    'facebook_S_transformed': 'facebook_S',
    'print_S_transformed': 'print_S',
    'ooh_S_transformed': 'ooh_S'
}

for idx, row in coef_df.iterrows():
    channel_transformed = row['channel']
    channel_raw = spend_mapping[channel_transformed]
    
    # Average contribution / average spend
    avg_spend = df[channel_raw].mean()
    avg_contribution = row['coefficient'] * df[channel_transformed].mean()
    
    coef_df.loc[idx, 'avg_spend'] = avg_spend
    coef_df.loc[idx, 'avg_contribution'] = avg_contribution
    coef_df.loc[idx, 'proxy_roi'] = avg_contribution / avg_spend if avg_spend > 0 else 0

coef_df.to_csv('data/coefficients.csv', index=False)
print(coef_df)
```

## Budget Optimization

Find the optimal budget allocation to maximize revenue under a fixed total budget constraint.

**Implementation (`src/optimizer.py`):**

```python
import numpy as np
from scipy.optimize import minimize

def optimize_budget(model, df, channel_mapping, total_budget, 
                    adstock_params, saturation_params):
    """
    Optimize budget allocation across channels.
    
    Parameters:
    -----------
    model : statsmodels RegressionResults
        Fitted OLS model
    df : DataFrame
        Historical data (for baseline calculation)
    channel_mapping : dict
        Maps transformed column names to raw spend columns
    total_budget : float
        Total budget constraint
    adstock_params : dict
        Adstock theta per channel
    saturation_params : dict
        Saturation alpha/gamma per channel
        
    Returns:
    --------
    dict : Optimized budget allocation
    """
    from adstock import geometric_adstock
    from saturation import hill_saturation
    
    channels = list(channel_mapping.keys())
    n_channels = len(channels)
    
    # Extract coefficients
    coefs = {col: model.params[col] for col in channels}
    
    def predict_revenue(budget_allocation):
        """Predict revenue given budget allocation."""
        revenue = 0
        
        for i, channel_raw in enumerate(channels):
            spend = budget_allocation[i]
            channel_transformed = channel_mapping[channel_raw]
            
            # Apply transformations
            adstocked = geometric_adstock(
                [spend], 
                theta=adstock_params[channel_raw], 
                L_max=1  # Single period optimization
            )[0]
            
            saturated = hill_saturation(
                [adstocked],
                alpha=saturation_params[channel_raw]['alpha'],
                gamma=saturation_params[channel_raw]['gamma']
            )[0]
            
            # Multiply by coefficient
            revenue += saturated * coefs[channel_transformed]
        
        return revenue
    
    # Objective: negative revenue (minimize = maximize revenue)
    def objective(budget_allocation):
        return -predict_revenue(budget_allocation)
    
    # Constraints
    constraints = [
        {'type': 'eq', 'fun': lambda x: np.sum(x) - total_budget}  # Sum = total
    ]
    
    # Bounds: non-negative spend
    bounds = [(0, total_budget) for _ in range(n_channels)]
    
    # Initial guess: equal split
    x0 = np.array([total_budget / n_channels] * n_channels)
    
    # Optimize
    result = minimize(
        objective,
        x0,
        method='SLSQP',
        bounds=bounds,
        constraints=constraints
    )
    
    # Format results
    optimized_budget = {
        channel: result.x[i] 
        for i, channel in enumerate(channels)
    }
    
    return optimized_budget, -result.fun  # Return revenue (negate objective)

# Example usage
channel_mapping = {
    'tv_S': 'tv_S_transformed',
    'search_S': 'search_S_transformed',
    'facebook_S': 'facebook_S_transformed',
    'print_S': 'print_S_transformed',
    'ooh_S': 'ooh_S_transformed'
}

# Calculate current total budget
current_budget = df[list(channel_mapping.keys())].sum(axis=1).mean()

# Optimize
optimized, projected_revenue = optimize_budget(
    model=model,
    df=df,
    channel_mapping=channel_mapping,
    total_budget=current_budget,
    adstock_params=adstock_params,
    saturation_params=saturation_params
)

# Compare current vs optimized
current_allocation = {ch: df[ch].mean() for ch in channel_mapping.keys()}

comparison = pd.DataFrame({
    'channel': list(channel_mapping.keys()),
    'current_budget': [current_allocation[ch] for ch in channel_mapping.keys()],
    'optimized_budget': [optimized[ch] for ch in channel_mapping.keys()]
})

comparison['change_pct'] = (
    (comparison['optimized_budget'] - comparison['current_budget']) / 
    comparison['current_budget'] * 100
)

comparison.to_csv('data/optimized_budget.csv', index=False)
print(comparison)
```

## Complete Workflow

```python
import pandas as pd
import numpy as np
from src.adstock import geometric_adstock
from src.saturation import hill_saturation
from src.model import fit_mmm_model
from src.optimizer import optimize_budget

# 1. Load data
df = pd.read_csv('data/dt_simulated_weekly.csv')
df['DATE'] = pd.to_datetime(df['DATE'])

# Clean events column (handle literal "na" string)
df['events'] = df['events'].replace('na', np.nan)

# 2. Define transformation parameters
channels = ['tv_S', 'search_S', 'facebook_S', 'print_S', 'ooh_S']

adstock_params = {
    'tv_S': 0.7,
    'search_S': 0.3,
    'facebook_S': 0.5,
    'print_S': 0.6,
    'ooh_S': 0.4
}

saturation_params = {
    'tv_S': {'alpha': 150, 'gamma': 2.0},
    'search_S': {'alpha': 80, 'gamma': 1.8},
    'facebook_S': {'alpha': 100, 'gamma': 1.9},
    'print_S': {'alpha': 120, 'gamma': 2.1},
    'ooh_S': {'alpha': 90, 'gamma': 2.0}
}

# 3. Apply transformations (adstock THEN saturation)
for channel in channels:
    # Adstock
    df[f'{channel}_adstocked'] = geometric_adstock(
        df[channel].values,
        theta=adstock_params[channel],
        L_max=8
    )
    
    # Saturation
    df[f'{channel}_transformed'] = hill_saturation(
        df[f'{channel}_adstocked'].values,
        alpha=saturation_params[channel]['alpha'],
        gamma=saturation_params[channel]['gamma']
    )

# 4. Fit model
transformed_channels = [f'{ch}_transformed' for ch in channels]
controls = ['competitor_sales_B', 'events']

model = fit_mmm_model(df, transformed_channels, controls, target='revenue')
print(f"Model R²: {model.rsquared:.3f}")

# 5. Optimize budget
channel_mapping = {ch: f'{ch}_transformed' for ch in channels}
total_budget = df[channels].sum(axis=1).mean()

optimized, projected_revenue = optimize_budget(
    model=model,
    df=df,
    channel_mapping=channel_mapping,
    total_budget=total_budget,
    adstock_params=adstock_params,
    saturation_params=saturation_params
)

# 6. Calculate uplift
current_avg_revenue = df['revenue'].mean()
uplift_pct = (projected_revenue - current_avg_revenue) / current_avg_revenue * 100

print(f"\nProjected revenue uplift: {uplift_pct:.1f}%")
print("\nOptimized allocation:")
for channel, budget in optimized.items():
    current = df[channel].mean()
    change = (budget - current) / current * 100
    print(f"  {channel}: {budget:.0f} ({change:+.1f}%)")
```

## Common Patterns

### Pattern 1: Sensitivity Analysis

Test how different adstock/saturation parameters affect model fit:

```python
from itertools import product

theta_range = [0.3, 0.5, 0.7]
gamma_range = [1.5, 2.0, 2.5]

results = []

for theta in theta_range:
    for gamma in gamma_range:
        # Apply transformations
        df_test = df.copy()
        df_test['tv_adstocked'] = geometric_adstock(df_test['tv_S'], theta=theta)
        df_test['tv_transformed'] = hill_saturation(
            df_test['tv_adstocked'], alpha=150, gamma=gamma
        )
        
        # Fit model
        model_test = fit_mmm_model(
            df_test, ['tv_transformed'] + other_channels, controls
        )
        
        results.append({
            'theta': theta,
            'gamma': gamma,
            'r_squared': model_test.rsquared,
            'tv_coef': model_test.params['tv_transformed']
        })

results_df = pd.DataFrame(results).sort_values('r_squared', ascending=False)
print(results_df.head())
```

### Pattern 2: Time-Based Validation

Split data into train/test periods:

```python
# Split at 80% mark
split_date = df['DATE'].quantile(0.8)
train = df[df['DATE'] <= split_date]
test = df[df['DATE'] > split_date]

# Fit on train
model_train = fit_mmm_model(train, transformed_channels, controls)

# Predict on test
X_test = test[transformed_channels + controls]
X_test = pd.get_dummies(X_test, columns=['events'], drop_first=True)
X_test = sm.add_constant(X_test)

# Align columns
X_test = X_test.reindex(columns=model_train.params.index, fill_value=0)

predictions = model_train.predict(X_test)
actuals = test['revenue'].values

# Calculate MAPE
mape = np.mean(np.abs((actuals - predictions) / actuals)) * 100
print(f"Test MAPE: {mape:.2f}%")
```

### Pattern 3: Channel Contribution Decomposition

Break down revenue into channel contributions + baseline:

```python
def decompose_revenue(df, model, transformed_channels):
    """Calculate contribution of each channel + baseline."""
    contributions = {}
    
    # Baseline (intercept)
    contributions['baseline'] = model.params['const']
    
    # Channel contributions
    for channel in transformed_channels:
        coef = model.params[channel]
        contributions[channel] = (df[channel] * coef).sum()
    
    # Controls
    control_vars = [c for c in model.params.index 
                   if c not in transformed_channels + ['const']]
    contributions['controls'] = sum(
        (df[c] * model.params[c]).sum() if c in df.columns else 0
        for c in control_vars
    )
    
    return contributions

contributions = decompose_revenue(df, model, transformed_channels)
contrib_df = pd.DataFrame.from_dict(contributions, orient='index', columns=['contribution'])
contrib_df['pct'] = contrib_df['contribution'] / contrib_df['contribution'].sum() * 100
print(contrib_df.sort_values('contribution', ascending=False))
```

## Troubleshooting

### Issue: Low R²

**Solution:** Check for:
- Missing transformations (ensure adstock → saturation order)
- Incorrect parameter values (theta too low, alpha misaligned with spend scale)
- Missing control variables

```python
# Diagnose with correlation matrix
import seaborn as sns
import matplotlib.pyplot as plt

corr = df[transformed_channels + ['revenue']].corr()
sns.heatmap(corr, annot=True, cmap='coolwarm')
plt.show()
```

### Issue: Negative Coefficients

**Solution:** Indicates potential multicollinearity or wrong transformation direction.

```python
from statsmodels.stats.outliers_influence import variance_inflation_factor

# Calculate VIF
X = df[transformed_channels]
vif_data = pd.DataFrame({
    'feature': X.columns,
    'VIF': [variance_inflation_factor(X.values, i) for i in range(len(X.columns))]
})
print(vif_data.sort_values('VIF', ascending=False))
# VIF > 10 indicates high multicollinearity
```

### Issue: Optimizer Returns Equal Budget

**Solution:** Saturation parameters may be too aggressive or budget constraint too loose.

```python
# Test saturation curve visually
x_range = np.linspace(0, df['tv_S'].max(), 100)
y_saturated = hill_saturation(x_range, alpha=150, gamma=2.0)

plt.plot(x_range, y_saturated)
plt.xlabel('Spend')
plt.ylabel('Saturated Response')
plt.title('TV Saturation Curve')
plt.show()
# Should show diminishing returns, not flat line
```

### Issue: "events" Column Encoding Errors

**Solution:** Clean literal "na" strings before one-hot encoding:

```python
# Before modeling
df['events'] = df['events'].replace('na', np.nan)

# Or drop if too many missing
if df['events'].isna().sum() / len(df) > 0.5:
    controls.remove('events')
```

## Configuration

Key hyperparameters to tune:

```python
config = {
    'adstock': {
        'theta': 0.5,      # 0.3-0.7 typical; higher = longer carryover
        'L_max': 8         # Max lag periods (weeks)
    },
    'saturation': {
        'alpha': 100,      # Half-saturation point (match spend scale)
        'gamma': 2.0       # 1.5-2.5 typical; higher = steeper S-curve
    },
    'optimization': {
        'method': 'SLSQP',  # Sequential Least Squares Programming
        'bounds_multiplier': 2.0  # Max individual channel = 2x current avg
    }
}
```

## Export for Power BI

```python
# Export model outputs for dashboard
df[['DATE', 'revenue'] + transformed_channels].to_csv(
    'data/powerbi/model_predictions.csv', index=False
)

coef_df.to_csv('data/powerbi/coefficients.csv', index=False)
comparison.to_csv('data/powerbi/optimized_budget.csv', index=False)

# Export decomposition
contrib_df.to_csv('data/powerbi/contribution_breakdown.csv')
```

This pipeline provides a complete, reproducible MMM workflow from raw spend data to optimized budget allocation, achieving R² = 0.91 and identifying a projected +24.4% revenue uplift.
