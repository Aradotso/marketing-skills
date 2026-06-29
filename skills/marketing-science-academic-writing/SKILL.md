---
name: marketing-science-academic-writing
description: AI agent skill for writing marketing science academic papers from topic selection through structural modeling, identification, estimation, and counterfactual analysis to complete manuscript
triggers:
  - "write a marketing science paper"
  - "help me with a JMR submission"
  - "design a BLP demand estimation model"
  - "create identification strategy for marketing paper"
  - "build counterfactual simulation for merger analysis"
  - "write consumer utility model"
  - "design conjoint analysis experiment"
  - "Marketing Science manuscript"
---

# Marketing Science Academic Writing

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive skill for writing marketing science academic papers covering the full pipeline: topic positioning, consumer utility modeling, causal identification, structural estimation, counterfactual simulations, and manuscript assembly for top-tier journals (Marketing Science, JMR, JM, JCR, JAMS, IJRM, QME, Marketing Letters).

## What This Skill Enables

This skill guides AI agents through the complete **0-to-Draft Pipeline** for quantitative marketing papers:

1. **Topic Positioning**: Gap analysis, journal selection, contribution framing
2. **Consumer Utility Modeling**: Random coefficients logit, nested logit, discrete choice models
3. **Identification & Estimation**: DID, RDD, IV strategies, BLP/GMM estimation
4. **Counterfactuals & Experiments**: Merger simulations, conjoint analysis, field experiments
5. **Manuscript Assembly**: INFORMS-style LaTeX manuscript with all sections

Domain-specific conventions include: utility specification structure, moment construction for structural estimation, identification strategy justification, counterfactual equilibrium computation, and journal-specific reviewer expectations.

## Installation

### Clone the Repository

```bash
git clone https://github.com/liyuanbo1024/marketing-science-writing.git
cd marketing-science-writing
```

### Install for Your AI Agent

**OpenCode:**
```bash
# Linux/macOS
cp -r . ~/.config/opencode/skills/marketing-science-writing/

# Windows PowerShell
Copy-Item -Recurse . "$env:USERPROFILE\.config\opencode\skills\marketing-science-writing\"
```

**Claude Code:**
```bash
# Linux/macOS
cp -r . ~/.claude/skills/marketing-science-writing/

# Windows PowerShell
Copy-Item -Recurse . "$env:USERPROFILE\.claude\skills\marketing-science-writing\"
```

**Cursor:**
```bash
cp -r . ~/.cursor/skills/marketing-science-writing/
```

**Windsurf:**
```bash
cp -r . ~/.windsurf/skills/marketing-science-writing/
```

**Manual (any agent supporting markdown skills):**
Point your agent's skills path to the cloned directory or reference `SKILL.md` directly in configuration.

## Project Structure

```
marketing-science-writing/
├── SKILL.md                          # Main skill definition
├── references/
│   ├── journal-characteristics.md    # 8 journals: profiles, expectations
│   ├── modeling-conventions.md       # Utility models, demand systems
│   ├── identification-guide.md       # DID, RDD, IV strategies
│   ├── estimation-guide.md           # BLP, GMM, MLE, Bayesian
│   ├── counterfactual-guide.md       # Simulation design
│   ├── conjoint-analysis.md          # Conjoint design, WTP
│   ├── field-experiments.md          # Field experiment design
│   ├── marketing-implications.md     # Actionable implications
│   ├── reviewer-expectations.md      # Review process, rebuttals
│   └── writing-patterns.md           # Reusable patterns
├── assets/
│   └── demand-model-reference.md     # Generic demand structures
├── examples/
│   ├── manuscript_template.tex       # INFORMS LaTeX template
│   └── blp_estimation_example.py     # BLP estimation code
└── README.md                         # Documentation
```

## Usage Patterns

### Full Pipeline Mode

User says: *"Take me through the full marketing science pipeline for my paper on smartphone demand estimation."*

**Agent workflow:**
1. Load skill and assess current stage
2. Execute Stage 1: Topic positioning → gap table, journal selection, contributions
3. Stage 2: Consumer utility model → specify utility, derive demand
4. Stage 3: Identification → IV strategy, BLP moments, GMM estimator
5. Stage 4: Counterfactuals → merger simulation design
6. Stage 5: Assembly → complete LaTeX manuscript

### Stage-Specific Mode

Jump to any stage:

```
"Help me design the consumer utility model" → Stage 2
"I need an identification strategy" → Stage 3
"Design counterfactual simulations" → Stage 4
"Write the full manuscript" → Stage 5
```

### Reference Mode

Use as passive reference:

```
"What are Marketing Science's expectations for identification?"
"How do I structure BLP moments?"
"Show me conjoint analysis design for JMR"
```

## Core Methodologies

### 1. Structural Modeling (BLP)

**Consumer Utility Specification:**
```python
# Random coefficients logit utility
# U_ijt = α_i * price_jt + β_i' * X_jt + ξ_jt + ε_ijt
# α_i = α + σ_α * ν_i, β_i = β + Σ * ν_i
# where ν_i ~ N(0, I)

import numpy as np
from scipy.optimize import minimize

def utility(price, X, xi, theta, nu):
    """
    Compute indirect utility for consumer i, product j, market t
    
    Args:
        price: (J,) array of prices
        X: (J, K) array of product characteristics
        xi: (J,) array of unobserved quality
        theta: dict with keys 'alpha', 'beta', 'sigma_alpha', 'Sigma'
        nu: (I, K+1) array of consumer taste draws
    
    Returns:
        U: (I, J) utility matrix
    """
    I, K_plus_1 = nu.shape
    J = len(price)
    
    # Random coefficients
    alpha_i = theta['alpha'] + theta['sigma_alpha'] * nu[:, 0]  # (I,)
    beta_i = theta['beta'] + theta['Sigma'] @ nu[:, 1:].T  # (K, I)
    
    # Utility components
    price_util = -alpha_i[:, None] * price[None, :]  # (I, J)
    char_util = (beta_i.T @ X.T).T  # (I, J)
    xi_util = xi[None, :]  # (1, J) broadcast to (I, J)
    
    U = price_util + char_util + xi_util
    return U

def choice_prob(U):
    """Compute choice probabilities via logit"""
    exp_U = np.exp(U)
    return exp_U / (1 + exp_U.sum(axis=1, keepdims=True))

def market_share(U, integration_weights):
    """Aggregate individual choices to market shares"""
    s_i = choice_prob(U)  # (I, J)
    return (s_i.T @ integration_weights).flatten()  # (J,)
```

**BLP Estimation Workflow:**

```python
def blp_gmm_objective(theta, data, instruments):
    """
    BLP GMM objective function
    
    Args:
        theta: parameter vector [alpha, beta, sigma_alpha, Sigma_elements]
        data: dict with 'price', 'X', 'shares', 'market_ids'
        instruments: (J, L) array of instruments
    
    Returns:
        GMM objective value
    """
    # 1. Unpack parameters
    theta_dict = unpack_theta(theta)
    
    # 2. Simulate consumer types
    nu = np.random.randn(500, len(theta_dict['beta']) + 1)
    weights = np.ones(500) / 500
    
    # 3. Compute predicted shares via contraction mapping
    xi = np.zeros(len(data['shares']))
    for _ in range(100):  # Contraction mapping iteration
        U = utility(data['price'], data['X'], xi, theta_dict, nu)
        s_pred = market_share(U, weights)
        xi += np.log(data['shares']) - np.log(s_pred)
    
    # 4. Construct moments
    moments = instruments.T @ xi  # (L,) moment vector
    
    # 5. GMM objective
    W = optimal_weight_matrix(instruments, xi)  # (L, L)
    obj = moments.T @ W @ moments
    return obj

# Estimate
theta_init = initialize_theta()
result = minimize(blp_gmm_objective, theta_init, 
                  args=(data, instruments), method='BFGS')
theta_hat = result.x
```

**Moment Construction:**

Demand-side moments:
- Product-market dummies × ξ
- Cost shifters × ξ

Supply-side moments:
- Cost shifters × ω (marginal cost shock)

Micro moments (optional):
- Match distributions of price elasticities
- Match correlation between income and WTP

### 2. Causal Identification Strategies

**Difference-in-Differences:**

```python
import pandas as pd
import statsmodels.formula.api as smf

# DID specification for marketing intervention
df = pd.read_csv('marketing_panel.csv')
# Columns: store_id, week, outcome, treated, post

model = smf.ols(
    'outcome ~ treated + post + treated:post + C(store_id) + C(week)',
    data=df
).fit(cov_type='cluster', cov_kwds={'groups': df['store_id']})

print(model.summary())
# Coefficient on treated:post is the ATT
```

**Regression Discontinuity:**

```python
from statsmodels.regression.linear_model import OLS
from statsmodels.tools import add_constant

# RDD at rating threshold (e.g., 4.5 stars)
df = pd.read_csv('rating_data.csv')
df['above_threshold'] = (df['rating'] >= 4.5).astype(int)
df['rating_centered'] = df['rating'] - 4.5

# Local linear regression with bandwidth
bandwidth = 0.5
df_rdd = df[abs(df['rating_centered']) <= bandwidth].copy()

X = add_constant(df_rdd[['above_threshold', 'rating_centered', 
                          'above_threshold * rating_centered']])
y = df_rdd['sales']

rdd_model = OLS(y, X).fit(cov_type='HC1')
print(rdd_model.summary())
# Coefficient on above_threshold is the LATE
```

**Instrumental Variables:**

```python
from linearmodels.iv import IV2SLS

# IV for endogenous price
# Instruments: cost shifters, Hausman instruments
formula = 'sales ~ 1 + characteristics + [price ~ cost_shifter + hausman_iv]'
iv_model = IV2SLS.from_formula(formula, data=df).fit(cov_type='clustered', 
                                                      clusters=df['market_id'])
print(iv_model.summary)
```

### 3. Counterfactual Simulations

**Merger Simulation:**

```python
def merger_simulation(theta, data, firm_ids, merging_firms):
    """
    Simulate post-merger equilibrium prices
    
    Args:
        theta: estimated demand parameters
        data: market data with pre-merger prices, shares
        firm_ids: product-to-firm mapping
        merging_firms: list of firm IDs involved in merger
    
    Returns:
        post_merger_prices, post_merger_shares, welfare_change
    """
    # 1. Compute pre-merger equilibrium
    pre_prices = data['price']
    pre_shares = data['shares']
    
    # 2. Update ownership matrix
    ownership_pre = create_ownership_matrix(firm_ids)
    ownership_post = ownership_pre.copy()
    for f1 in merging_firms:
        for f2 in merging_firms:
            ownership_post[firm_ids == f1, firm_ids == f2] = 1
    
    # 3. Compute marginal costs (from pre-merger FOC)
    mc = invert_pricing_foc(pre_prices, pre_shares, ownership_pre, theta)
    
    # 4. Solve post-merger pricing game
    post_prices = solve_pricing_equilibrium(mc, ownership_post, theta)
    post_shares = compute_shares(post_prices, data['X'], theta)
    
    # 5. Welfare analysis
    consumer_surplus_pre = compute_consumer_surplus(pre_prices, pre_shares, theta)
    consumer_surplus_post = compute_consumer_surplus(post_prices, post_shares, theta)
    
    producer_profit_pre = compute_profit(pre_prices, mc, pre_shares)
    producer_profit_post = compute_profit(post_prices, mc, post_shares)
    
    welfare_change = {
        'delta_CS': consumer_surplus_post - consumer_surplus_pre,
        'delta_PS': producer_profit_post - producer_profit_pre,
        'delta_TS': (consumer_surplus_post + producer_profit_post) - 
                    (consumer_surplus_pre + producer_profit_pre)
    }
    
    return post_prices, post_shares, welfare_change
```

### 4. Conjoint Analysis

**Conjoint Design:**

```python
import itertools
import pandas as pd
from scipy.stats import norm

# Define attributes and levels
attributes = {
    'price': [10, 15, 20, 25],
    'brand': ['A', 'B', 'C'],
    'feature': ['basic', 'premium']
}

# Full factorial design
profiles = list(itertools.product(*attributes.values()))
conjoint_df = pd.DataFrame(profiles, columns=attributes.keys())

# Add dominated profiles check
# (optional: fractional factorial via D-optimal design)

# Export for Qualtrics/online survey
conjoint_df.to_csv('conjoint_profiles.csv', index=False)
```

**Part-Worth Estimation:**

```python
import statsmodels.formula.api as smf

# Responses: choice_data.csv with columns [respondent_id, profile_id, choice]
choices = pd.read_csv('choice_data.csv')
profiles = pd.read_csv('conjoint_profiles.csv')

# Merge
data = choices.merge(profiles, on='profile_id')

# Hierarchical Bayes (simplified frequentist version)
model = smf.logit('choice ~ C(brand) + C(feature) + price', data=data).fit()
print(model.summary())

# WTP calculation
wtp_premium = -model.params['C(feature)[T.premium]'] / model.params['price']
print(f"WTP for premium feature: ${wtp_premium:.2f}")
```

## Key Configuration

### LaTeX Manuscript Template

Use `examples/manuscript_template.tex` as base:

```latex
\documentclass[12pt]{article}
\usepackage{amsmath, amssymb, amsthm}
\usepackage{natbib}
\usepackage{graphicx}
\usepackage{booktabs}

% INFORMS-style spacing
\usepackage{setspace}
\doublespacing

\title{Your Title Here}
\author{Author Names \\ Affiliations}
\date{\today}

\begin{document}
\maketitle

\begin{abstract}
Your abstract (150 words max for Marketing Science).
\end{abstract}

\section{Introduction}
% 5-7 pages

\section{Literature Review}
% 3-4 pages

\section{Model}
% 4-6 pages: utility specification, demand derivation

\section{Identification and Estimation}
% 3-5 pages: identification strategy, moments, estimator

\section{Data}
% 2-3 pages

\section{Results}
% 4-6 pages: parameter estimates, elasticities, model fit

\section{Counterfactual Simulations}
% 3-5 pages: merger, policy, welfare

\section{Marketing Implications}
% 2-3 pages: actionable insights for managers

\section{Conclusion}
% 1-2 pages

\bibliographystyle{informs2014}
\bibliography{references}

\end{document}
```

### Python Environment Setup

For BLP estimation and counterfactuals:

```bash
# Create virtual environment
python3 -m venv marketing-env
source marketing-env/bin/activate  # Windows: marketing-env\Scripts\activate

# Install dependencies
pip install numpy scipy pandas statsmodels linearmodels matplotlib seaborn pyblp
```

**PyBLP Integration** (high-level BLP wrapper):

```python
import pyblp
import pandas as pd

# Load data in PyBLP format
# Columns: market_ids, product_ids, shares, prices, X1, X2, ..., demand_instruments0, ...
data = pd.read_csv('blp_data.csv')

# Define problem
problem = pyblp.Problem(
    product_formulations=(
        pyblp.Formulation('0 + prices + X1 + X2'),  # Linear characteristics
        pyblp.Formulation('0 + prices + X1')         # Random coefficients
    ),
    product_data=data
)

# Solve
results = problem.solve(
    sigma=np.eye(2),  # Initial Sigma for random coefficients
    pi=None,
    optimization=pyblp.Optimization('bfgs')
)

print(results)

# Compute elasticities
elasticities = results.compute_elasticities()
print(elasticities)
```

## Common Patterns

### Pattern 1: Gap Table Construction

When starting a paper:

```markdown
| Paper | Context | Method | Finding | Gap |
|-------|---------|--------|---------|-----|
| AuthorA (JMR 2021) | Online advertising | RCT | +10% CTR | Doesn't consider long-run | 
| AuthorB (MktSci 2020) | Search ads | Structural | Spillover effects | Single channel |
| AuthorC (JM 2019) | Display ads | Field exp | Brand lift | No demand model |

**Our contribution:**
1. Multi-channel structural model with long-run dynamics
2. Credible identification via IV + field experiment
3. Welfare analysis of ad policy counterfactuals
```

### Pattern 2: Utility Specification Justification

In Model section:

```markdown
We specify consumer i's indirect utility for product j in market t as:

U_ijt = α_i price_jt + β_i' X_jt + ξ_jt + ε_ijt

where:
- α_i = α + σ_α ν_iα: random price coefficient (income heterogeneity)
- β_i = β + Σ ν_iβ: random taste for characteristics
- ξ_jt: unobserved (to econometrician) quality
- ε_ijt: i.i.d. type-I extreme value error

**Assumptions:**
1. ν_i ~ N(0, I): normal taste distribution (allows closed-form shares)
2. E[ξ_jt | Z_jt] = 0: exogeneity of instruments Z
3. Independence of irrelevant alternatives relaxed via random coefficients

**Justification:**
- Random coefficients capture preference heterogeneity (Train 2009)
- Nests multinomial logit (σ_α = 0, Σ = 0)
- Standard in empirical IO (Berry et al. 1995, Nevo 2000)
```

### Pattern 3: Identification Narrative

In Identification section:

```markdown
**Challenge:** price_jt is endogenous (correlated with ξ_jt).

**Strategy:** Instrumental variables based on:
1. **Cost shifters:** Input prices, distance to manufacturing plants (excluded from demand)
2. **Hausman instruments:** Prices of same product in other markets (Hausman 1996)
3. **BLP instruments:** Sum of characteristics of other products by same firm (Berry et al. 1995)

**Validity checks:**
- First-stage F-stat > 10 (Stock-Yogo)
- Overidentification test: Hansen J p-value = 0.32 (cannot reject)
- Placebo test: no correlation with lagged ξ
```

### Pattern 4: Counterfactual Setup

In Counterfactual section:

```markdown
**Scenario:** Hypothetical merger between Firm A and Firm B.

**Mechanism:** Post-merger, merged entity internalizes cannibalization across its portfolio → higher prices.

**Computation:**
1. Use pre-merger FOC to back out marginal costs: mc_j = price_j + Δ^{-1} s_j
2. Update ownership matrix Δ to reflect merger
3. Solve post-merger Nash equilibrium in prices (fixed point)
4. Compute new shares, consumer surplus, producer profits

**Welfare decomposition:**
- ΔCS: consumer surplus change
- ΔPS: producer profit change (split by merged vs. non-merged firms)
- ΔTS = ΔCS + ΔPS: total welfare change
```

## Journal Selection Quick Reference

| Journal | Best For | Key Requirements |
|---------|----------|------------------|
| **Marketing Science** | Structural models, analytical rigor, methodological innovation | Strong identification, BLP/GMM, counterfactuals, technical appendix |
| **JMR** | Empirical marketing, experiments, behavioral mechanisms | Clean causal design (RCT/DID/RDD), replication, multiple studies |
| **JM** | Broad strategy, substantive impact, multi-method | Managerial relevance, large sample, robustness checks |
| **JCR** | Consumer psychology, lab experiments, theory building | Psychological mechanisms, moderation/mediation, controlled experiments |
| **JAMS** | Conceptual + empirical, meta-analysis, reviews | Broad appeal, mixed methods, literature synthesis |
| **IJRM** | European context, diverse methods, international | Context-specific insights, data from multiple countries |
| **QME** | Structural IO, pure econometrics, game theory | Formal modeling, analytical solutions, computational methods |
| **Marketing Letters** | Concise empirical findings, methodological notes | Sharp contribution (< 5000 words), quick turnaround |

## Troubleshooting

### Issue: BLP Contraction Mapping Not Converging

**Solution:**
- Reduce tolerance threshold gradually (start at 1e-6, increase to 1e-12)
- Use better initial guess for ξ (e.g., log(observed share) - log(outside share))
- Check for collinearity in characteristics
- Ensure sufficient instruments (rule of thumb: L ≥ 2K where K = # parameters)

```python
def contraction_mapping(s_observed, theta, nu, max_iter=1000, tol=1e-12):
    xi = np.log(s_observed) - np.log(1 - s_observed.sum())  # Initial guess
    for iteration in range(max_iter):
        s_pred = compute_shares(xi, theta, nu)
        xi_new = xi + 0.5 * (np.log(s_observed) - np.log(s_pred))  # Damping
        if np.max(np.abs(xi_new - xi)) < tol:
            return xi_new
        xi = xi_new
    raise ValueError("Contraction mapping did not converge")
```

### Issue: Weak Instruments (First-Stage F < 10)

**Solution:**
- Add more instruments (BLP sum of characteristics, cost shifters)
- Use LIML instead of 2SLS (more robust to weak instruments)
- Consider control function approach if exclusion restriction questionable

```python
from linearmodels.iv import IVLIML

model = IVLIML.from_formula(
    'sales ~ 1 + X [price ~ instruments]',
    data=df
).fit()
```

### Issue: Reviewer Asks for "Economic Significance"

**Response pattern:**
```markdown
We translate the coefficient into economically meaningful units:

- A $1 increase in price → -0.15 log-share change → -12% market share
- At mean price ($50) and mean share (5%), this implies:
  - Elasticity: -12% / 2% = -6
  - Revenue effect: (+2% price) × (-12% quantity) = -10% revenue
  - Monthly revenue loss: $1.2M for average product
  
This is substantial compared to typical profit margins (15-20% in this industry).
```

### Issue: "Not Enough Robustness Checks"

**Standard robustness battery:**
1. Alternative IV sets (report first-stage F, overID test)
2. Different demand specifications (logit, nested logit, random coefficients)
3. Subsample analysis (by market size, time period)
4. Placebo tests (lagged outcomes, non-treated markets)
5. Alternative clustering (product-level, market-level, two-way)

### Issue: Counterfactual Equilibrium Solver Diverges

**Solution:**
```python
def solve_pricing_equilibrium(mc, ownership, theta, damping=0.3, max_iter=500):
    """Bertrand-Nash equilibrium with damping"""
    prices = mc * 1.5  # Initial guess: 50% markup
    for iteration in range(max_iter):
        shares = compute_shares(prices, theta)
        jacobian = compute_share_jacobian(prices, theta)
        foc = shares + ownership * (jacobian @ (prices - mc))
        
        # Newton step with damping
        step = np.linalg.solve(jacobian, foc)
        prices_new = prices - damping * step
        
        if np.max(np.abs(prices_new - prices)) < 1e-6:
            return prices_new
        prices = prices_new
    raise ValueError("Pricing equilibrium did not converge")
```

## Reference Files

When user asks domain-specific questions, consult:

- `references/journal-characteristics.md` → journal selection, editorial priorities
- `references/modeling-conventions.md` → utility specs, demand systems, notation
- `references/identification-guide.md` → DID, RDD, IV, structural identification
- `references/estimation-guide.md` → BLP, GMM, MLE, Bayesian methods
- `references/counterfactual-guide.md` → equilibrium computation, welfare analysis
- `references/conjoint-analysis.md` → conjoint design, WTP estimation
- `references/reviewer-expectations.md` → rebuttal strategies, common critiques
- `references/writing-patterns.md` → introduction templates, transition phrases

## Additional Resources

**BLP Estimation:**
- PyBLP documentation: https://pyblp.readthedocs.io
- Conlon & Gortmaker (2020) tutorial

**Causal Inference:**
- Angrist & Pischke (2009) *Mostly Harmless Econometrics*
- Cunningham (2021) *Causal Inference: The Mixtape*

**Marketing Journals:**
- Marketing Science: https://pubsonline.informs.org/journal/mksc
- JMR: https://journals.sagepub.com/home/mrj

**LaTeX:**
- INFORMS style files: https://pubsonline.informs.org/page/authors/style-guide

---

**When to trigger this skill:**

User mentions writing for marketing journals (Marketing Science, JMR, JM, JCR, JAMS, IJRM, QME, Marketing Letters), structural modeling, BLP estimation, consumer utility models, causal identification strategies, counterfactual simulations, conjoint analysis, or asks for help with any stage of the marketing paper pipeline.
