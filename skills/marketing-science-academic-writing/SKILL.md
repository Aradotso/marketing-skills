---
name: marketing-science-academic-writing
description: AI agent skill for writing academic marketing science papers from topic selection through structural modeling, identification, estimation, and full draft assembly
triggers:
  - "write a marketing science paper"
  - "help me write for Marketing Science journal"
  - "design a consumer utility model"
  - "set up BLP demand estimation"
  - "design identification strategy for marketing research"
  - "create counterfactual simulations"
  - "write academic marketing paper"
  - "structural estimation for marketing"
---

# Marketing Science Academic Writing

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive skill for writing quantitative marketing science papers targeting flagship journals (Marketing Science, JMR, JM, JCR, JAMS, IJRM, QME, Marketing Letters). Guides you through the complete 0-to-draft pipeline: topic positioning, consumer utility modeling, identification design, structural estimation, counterfactual simulations, and manuscript assembly.

## What This Skill Does

This skill transforms your research idea into a complete academic paper draft by:

1. **Topic Positioning**: Gap analysis, journal selection, contribution framing
2. **Consumer Utility Modeling**: Micro-founded demand systems, notation, utility specification
3. **Identification & Estimation**: Causal inference strategies (DID/RDD/IV), structural estimation (BLP/GMM)
4. **Counterfactuals & Experiments**: Simulation design, conjoint analysis, field experiments
5. **Full Draft Assembly**: Complete LaTeX manuscript with INFORMS formatting

**Key differentiator**: Domain-specific conventions for marketing journals that generic writing skills don't cover — utility model structure, identification justification, structural estimation workflows, and journal-specific reviewer expectations.

## Installation

The project is a skill repository that you reference, not a package you install. The skill files provide structured guidance and reference materials.

### For AI Coding Agents

**OpenCode**:
```bash
cp -r marketing-science-writing ~/.config/opencode/skills/
```

**Claude Code**:
```bash
cp -r marketing-science-writing ~/.claude/skills/
```

**Codex**:
```bash
cp -r marketing-science-writing ~/.agents/skills/
```

**Cursor / Windsurf**:
```bash
cp -r marketing-science-writing ~/.cursor/skills/
# or
cp -r marketing-science-writing ~/.windsurf/skills/
```

### Manual Integration

Reference `SKILL.md` and the `references/` directory in your agent's custom skills path, or concatenate the markdown files into your system prompt.

## Project Structure

```
marketing-science-writing/
├── SKILL.md                         # Main skill orchestration
├── references/
│   ├── journal-characteristics.md   # 8 journals: profiles & expectations
│   ├── modeling-conventions.md      # Utility models, demand systems
│   ├── identification-guide.md      # DID, RDD, IV strategies
│   ├── estimation-guide.md          # BLP, GMM, MLE, Bayesian
│   ├── counterfactual-guide.md      # Simulation design
│   ├── conjoint-analysis.md         # Conjoint experiment design
│   ├── field-experiments.md         # Field experiment protocols
│   ├── marketing-implications.md    # Actionable implications framework
│   ├── reviewer-expectations.md     # Reviewer checklist
│   └── writing-patterns.md          # Reusable writing templates
├── assets/
│   └── demand-model-reference.md    # Generic demand model structures
├── examples/
│   ├── manuscript_template.tex      # INFORMS LaTeX template
│   └── blp_estimation_example.py    # BLP estimation starter code
└── README.md
```

## Core Usage Patterns

### Pattern 1: Full 0-to-Draft Pipeline

**Trigger**: "Take me through the full marketing science pipeline for a paper on [topic]"

**Workflow**:
1. **Stage 1**: Topic positioning → gap table, journal selection, contribution list
2. **Stage 2**: Consumer utility model → notation, utility specification, demand derivation
3. **Stage 3**: Identification & estimation → DID/RDD/IV design, estimator choice
4. **Stage 4**: Counterfactuals & experiments → simulation design, experiment protocol
5. **Stage 5**: Full draft → LaTeX manuscript assembly

**Example**:
```
User: "Write a Marketing Science paper on BLP demand estimation 
      for the smartphone market with merger counterfactuals."

Agent: [Executes Stage 1]
       → Gap table: 8 closest papers (Nair 2007, Berry et al. 1995, etc.)
       → Target journal: Marketing Science (structural + IO focus)
       → Contributions: (1) First BLP on smartphone market, 
                       (2) New demand-side micro moments using clickstream data,
                       (3) Merger simulation with regulatory implications

       [Stage 2]
       → Utility specification with random coefficients on price, screen size, brand
       → Nested logit within operating system
       → Supply-side: Bertrand-Nash pricing game

       [Stage 3]
       → Identification: Cost shifters as IVs (component prices from Counterpoint)
       → Micro moments: Match cross-price elasticities from clickstream
       → GMM estimation with optimal weighting matrix

       [Stage 4]
       → Counterfactual: Simulate Apple-Samsung merger
       → Equilibrium: New Nash equilibrium prices post-merger
       → Welfare: Consumer surplus loss, producer profit gain

       [Stage 5]
       → Complete LaTeX manuscript with INFORMS template
```

### Pattern 2: Stage-Specific Deep Dive

**Trigger**: "Help me design the consumer utility model for my paper"

**Focus**: Stage 2 only — utility specification, notation system, demand derivation.

**Example**:
```
User: "I'm studying ride-sharing demand. Help me specify the utility model."

Agent: 
# Consumer Utility Model

## Notation
- $i$: consumer index
- $j$: ride option (Uber, Lyft, taxi, outside option)
- $t$: time period
- $p_{jt}$: price per mile
- $x_{jt}$: observable characteristics (wait time, vehicle quality)
- $\xi_{jt}$: unobserved quality shock

## Indirect Utility
Consumer $i$ obtains utility from option $j$ at time $t$:

$$u_{ijt} = \alpha_i p_{jt} + x_{jt}'\beta_i + \xi_{jt} + \epsilon_{ijt}$$

where:
- $\alpha_i \sim N(\alpha, \sigma_\alpha^2)$: random price coefficient (negative)
- $\beta_i \sim N(\beta, \Sigma_\beta)$: random coefficients on characteristics
- $\epsilon_{ijt}$: i.i.d. Type-I extreme value

## Choice Probability (Random Coefficients Logit)
The probability consumer $i$ chooses option $j$:

$$s_{ij}(\theta) = \int \frac{\exp(u_{ijt})}{\sum_{k=0}^J \exp(u_{ikt})} dF(\alpha_i, \beta_i)$$

Aggregate market share: $s_j = \int s_{ij}(\theta) dF(i)$

## Demand Inversion
For estimation, invert share equation:
$$\delta_j = \ln(s_j) - \ln(s_0)$$

where $\delta_j = \alpha p_j + x_j'\beta + \xi_j$ (mean utility).
```

### Pattern 3: Identification Strategy Design

**Trigger**: "I need to design the identification strategy for my marketing paper"

**Focus**: Stage 3 — causal inference or structural identification.

**Example for DID**:
```
User: "Amazon introduced same-day delivery in select cities. 
      I want to estimate the causal effect on sales."

Agent:
# Identification Strategy: Difference-in-Differences

## Treatment & Control
- **Treatment**: Cities with same-day delivery (rollout: 2014-2016, staggered)
- **Control**: Cities without same-day delivery (matched on demographics)

## Specification
$$\log(sales_{ct}) = \beta_0 + \beta_1 SameDay_{ct} + \gamma_c + \lambda_t + \epsilon_{ct}$$

where:
- $SameDay_{ct}$: indicator for city $c$ having same-day delivery at time $t$
- $\gamma_c$: city fixed effects
- $\lambda_t$: time fixed effects

## Parallel Trends Test
Event study specification:
$$\log(sales_{ct}) = \sum_{k=-K}^{-2} \delta_k D_{ct}^k + \sum_{k=0}^{K} \delta_k D_{ct}^k + \gamma_c + \lambda_t + \epsilon_{ct}$$

where $D_{ct}^k = 1$ if city $c$ is $k$ periods from treatment.

**Test**: $H_0: \delta_{-K} = \cdots = \delta_{-2} = 0$ (pre-trends)

## Robustness Checks
1. Callaway-Sant'Anna estimator (staggered treatment)
2. Synthetic control for treated cities
3. Triple-differences: same-day × product category × time
```

**Example for IV (Structural)**:
```
User: "Price is endogenous in my demand model. How do I construct instruments?"

Agent:
# Instrumental Variables for Demand Estimation

## Endogeneity Problem
Price $p_j$ is correlated with unobserved quality $\xi_j$ due to:
1. Simultaneous pricing (supply-side response to demand shocks)
2. Omitted product characteristics

## IV Strategy: BLP Instruments

### 1. Cost Shifters
**Instruments**: Input prices, wage indices, exchange rates
**Exclusion restriction**: Affect marginal cost → price, but not consumer utility directly
**Example**: Steel prices for automotive demand estimation

### 2. Hausman Instruments
**Instruments**: Prices of same product in other markets
**Assumption**: Market-specific demand shocks are uncorrelated
**Example**: Price of iPhone 13 in New York as IV for Boston demand

### 3. Differentiation Instruments
**Instruments**: Number of competing products, sum of rival characteristics
**Intuition**: More competition → lower markups → lower prices
**Construction**:
```python
# Sum of rival characteristics
df['IV_rival_features'] = df.groupby('market')['feature_score'].transform(
    lambda x: x.sum() - x
)

# Number of rivals in same nest
df['IV_nest_count'] = df.groupby(['market', 'nest']).transform('size') - 1
```

### First-Stage Test
$$p_j = \pi_0 + \pi_1 Z_j + \pi_2 x_j + \nu_j$$

**Requirement**: $F$-statistic > 10 (Stock-Yogo weak IV test)
```

### Pattern 4: BLP Estimation Implementation

**Trigger**: "Set up BLP estimation code for my demand model"

**Output**: Working Python code using `pyblp` library.

**Example**:
```python
import pyblp
import numpy as np
import pandas as pd

# Load data
# Expected columns: market_ids, product_ids, shares, prices, characteristics, instruments
data = pd.read_csv('smartphone_data.csv')

# Define product formulation
# Random coefficients on: price, screen_size, brand_apple, brand_samsung
product_formulations = (
    pyblp.Formulation('1 + prices + screen_size + C(brand)'),  # linear
    pyblp.Formulation('1 + prices + screen_size')              # random coefficients
)

# Define agent formulation (demographics for random coefficients)
agent_formulation = pyblp.Formulation('0 + income + age')

# Problem setup
problem = pyblp.Problem(
    product_formulations=product_formulations,
    product_data=data,
    agent_formulation=agent_formulation,
    integration=pyblp.Integration('monte_carlo', size=1000, seed=0)
)

# Initial parameters
initial_sigma = np.diag([0.5, 0.3, 0.2])  # std dev for price, screen, brand coefs
initial_pi = np.zeros((3, 2))              # interactions with demographics

# Optimization
results = problem.solve(
    sigma=initial_sigma,
    pi=initial_pi,
    optimization=pyblp.Optimization('l-bfgs-b', {'maxiter': 1000}),
    iteration=pyblp.Iteration('simple', {'atol': 1e-12})
)

# Results summary
print(results)

# Elasticities
elasticities = results.compute_elasticities()
print("\nOwn-price elasticities:")
print(np.diag(elasticities).mean())

# Micro moments (optional)
# Match observed cross-price elasticity from clickstream data
observed_cross_elasticity = 0.15
micro_moments = pyblp.MicroMoment(
    name='cross_price_elasticity',
    value=observed_cross_elasticity,
    compute=lambda data, results: results.compute_elasticities()[0, 1]
)

# Re-estimate with micro moments
results_micro = problem.solve(
    sigma=initial_sigma,
    pi=initial_pi,
    micro_moments=[micro_moments],
    optimization=pyblp.Optimization('l-bfgs-b')
)

# Counterfactual: Merger simulation
# Merge firm 1 (Apple) and firm 2 (Samsung)
changed_ownership = data['firm_ids'].copy()
changed_ownership[data['firm_ids'] == 2] = 1  # Samsung products now owned by Apple

merger_results = results.solve_merger(
    changed_ownership=changed_ownership
)

print("\nMerger Effects:")
print(f"Price change: {merger_results.price_changes.mean():.2%}")
print(f"Consumer surplus change: ${merger_results.consumer_surplus_change.sum()/1e6:.1f}M")
```

**Key Configuration**:
- **Integration method**: Monte Carlo (1000 draws) for random coefficients
- **Optimization**: L-BFGS-B (handles bounds on sigma)
- **Iteration**: Simple fixed-point iteration for contraction mapping
- **Micro moments**: Optional — improve identification with additional data moments

**Environment variables** (if using cloud compute):
```bash
export PYBLP_CACHE_DIR=/path/to/cache  # Cache intermediate results
export OMP_NUM_THREADS=8               # Parallel computation
```

### Pattern 5: Counterfactual Simulation

**Trigger**: "Design counterfactual simulations for my structural model"

**Example**:
```
User: "I estimated a demand model. Now simulate a price discrimination ban."

Agent:
# Counterfactual: Uniform Pricing Mandate

## Baseline Equilibrium
Current state: Firms set different prices across markets (3rd-degree price discrimination)

## Counterfactual Scenario
Policy: Firms must charge the same price in all markets

## Equilibrium Computation

### Step 1: Compute Baseline
Already have: $p^*, q^*, \pi^*$ from estimation

### Step 2: Constrained Optimization
Firm $f$ solves:
$$\max_{p_f} \sum_{m=1}^M (p_f - mc_{fm}) \cdot q_{fm}(p_f, p_{-f})$$

subject to: $p_f$ is constant across markets $m$

### Step 3: Nash Equilibrium
Iterate best responses until convergence:
```python
def compute_uniform_pricing_equilibrium(results, mc, markets, max_iter=100, tol=1e-6):
    """
    Compute Nash equilibrium under uniform pricing constraint.
    
    Parameters:
    - results: pyblp estimation results
    - mc: marginal costs (firm × market)
    - markets: list of market identifiers
    - max_iter: maximum iterations
    - tol: convergence tolerance
    """
    firms = results.product_data['firm_ids'].unique()
    prices = results.product_data['prices'].copy()
    
    for iteration in range(max_iter):
        prices_old = prices.copy()
        
        for firm in firms:
            # Firm's products
            firm_mask = results.product_data['firm_ids'] == firm
            
            # Objective: total profit across markets
            def profit(p_uniform):
                # Set uniform price for firm's products
                prices_temp = prices.copy()
                prices_temp[firm_mask] = p_uniform
                
                # Compute quantities with new prices
                delta = results.compute_delta(prices=prices_temp)
                shares = results.compute_shares(delta=delta)
                quantities = shares * results.product_data['market_size']
                
                # Profit = (price - mc) * quantity, summed across markets
                firm_profit = ((p_uniform - mc[firm_mask]) * quantities[firm_mask]).sum()
                return -firm_profit  # Minimize negative profit
            
            # Optimize
            from scipy.optimize import minimize_scalar
            res = minimize_scalar(profit, bounds=(mc[firm_mask].min(), prices[firm_mask].max()*2))
            prices[firm_mask] = res.x
        
        # Check convergence
        if np.abs(prices - prices_old).max() < tol:
            print(f"Converged in {iteration+1} iterations")
            break
    
    return prices

# Run counterfactual
uniform_prices = compute_uniform_pricing_equilibrium(results, mc, markets)

# Welfare analysis
baseline_cs = results.consumer_surplus.sum()
counterfactual_cs = results.compute_consumer_surplus(prices=uniform_prices).sum()

print(f"Consumer surplus change: ${(counterfactual_cs - baseline_cs)/1e6:.1f}M")
```

### Step 4: Welfare Decomposition
- **Consumer surplus**: $\Delta CS = CS_{uniform} - CS_{baseline}$
- **Producer profit**: $\Delta \pi = \pi_{uniform} - \pi_{baseline}$
- **Total welfare**: $\Delta W = \Delta CS + \Delta \pi$

### Step 5: Heterogeneous Effects
Break down by market demographics:
```python
# Compute CS change by market income quartile
market_data = results.product_data.groupby('market_ids').agg({
    'median_income': 'first',
    'consumer_surplus_baseline': 'sum',
    'consumer_surplus_counterfactual': 'sum'
})

market_data['cs_change'] = (market_data['consumer_surplus_counterfactual'] 
                             - market_data['consumer_surplus_baseline'])

# Regression: CS change ~ market income
import statsmodels.api as sm
X = sm.add_constant(market_data['median_income'])
model = sm.OLS(market_data['cs_change'], X).fit()
print(model.summary())
```

## Result Interpretation
- **Sign of $\Delta CS$**: Positive → consumers benefit from uniform pricing
- **Distributional effects**: Low-income markets likely gain (were charged high prices before)
- **Producer response**: Firms may reduce product variety under uniform pricing
```

### Pattern 6: Journal Selection & Positioning

**Trigger**: "Which marketing journal should I target?"

**Decision Framework**:
```
User: "I have a structural demand model with field experiment validation. 
      Where should I submit?"

Agent:
# Journal Selection: Decision Tree

## Your Paper Profile
- **Method**: Structural (BLP) + Field Experiment
- **Contribution**: Methodological + Substantive
- **Data**: Proprietary industry data + RCT

## Journal Matching

### Tier 1 Options

**Marketing Science**
- ✅ Structural modeling (perfect fit)
- ✅ Quantitative rigor (BLP estimation)
- ✅ Field experiment (adds credibility)
- ⚠️  Need strong theoretical contribution beyond estimation
- **Recommendation**: Top choice if you have novel identification strategy or methodological innovation

**Journal of Marketing Research (JMR)**
- ✅ Field experiment (strong fit)
- ✅ Structural model (increasingly common)
- ✅ Managerial relevance (JMR values actionability)
- ⚠️  May need to emphasize behavioral insights from experiment
- **Recommendation**: Strong alternative if experiment is primary contribution

**Journal of Marketing (JM)**
- ⚠️  Structural models less common (more conceptual focus)
- ✅ Field experiment with strategic implications
- ⚠️  Need transformative substantive contribution (not just methodological)
- **Recommendation**: Only if findings have broad strategic impact

**Journal of Consumer Research (JCR)**
- ❌ Structural modeling rare (psychological focus)
- ⚠️  Field experiment must reveal consumer psychology
- **Recommendation**: Not a good fit unless uncovering behavioral mechanisms

### Tier 2 Options (if Tier 1 not ready)

**Quantitative Marketing and Economics (QME)**
- ✅✅ Perfect fit for structural + IO
- ✅ Shorter, more technical papers accepted
- **Recommendation**: Safe choice for purely methodological contribution

**International Journal of Research in Marketing (IJRM)**
- ✅ Broad methodological acceptance
- ✅ Faster turnaround than Tier 1
- **Recommendation**: Good backup option

## Positioning Strategy for Marketing Science

### Contribution Statements
1. **Methodological**: "We extend BLP to incorporate [specific innovation: e.g., supply-side learning, dynamic pricing]"
2. **Identification**: "We exploit a novel natural experiment [describe setting] combined with industry cost data to overcome traditional BLP identification challenges"
3. **Substantive**: "We quantify [specific marketing phenomenon: e.g., cross-channel substitution] in a setting where prior work lacked micro-data"
4. **Validation**: "We validate structural estimates with a randomized field experiment, demonstrating out-of-sample predictive accuracy"

### Gap Table (8-12 papers)
| Paper | Method | Data | Gap |
|-------|--------|------|-----|
| Nair (2007, MS) | BLP | Video game consoles | No field experiment validation |
| Datta & Sudhir (2013, MktSci) | Structural | CPG | No supply-side |
| [Your paper] | BLP + RCT | [Your setting] | First to validate BLP with experiment |

### Writing Plan
1. Introduction: Start with field experiment result (surprising finding), then motivate need for structural model
2. Model: Standard BLP with [your innovation]
3. Identification: Emphasize what makes your setting special
4. Estimation: Micro moments from experiment
5. Counterfactuals: Policy-relevant simulations
6. Conclusion: Methodological takeaway + managerial implications

## Final Recommendation
**Target**: Marketing Science
**Backup**: QME (if MS rejects on "insufficient contribution")
**Positioning**: Lead with methodological innovation, support with substantive application
```

## Covered Journals

### Tier 1 (UTD-24)
- **Marketing Science**: Structural models, analytical rigor, IO approach
- **Journal of Marketing Research (JMR)**: Experiments, empirical marketing, broad methods
- **Journal of Marketing (JM)**: Strategy, substantive contributions, managerial relevance
- **Journal of Consumer Research (JCR)**: Consumer psychology, behavioral foundations

### Tier 2 (Strong Field Journals)
- **Quantitative Marketing and Economics (QME)**: Structural econometrics, IO
- **International Journal of Research in Marketing (IJRM)**: Diverse methods, European audience
- **Journal of the Academy of Marketing Science (JAMS)**: Broad marketing science
- **Marketing Letters**: Concise methodological advances

## Common Troubleshooting

### "My identification strategy is weak"

**Problem**: Endogeneity not convincingly addressed.

**Solution checklist**:
1. **For IV**: 
   - First-stage F-stat > 10
   - Exclusion restriction narrative (economic story for why IV affects outcome only through endogenous variable)
   - Overidentification test (if multiple IVs)
2. **For DID**:
   - Parallel trends event study (pre-treatment coefficients near zero)
   - Callaway-Sant'Anna estimator (robust to staggered treatment)
   - Placebo tests (wrong treatment timing, wrong outcomes)
3. **For RDD**:
   - Continuity of covariates at cutoff (McCrary test)
   - Sensitivity to bandwidth choice
   - Donut RDD (exclude observations near cutoff)

**Code example (DID parallel trends test)**:
```python
import pandas as pd
import statsmodels.formula.api as smf

# Event study specification
df['rel_time'] = df['year'] - df['treatment_year']
df['rel_time'] = df['rel_time'].clip(-5, 5)  # Clip to ±5 years

# Create dummies for each rel_time (omit -1 as baseline)
for k in range(-5, 6):
    if k != -1:
        df[f'lead_lag_{k}'] = (df['rel_time'] == k).astype(int)

# Regression
formula = 'log_sales ~ ' + ' + '.join([f'lead_lag_{k}' for k in range(-5, 6) if k != -1])
formula += ' + C(city_id) + C(year)'

model = smf.ols(formula, data=df).fit(cov_type='cluster', cov_kwds={'groups': df['city_id']})
print(model.summary())

# Test pre-trends: H0: lead_lag_-5 = ... = lead_lag_-2 = 0
from scipy.stats import f as f_dist
pre_trend_coefs = [f'lead_lag_{k}' for k in range(-5, -1)]
r_matrix = np.eye(len(model.params))[[model.params.index.get_loc(c) for c in pre_trend_coefs]]
f_stat = model.f_test(r_matrix).fvalue[0][0]
print(f"\nPre-trends F-stat: {f_stat:.2f} (reject if > 2.5)")
```

### "Reviewers say my contribution is incremental"

**Problem**: Paper feels like "BLP applied to new dataset" without sufficient innovation.

**Solution strategies**:
1. **Methodological hook**: Add a novel estimation technique (e.g., Bayesian BLP with hierarchical priors, machine learning for nonparametric heterogeneity)
2. **Substantive hook**: Frame around a first-order marketing question (e.g., "How much do consumers value privacy?" vs. "Estimate demand for smartphones")
3. **Data hook**: Emphasize unique data access (e.g., "First paper with cost data from manufacturer", "Click-stream data reveals consideration sets")
4. **Validation hook**: Out-of-sample predictions, field experiment validation, structural break analysis

**Reframing example**:
```
❌ Weak framing:
"We estimate a BLP model of demand for smartphones."

✅ Strong framing:
"We quantify consumers' willingness-to-pay for privacy features using a structural model validated by a randomized pricing experiment. Our estimates reveal that privacy-conscious consumers have WTP of $150 for enhanced encryption, implying a $12B untapped market for privacy-focused devices."
```

### "My counterfactual results seem unrealistic"

**Problem**: Simulated equilibrium prices/quantities are implausible.

**Debugging checklist**:
1. **Check marginal costs**: Are recovered marginal costs positive? Reasonable magnitude?
   ```python
   mc = results.compute_costs()
   print(f"MC range: ${mc.min():.2f} - ${mc.max():.2f}")
   print(f"Markup range: {((prices - mc) / prices).min():.1%} - {((prices - mc) / prices).max():.1%}")
   ```
2. **Elasticity magnitudes**: Own-price elasticities should be negative and > 1 in absolute value for differentiated goods
   ```python
   elast = results.compute_elasticities()
   print(f"Own-price elasticity range: {np.diag(elast).min():.2f} - {np.diag(elast).max():.2f}")
   # Should be roughly -2 to -5 for typical markets
   ```
3. **Equilibrium convergence**: Did counterfactual optimization converge?
   ```python
   # Add diagnostics to equilibrium solver
   print(f"Iterations: {iteration}, Max price change: {np.abs(prices - prices_old).max():.4f}")
   ```
4. **Boundary solutions**: Are firms hitting corner solutions (price = MC)?

### "I don't have good instruments"

**Problem**: No credible cost shifters or differentiation instruments.

**Alternative strategies**:
1. **Control function approach**: Model the endogeneity explicitly
   ```python
   # Two-stage control function
   # Stage 1: Price regression
   stage1 = smf.ols('price ~ x1 + x2 + x3 + instrument1 + instrument2', data=df).fit()
   df['price_residual'] = stage1.resid
   
   # Stage 2: Include residual as control
   stage2 = smf.ols('log_share ~ price + x1 + x2 + price_residual', data=df).fit()
   ```
2. **Micro moments**: Use additional data moments to identify parameters (clickstream, survey WTP)
3. **Panel data**: Within-product variation + fixed effects
   ```python
   formula = 'log_share ~ price + x1 + x2 + C(product_id) + C(time_id)'
   model = smf.ols(formula, data=df).fit(cov_type='cluster', cov_kwds={'groups': df['product_id']})
   ```
4. **Supply-side moments**: Joint estimation of demand + supply (supply shifters as instruments)

### "I need to write the paper faster"

**Template-driven approach**:
1. Use `examples/manuscript_template.tex` as starting point
2. For each section, reference `references/writing-patterns.md` for reusable templates
3. Generate section drafts in order:
   - Model section first (easiest, most formulaic)
   - Identification section second (follows from model)
   - Results section third (tables + interpretation)
   - Introduction last (now you know what the paper says)

**Time-saving code generation**:
```python
# Auto-generate LaTeX tables from estimation results
results.to_latex('tables/blp_estimates.tex', 
                 caption='Demand Estimation Results',
                 label='tab:blp')

# Auto-generate elasticity table
elasticities_df = pd.DataFrame(
    elasticities, 
    index=product_names, 
    columns=product_names
)
elasticities_df.to_latex('tables/elasticities.tex', float_format='%.2f')
```

## Configuration

This skill has no runtime configuration — it's a reference system. However, for BLP estimation workflows:

**Environment variables**:
```bash
# Computation
export OMP_NUM_THREADS=8              # Parallel cores for pyblp
export PYBLP_CACHE_DIR=/scratch/blp   # Cache directory for large problems

# Reproducibility
export PYTHONHASHSEED=0               # Deterministic hashing
export PYBLP_RANDOM_SEED=12345        # Monte Carlo integration seed
```

**pyblp configuration file** (`~/.pyblp/config.json`):
```json
{
  "integration": {
    "method": "monte_carlo",
    "size": 1000
  },
  "optimization": {
    "method": "l-bfgs-b",
    "options": {
      "maxiter": 1
