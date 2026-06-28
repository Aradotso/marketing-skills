---
name: marketing-science-academic-writing
description: AI agent skill for writing marketing science papers from topic selection through consumer utility modeling, identification design, structural estimation, counterfactual simulations, to complete first draft
triggers:
  - "write a marketing science paper"
  - "help me with structural demand estimation"
  - "design a conjoint analysis study"
  - "create a BLP model for my paper"
  - "position my research for Marketing Science journal"
  - "design identification strategy for causal marketing research"
  - "write consumer utility model specification"
  - "create counterfactual simulations for merger analysis"
---

# Marketing Science Academic Writing Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive skill for AI agents to guide researchers through writing marketing science papers for top journals (Marketing Science, JMR, JM, JCR, JAMS, IJRM, QME, Marketing Letters). Covers the complete workflow from topic selection through consumer utility modeling, identification design, structural estimation, counterfactual simulations, to a full manuscript draft.

## What This Skill Enables

This skill equips AI coding agents to help researchers with:

- **Topic Positioning**: Identify contribution gaps, select target journals, formulate positioning statements
- **Consumer Utility Modeling**: Design micro-founded demand models (logit, nested logit, random coefficients)
- **Identification Strategy**: Design causal inference approaches (DID, RDD, IV) and structural identification
- **Structural Estimation**: Implement BLP, GMM, MLE, Bayesian methods with proper specification
- **Counterfactual Simulations**: Design and execute policy simulations, merger analysis, welfare calculations
- **Conjoint Analysis**: Design choice experiments, estimate WTP, simulate market scenarios
- **Field Experiments**: Design randomized trials, power analysis, treatment effect estimation
- **Full Manuscript Assembly**: Generate complete LaTeX drafts with INFORMS formatting

## Installation

### For OpenCode

```bash
# Clone the repository
git clone https://github.com/liyuanbo1024/marketing-science-writing.git

# Linux/macOS - copy to OpenCode skills folder
cp -r marketing-science-writing ~/.config/opencode/skills/

# Windows PowerShell
Copy-Item -Recurse marketing-science-writing "$env:USERPROFILE\.config\opencode\skills\"
```

Or register a custom path in `~/.config/opencode/opencode.json`:

```json
{
  "skills": {
    "paths": [
      "~/.config/opencode/skills",
      "/path/to/marketing-science-writing"
    ]
  }
}
```

### For Claude Code

```bash
# Linux/macOS
cp -r marketing-science-writing ~/.claude/skills/

# Windows PowerShell
Copy-Item -Recurse marketing-science-writing "$env:USERPROFILE\.claude\skills\"
```

### For Cursor / Windsurf

```bash
# Cursor
cp -r marketing-science-writing ~/.cursor/skills/

# Windsurf
cp -r marketing-science-writing ~/.windsurf/skills/
```

### For Other AI Agents

Point your agent's skills directory to the cloned repository, or reference `SKILL.md` directly in your agent configuration.

## Core Components

### Directory Structure

```
marketing-science-writing/
├── SKILL.md                           # Main skill orchestration logic
├── references/
│   ├── journal-characteristics.md     # 8 journal profiles and expectations
│   ├── modeling-conventions.md        # Utility model specifications
│   ├── identification-guide.md        # DID, RDD, IV strategies
│   ├── estimation-guide.md            # BLP, GMM, MLE implementation
│   ├── counterfactual-guide.md        # Simulation design
│   ├── conjoint-analysis.md           # Conjoint experiment design
│   ├── field-experiments.md           # RCT design and analysis
│   ├── marketing-implications.md      # Managerial insights framework
│   ├── reviewer-expectations.md       # Journal-specific review criteria
│   └── writing-patterns.md            # Reusable writing templates
├── assets/
│   └── demand-model-reference.md      # Generic demand model structures
└── examples/
    ├── manuscript_template.tex        # INFORMS LaTeX template
    └── blp_estimation_example.py      # BLP estimation code
```

## Usage Patterns

### Pattern 1: Full Pipeline (0-to-Draft)

Guide the agent through all five stages:

```
User: "Take me through the full marketing science pipeline. My topic is 
demand estimation for electric vehicles with consumer heterogeneity."

Agent: [Loads skill, executes Stage 1]
- Conducts literature search for EV demand papers
- Builds gap table comparing your approach to existing work
- Recommends Marketing Science (structural model focus) or QME (IO focus)
- Formulates 3-5 contribution statements
- Confirms with user before proceeding to Stage 2
```

### Pattern 2: Stage-Specific Entry

Jump directly to a specific stage:

**Stage 1: Topic Positioning**
```
"I have a research idea about social media influencer effects on brand equity. 
Help me position this for a top journal."
```

**Stage 2: Consumer Utility Modeling**
```
"Design a consumer utility model for a differentiated products market 
with price, quality, and brand effects. Include random coefficients."
```

**Stage 3: Identification & Estimation**
```
"I have panel data on product choices. Design an identification strategy 
using instrumental variables for endogenous price."
```

**Stage 4: Counterfactuals**
```
"My BLP model is estimated. Design counterfactual simulations for 
a merger between the top two firms."
```

**Stage 5: Writing & Assembly**
```
"Write the complete paper draft with Introduction, Model, Identification, 
Results, and Discussion sections."
```

### Pattern 3: Reference-Only Mode

Use as a knowledge base:

```
"What are Marketing Science's reviewer expectations for identification?"
"How should I structure utility specification for a nested logit model?"
"What micro moments should I use in BLP estimation?"
"How do I design a conjoint analysis for luxury goods?"
```

## Key Methodologies Covered

### 1. Structural Demand Estimation (BLP)

**Consumer Utility Specification:**

```python
# Random coefficients logit utility
# U_ijt = δ_jt + μ_ijt + ε_ijt
# δ_jt = X_jt * β - α * p_jt + ξ_jt  (mean utility)
# μ_ijt = (X_jt, p_jt) * Σ * ν_i     (random coefficients)
# ε_ijt ~ Type I Extreme Value

import numpy as np
from scipy.optimize import minimize

def compute_market_shares(delta, mu, market_ids):
    """
    Compute predicted market shares from mean utilities (delta) 
    and individual-level deviations (mu).
    
    Args:
        delta: Mean utility vector (J products)
        mu: Individual utility deviations (J x N matrix)
        market_ids: Market identifier for each product
    
    Returns:
        s_jt: Predicted market shares
    """
    # Compute individual choice probabilities
    # exp(δ_j + μ_ij) / (1 + Σ_k exp(δ_k + μ_ik))
    exp_utils = np.exp(delta[:, None] + mu)  # J x N
    
    # Add outside option (utility = 0)
    choice_probs = exp_utils / (1 + exp_utils.sum(axis=0))  # J x N
    
    # Aggregate to market shares (average over consumers)
    market_shares = choice_probs.mean(axis=1)
    
    return market_shares

def blp_inner_loop(theta2, X, p, s_obs, market_ids, nu_draws):
    """
    BLP contraction mapping to recover mean utilities (delta).
    
    Args:
        theta2: Nonlinear parameters (Σ in random coefficients)
        X: Product characteristics (J x K)
        p: Prices (J x 1)
        s_obs: Observed market shares (J x 1)
        market_ids: Market identifiers
        nu_draws: Random draws for simulation (K x N)
    
    Returns:
        delta: Mean utilities
    """
    J = len(s_obs)
    N = nu_draws.shape[1]
    
    # Compute μ_ijt = (X, p) * Σ * ν_i
    mu = np.dot(np.column_stack([X, p]), theta2.reshape(-1, 1)) * nu_draws.T  # J x N
    
    # Initialize delta
    delta = np.log(s_obs) - np.log(1 - s_obs.sum())
    
    # Contraction mapping
    max_iter = 1000
    tol = 1e-12
    for iteration in range(max_iter):
        s_pred = compute_market_shares(delta, mu, market_ids)
        delta_new = delta + np.log(s_obs) - np.log(s_pred)
        
        if np.max(np.abs(delta_new - delta)) < tol:
            break
        delta = delta_new
    
    return delta

def blp_gmm_objective(theta2, X, p, s_obs, Z, W, market_ids, nu_draws):
    """
    BLP GMM objective function.
    
    Args:
        theta2: Nonlinear parameters (random coefficients)
        X: Product characteristics
        p: Prices
        s_obs: Observed market shares
        Z: Instruments (J x L)
        W: Weighting matrix (L x L)
        market_ids: Market identifiers
        nu_draws: Random draws for simulation
    
    Returns:
        objective: GMM objective value
    """
    # Recover mean utilities via contraction
    delta = blp_inner_loop(theta2, X, p, s_obs, market_ids, nu_draws)
    
    # Compute moments E[Z' ξ] where ξ = δ - X*β + α*p
    # For simplicity, regress delta on X and p to get residuals
    Xp = np.column_stack([X, p])
    beta = np.linalg.lstsq(Xp, delta, rcond=None)[0]
    xi = delta - np.dot(Xp, beta)
    
    # Moment conditions
    moments = np.dot(Z.T, xi) / len(xi)  # L x 1
    
    # GMM objective
    objective = np.dot(moments.T, np.dot(W, moments))
    
    return objective

# Example usage
np.random.seed(42)
J = 100  # Number of products
N = 500  # Number of consumer draws
K = 3    # Number of characteristics

# Simulate data
X = np.random.randn(J, K)
p = 5 + 2 * X[:, 0] + np.random.randn(J)
s_obs = np.random.dirichlet(np.ones(J))
Z = np.column_stack([X, np.random.randn(J, 2)])  # Cost shifters as instruments
market_ids = np.zeros(J, dtype=int)  # Single market
nu_draws = np.random.randn(K + 1, N)  # +1 for price coefficient

# Initial guess for theta2 (standard deviations of random coefficients)
theta2_init = np.ones(K + 1) * 0.5

# Optimal weighting matrix (identity for two-step GMM)
W = np.eye(Z.shape[1])

# Optimize
result = minimize(
    blp_gmm_objective,
    theta2_init,
    args=(X, p, s_obs, Z, W, market_ids, nu_draws),
    method='Nelder-Mead',
    options={'maxiter': 100}
)

print(f"Estimated theta2 (random coefficient std devs): {result.x}")
```

**Key Implementation Steps:**

1. **Specify utility function**: Linear in parameters with random coefficients
2. **Generate simulation draws**: Halton or pseudo-random draws for consumer heterogeneity
3. **Inner loop (contraction)**: Invert from observed shares to mean utilities
4. **Outer loop (GMM)**: Optimize over nonlinear parameters using moment conditions
5. **Standard errors**: Bootstrap or asymptotic formula with optimal weighting matrix

### 2. Causal Identification: Difference-in-Differences

**DID with Staggered Treatment:**

```python
import pandas as pd
import numpy as np
from linearmodels import PanelOLS

def prepare_did_data(df, unit_col, time_col, outcome_col, treatment_col):
    """
    Prepare panel data for DID estimation.
    
    Args:
        df: DataFrame with panel structure
        unit_col: Column name for cross-sectional units
        time_col: Column name for time periods
        outcome_col: Column name for outcome variable
        treatment_col: Column name for treatment indicator (0/1)
    
    Returns:
        Prepared DataFrame with multi-index (unit, time)
    """
    df_clean = df[[unit_col, time_col, outcome_col, treatment_col]].copy()
    df_clean = df_clean.set_index([unit_col, time_col])
    return df_clean

def estimate_twfe_did(df, outcome_col, treatment_col, unit_col, time_col, 
                      controls=None, cluster_unit=True):
    """
    Two-Way Fixed Effects DID estimation.
    
    Specification: Y_it = α_i + λ_t + β * Treat_it + X_it * γ + ε_it
    
    Args:
        df: Panel DataFrame (must be indexed by unit and time)
        outcome_col: Outcome variable name
        treatment_col: Treatment indicator name
        unit_col: Unit identifier for clustering
        time_col: Time identifier for clustering
        controls: List of control variable names
        cluster_unit: Whether to cluster standard errors by unit
    
    Returns:
        Regression results
    """
    # Prepare formula
    controls_str = " + ".join(controls) if controls else ""
    formula = f"{outcome_col} ~ {treatment_col}"
    if controls_str:
        formula += f" + {controls_str}"
    formula += " + EntityEffects + TimeEffects"
    
    # Estimate with two-way fixed effects
    model = PanelOLS.from_formula(
        formula,
        data=df,
        drop_absorbed=True
    )
    
    # Cluster standard errors
    if cluster_unit:
        results = model.fit(cov_type='clustered', cluster_entity=True)
    else:
        results = model.fit(cov_type='robust')
    
    return results

# Example: Social media campaign DID
np.random.seed(42)

# Simulate panel data
n_stores = 200
n_periods = 24
treatment_start = 12  # Half the stores treated at period 12

data = []
for store in range(n_stores):
    treated = store < n_stores // 2
    treatment_time = treatment_start if treated else None
    
    for t in range(n_periods):
        # Baseline sales with store and time fixed effects
        sales = 100 + store * 0.5 + t * 2 + np.random.randn() * 10
        
        # Treatment effect (15% increase after treatment)
        treat_indicator = 1 if (treated and t >= treatment_start) else 0
        sales += treat_indicator * 15
        
        data.append({
            'store_id': store,
            'month': t,
            'sales': sales,
            'treated': treat_indicator,
            'store_size': np.random.choice(['small', 'medium', 'large'])
        })

df = pd.DataFrame(data)

# One-hot encode store size
df = pd.get_dummies(df, columns=['store_size'], drop_first=True)

# Prepare for panel regression
df_panel = df.set_index(['store_id', 'month'])

# Estimate DID
results = estimate_twfe_did(
    df_panel,
    outcome_col='sales',
    treatment_col='treated',
    unit_col='store_id',
    time_col='month',
    controls=['store_size_medium', 'store_size_large'],
    cluster_unit=True
)

print(results.summary)
print(f"\nEstimated Treatment Effect: {results.params['treated']:.2f}")
print(f"95% CI: [{results.conf_int().loc['treated', 'lower']:.2f}, "
      f"{results.conf_int().loc['treated', 'upper']:.2f}]")
```

**DID Design Checklist (per identification-guide.md):**

- [ ] Parallel trends assumption: Plot pre-treatment trends for treated vs. control
- [ ] No anticipation: Treatment effect should be zero in pre-periods
- [ ] No spillovers: Control units should not be affected by treatment
- [ ] Event study specification: Include leads and lags of treatment
- [ ] Robust standard errors: Cluster by unit (or unit and time)
- [ ] Sensitivity: Test with alternative control groups, placebo tests

### 3. Conjoint Analysis Design

**Efficient Conjoint Design:**

```python
import numpy as np
import pandas as pd
from itertools import product
from sklearn.linear_model import LogisticRegression

def generate_orthogonal_design(attributes, n_profiles=16):
    """
    Generate a near-orthogonal fractional factorial design for conjoint.
    
    Args:
        attributes: Dict mapping attribute names to lists of levels
                   e.g., {'price': [10, 20, 30], 'quality': ['low', 'high']}
        n_profiles: Number of profiles to generate
    
    Returns:
        DataFrame with conjoint profiles
    """
    # Full factorial
    attr_names = list(attributes.keys())
    levels = [attributes[attr] for attr in attr_names]
    full_factorial = list(product(*levels))
    
    # Sample n_profiles (ideally use D-optimal design software for real studies)
    if len(full_factorial) > n_profiles:
        selected_indices = np.random.choice(len(full_factorial), n_profiles, replace=False)
        selected = [full_factorial[i] for i in selected_indices]
    else:
        selected = full_factorial
    
    # Create DataFrame
    df = pd.DataFrame(selected, columns=attr_names)
    return df

def create_choice_tasks(profiles, n_tasks=10, n_alternatives=3):
    """
    Create choice tasks by randomly sampling alternatives per task.
    
    Args:
        profiles: DataFrame of conjoint profiles
        n_tasks: Number of choice tasks
        n_alternatives: Number of alternatives per task (excluding no-choice)
    
    Returns:
        List of choice tasks (each task is a DataFrame)
    """
    tasks = []
    for task_id in range(n_tasks):
        # Sample alternatives
        alternatives = profiles.sample(n_alternatives, replace=False).copy()
        alternatives['task_id'] = task_id
        alternatives['alt_id'] = range(n_alternatives)
        tasks.append(alternatives)
    
    return tasks

def estimate_partworths(choice_data, attributes):
    """
    Estimate part-worth utilities from conjoint choice data using logit.
    
    Args:
        choice_data: DataFrame with columns [task_id, alt_id, chosen, ...attributes]
        attributes: List of attribute column names
    
    Returns:
        Dict of part-worth utilities
    """
    # Prepare design matrix (effects coding)
    X = pd.get_dummies(choice_data[attributes], drop_first=True)
    y = choice_data['chosen']
    
    # Logistic regression
    model = LogisticRegression(penalty=None, max_iter=1000)
    model.fit(X, y)
    
    # Extract part-worths
    partworths = dict(zip(X.columns, model.coef_[0]))
    
    return partworths, model

# Example: Smartphone conjoint
attributes = {
    'brand': ['Apple', 'Samsung', 'Google'],
    'price': [699, 899, 1099],
    'storage': ['128GB', '256GB', '512GB'],
    'camera': ['12MP', '48MP', '108MP']
}

# Generate profiles
profiles = generate_orthogonal_design(attributes, n_profiles=24)
print("Sample profiles:")
print(profiles.head())

# Create choice tasks
tasks = create_choice_tasks(profiles, n_tasks=15, n_alternatives=3)

# Simulate responses (for demonstration)
np.random.seed(42)
choice_data = []
for task in tasks:
    task['utility'] = (
        (task['brand'] == 'Apple').astype(int) * 0.5 +
        -task['price'] / 1000 +
        (task['storage'] == '512GB').astype(int) * 0.3 +
        (task['camera'] == '108MP').astype(int) * 0.4 +
        np.random.gumbel(size=len(task))  # Logit error
    )
    chosen_alt = task['utility'].idxmax()
    task['chosen'] = 0
    task.loc[chosen_alt, 'chosen'] = 1
    choice_data.append(task[['task_id', 'alt_id', 'brand', 'price', 'storage', 'camera', 'chosen']])

choice_df = pd.concat(choice_data, ignore_index=True)

# Estimate part-worths
partworths, model = estimate_partworths(choice_df, ['brand', 'price', 'storage', 'camera'])
print("\nEstimated Part-Worths:")
for attr, value in partworths.items():
    print(f"  {attr}: {value:.3f}")

# Compute willingness-to-pay
# WTP for attribute = β_attribute / |β_price|
price_coef = partworths['price'] if 'price' in partworths else -0.001
wtp = {attr: val / abs(price_coef) for attr, val in partworths.items() if attr != 'price'}
print("\nWillingness-to-Pay (relative to price coefficient):")
for attr, val in wtp.items():
    print(f"  {attr}: ${val:.0f}")
```

**Conjoint Best Practices (per conjoint-analysis.md):**

- Use D-optimal or orthogonal designs to minimize multicollinearity
- Include 12-20 choice tasks per respondent
- Add a "no-choice" option to capture opt-out behavior
- Validate with holdout tasks (20% of tasks)
- Estimate hierarchical Bayes models for individual-level part-worths
- Report WTP with confidence intervals (bootstrap or delta method)

### 4. Counterfactual Simulations

**Merger Simulation:**

```python
import numpy as np
from scipy.optimize import fsolve

def compute_equilibrium_prices(mc, delta_pre, theta, ownership_matrix):
    """
    Compute Nash-Bertrand equilibrium prices given marginal costs.
    
    Args:
        mc: Marginal costs (J x 1)
        delta_pre: Pre-merger mean utilities (J x 1)
        theta: Demand parameters (price coefficient, etc.)
        ownership_matrix: J x J matrix (1 if same owner, 0 otherwise)
    
    Returns:
        Equilibrium prices
    """
    J = len(mc)
    alpha = theta['price_coef']  # Price sensitivity
    
    def foc(prices):
        """First-order conditions for multi-product firm pricing."""
        # Compute market shares and derivatives
        utils = delta_pre - alpha * prices
        exp_utils = np.exp(utils)
        shares = exp_utils / (1 + exp_utils.sum())
        
        # Price derivatives of shares (logit)
        share_derivs = np.outer(shares, shares) * alpha
        np.fill_diagonal(share_derivs, -alpha * shares * (1 - shares))
        
        # FOC: s_j + Σ_k Ω_jk * (p_k - mc_k) * ∂s_k/∂p_j = 0
        markup_matrix = ownership_matrix * share_derivs
        foc_values = shares + markup_matrix @ (prices - mc)
        
        return foc_values
    
    # Solve for equilibrium prices
    p_init = mc + 10  # Initial guess: cost plus markup
    prices_eq = fsolve(foc, p_init)
    
    return prices_eq

def simulate_merger(mc, delta_pre, theta, firm_ids, merging_firms):
    """
    Simulate a merger between two firms.
    
    Args:
        mc: Marginal costs
        delta_pre: Pre-merger mean utilities
        theta: Demand parameters
        firm_ids: Array of firm IDs for each product
        merging_firms: Tuple of firm IDs that merge (e.g., (1, 2))
    
    Returns:
        Dict with pre- and post-merger prices, shares, consumer surplus
    """
    J = len(mc)
    alpha = theta['price_coef']
    
    # Pre-merger ownership matrix
    ownership_pre = np.array([[1 if firm_ids[i] == firm_ids[j] else 0 
                                for j in range(J)] for i in range(J)])
    
    # Post-merger ownership matrix (merge firms)
    firm_ids_post = firm_ids.copy()
    firm_ids_post[firm_ids == merging_firms[1]] = merging_firms[0]
    ownership_post = np.array([[1 if firm_ids_post[i] == firm_ids_post[j] else 0 
                                 for j in range(J)] for i in range(J)])
    
    # Compute equilibrium prices
    prices_pre = compute_equilibrium_prices(mc, delta_pre, theta, ownership_pre)
    prices_post = compute_equilibrium_prices(mc, delta_pre, theta, ownership_post)
    
    # Compute market shares
    utils_pre = delta_pre - alpha * prices_pre
    shares_pre = np.exp(utils_pre) / (1 + np.exp(utils_pre).sum())
    
    utils_post = delta_pre - alpha * prices_post
    shares_post = np.exp(utils_post) / (1 + np.exp(utils_post).sum())
    
    # Consumer surplus (logit inclusive value / price coefficient)
    cs_pre = np.log(1 + np.exp(utils_pre).sum()) / abs(alpha)
    cs_post = np.log(1 + np.exp(utils_post).sum()) / abs(alpha)
    
    # Producer profit
    profit_pre = ((prices_pre - mc) * shares_pre).sum()
    profit_post = ((prices_post - mc) * shares_post).sum()
    
    return {
        'prices_pre': prices_pre,
        'prices_post': prices_post,
        'shares_pre': shares_pre,
        'shares_post': shares_post,
        'consumer_surplus_pre': cs_pre,
        'consumer_surplus_post': cs_post,
        'profit_pre': profit_pre,
        'profit_post': profit_post,
        'price_change_pct': (prices_post - prices_pre) / prices_pre * 100
    }

# Example: Simulate merger in smartphone market
np.random.seed(42)
J = 10
mc = np.random.uniform(300, 500, J)  # Marginal costs
delta_pre = np.random.randn(J) * 0.5  # Mean utilities (price-independent component)
theta = {'price_coef': -0.003}  # Price coefficient (negative)

# Three firms, each with multiple products
firm_ids = np.array([1, 1, 1, 2, 2, 2, 3, 3, 3, 3])

# Simulate merger between firm 1 and firm 2
merger_results = simulate_merger(mc, delta_pre, theta, firm_ids, merging_firms=(1, 2))

print("Merger Simulation Results:")
print(f"  Pre-merger avg price: ${merger_results['prices_pre'].mean():.2f}")
print(f"  Post-merger avg price: ${merger_results['prices_post'].mean():.2f}")
print(f"  Average price increase: {merger_results['price_change_pct'].mean():.2f}%")
print(f"  Consumer surplus change: ${merger_results['consumer_surplus_post'] - merger_results['consumer_surplus_pre']:.2f}")
print(f"  Producer profit change: ${merger_results['profit_post'] - merger_results['profit_pre']:.2f}")
```

**Counterfactual Reporting Guidelines:**

- Present baseline (status quo) vs. counterfactual side-by-side
- Report welfare decomposition: consumer surplus, producer profit, total welfare
- Sensitivity analysis: Vary demand elasticity, cost assumptions
- Connect to marketing implications: "A 10% price increase would reduce market penetration by X% but increase category profit by Y%"

## Journal Selection Quick Reference

| Journal | When to Choose | Red Flags |
|---------|---------------|-----------|
| **Marketing Science** | Structural models, analytical rigor, BLP/GMM/MLE | Weak identification, descriptive-only, no theory |
| **JMR** | Causal inference (DID/RDD/IV), experiments, rich data | Pure theory, no empirical validation, small samples |
| **JM** | Broad substantive contribution, strategy, B2B | Methodological novelty only, narrow application |
| **JCR** | Consumer psychology, qualitative + quant, rich constructs | Firm-level only, no consumer-level data or theory |
| **JAMS** | Conceptual + empirical, broader audience, meta-analysis | Highly technical with limited substantive insight |
| **IJRM** | European context, diverse methods, international data | US-only context, single-method papers |
| **QME** | IO approach, structural econometrics, theoretical depth | Weak economic foundations, atheoretical |
| **Marketing Letters** | Short, sharp, focused contribution (< 6000 words) | Incremental extensions, lacks novelty |

## Five-Stage Workflow

### Stage 1: Topic Positioning & Journal Selection

**Agent Actions:**

1. **Understand the Research Question**
   - Ask user to describe the substantive marketing problem
   - Identify: What is the decision-maker's problem? What is the outcome of interest?

2. **Literature Gap Analysis**
   - Search for 8-12 closest papers
   - Build a Gap Table (columns: Paper, Data, Method, Finding, Gap)
   - Identify what is missing: Data source? Method? Substantive context? Theoretical mechanism?

3. **Match to Journal Identity**
   - Use `references/journal-characteristics.md` to match paper profile to journal
   - Key dimensions: Method sophistication, substantive vs. methodological, audience breadth
   - Recommendation: Target journal + backup journal

4. **Formulate Contribution Statement**
   - Write 3-5 numbered contributions (one substantive, one methodological, one data/context)
   - Use language from `references/writing-patterns.md` (e.g., "We are the first to...", "We show that...", "We contribute by...")

5. **Section Plan**
   
