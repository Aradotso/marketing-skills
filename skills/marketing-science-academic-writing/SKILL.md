---
name: marketing-science-academic-writing
description: AI agent skill for writing marketing science academic papers from topic selection through structural modeling, identification, estimation, and counterfactual analysis to complete manuscript
triggers:
  - "write a marketing science paper"
  - "help me structure a JMR submission"
  - "design a consumer utility model for demand estimation"
  - "set up BLP structural estimation"
  - "create counterfactual simulations for merger analysis"
  - "build identification strategy with instrumental variables"
  - "format paper for Marketing Science journal"
  - "design conjoint analysis for marketing research"
---

# Marketing Science Academic Writing Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive skill for AI coding agents to guide users through writing quantitative marketing science papers — from topic selection through consumer utility modeling, identification design, structural estimation, counterfactual simulations, to a complete first draft.

Covers 8 flagship marketing journals: **Marketing Science, JMR, JM, JCR** (UTD-24) and **JAMS, IJRM, QME, Marketing Letters**.

## What This Skill Enables

This skill gives AI agents expertise in the full **0-to-Draft Pipeline** for marketing science research:

| Stage | Output | Key Deliverable |
|-------|--------|----------------|
| **Stage 1**: Topic Positioning | Gap analysis, journal selection, contribution statement | Target journal + novelty positioning |
| **Stage 2**: Consumer Utility Model | Utility specification, demand derivation, notation system | Model section draft (§3) |
| **Stage 3**: Identification & Estimation | Identification strategy (DID/RDD/IV), estimator choice, specification tests | Method section draft (§4) |
| **Stage 4**: Counterfactuals & Experiments | Counterfactual design, conjoint analysis, field experiment protocols | Results section draft (§5-6) |
| **Stage 5**: Writing & Assembly | Introduction, literature review, implications, abstract | Complete manuscript |

### Domain-Specific Expertise

Unlike generic scientific writing skills, this encodes marketing-specific conventions:
- **Structural modeling**: BLP demand estimation, random coefficients logit, aggregate demand systems
- **Causal identification**: Difference-in-differences, regression discontinuity, instrumental variables in marketing contexts
- **Consumer behavior**: Utility specification, heterogeneous preferences, choice modeling
- **Field experiments**: A/B testing, pricing experiments, digital marketing trials
- **Counterfactual analysis**: Merger simulations, policy evaluation, welfare analysis
- **Journal expectations**: Reviewer norms, positioning strategies, rebuttal patterns

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
cp -r marketing-science-writing ~/.config/opencode/skills/

# Windows PowerShell
Copy-Item -Recurse marketing-science-writing "$env:USERPROFILE\.config\opencode\skills\"
```

**Claude Code:**
```bash
# Linux/macOS
cp -r marketing-science-writing ~/.claude/skills/

# Windows PowerShell
Copy-Item -Recurse marketing-science-writing "$env:USERPROFILE\.claude\skills\"
```

**Codex:**
```bash
# Linux/macOS
cp -r marketing-science-writing ~/.agents/skills/

# Windows PowerShell
Copy-Item -Recurse marketing-science-writing "$env:USERPROFILE\.agents\skills\"
```

**Cursor / Windsurf:**
```bash
# Cursor
cp -r marketing-science-writing ~/.cursor/skills/

# Windsurf
cp -r marketing-science-writing ~/.windsurf/skills/
```

The skill follows the [agentskills.io specification](https://agentskills.io/specification) with YAML frontmatter and will be auto-discovered by compatible agents.

## Core Usage Patterns

### Pattern 1: Full Pipeline (0-to-Draft)

When a user wants to write a complete paper from scratch:

```
User: "I want to write a Marketing Science paper on smartphone demand estimation using BLP. Take me through the full pipeline."

Agent Actions:
1. Load marketing-science-academic-writing skill
2. Stage 1: Topic Positioning
   - Ask: "What's your substantive marketing question?"
   - Search 8-12 related papers in Marketing Science, QME, JMR
   - Build gap table comparing identification strategies
   - Recommend: "Target Marketing Science — structural demand estimation fits tier-1 expectations"
   - Output: Contribution list (3-5 numbered points)

3. Stage 2: Consumer Utility Model
   - Ask: "What product characteristics matter to consumers?"
   - Design utility specification:
     U_ijt = α_i * price_jt + β_i * X_jt + ξ_jt + ε_ijt
     where α_i ~ N(α_bar, σ_α), β_i ~ N(β_bar, Σ_β)
   - Derive aggregate demand via random coefficients logit
   - Specify supply side: Bertrand pricing, constant marginal cost
   - Output: Complete Model section draft with notation table

4. Stage 3: Identification & Estimation
   - Ask: "What exogenous variation do you have?"
   - Design identification strategy:
     * BLP instruments: characteristics of other products
     * Cost shifters: input prices, exchange rates
   - Specify GMM estimator with optimal weighting matrix
   - Output: Method section draft with moment conditions

5. Stage 4: Counterfactuals
   - Ask: "What counterfactual scenarios matter for marketing?"
   - Design simulations:
     * Merger simulation: predict post-merger prices
     * New product introduction: compute cannibalization
     * Price discrimination: compare uniform vs. personalized pricing
   - Output: Results section draft with welfare analysis

6. Stage 5: Assembly
   - Write Introduction (hook → gap → contributions → roadmap)
   - Write Literature Review (organize by identification strategy)
   - Write Marketing Implications (actionable insights for managers)
   - Write Abstract (150 words, structured)
   - Format in INFORMS LaTeX style
   - Output: Complete manuscript.pdf
```

**Key files used:**
- `SKILL.md`: Main pipeline logic
- `references/modeling-conventions.md`: Utility specification guide
- `references/identification-guide.md`: Identification strategy templates
- `references/estimation-guide.md`: BLP estimation procedures
- `references/counterfactual-guide.md`: Counterfactual design patterns
- `examples/manuscript_template.tex`: LaTeX formatting

### Pattern 2: Stage-Specific Assistance

When a user already has partial work and needs help with one stage:

```
User: "I have my utility model. Help me design the identification strategy for endogeneity in price."

Agent Actions:
1. Load identification-guide.md
2. Assess endogeneity source: price set by firms who observe ξ_jt
3. Recommend IV strategy:
   - Hausman instruments: prices in other markets
   - BLP instruments: sum of characteristics of other products
   - Cost shifters: input prices (if available)
4. Formalize moment conditions:
   E[Z_jt * ξ_jt(θ)] = 0
   where Z_jt = [X_jt, BLP_instruments_jt, cost_shifters_jt]
5. Specify 2-stage GMM:
   - Stage 1: θ_hat_1 = argmin Q(θ, W_I)
   - Stage 2: θ_hat_2 = argmin Q(θ, W_opt(θ_hat_1))
6. Output: Method section draft with identification argument
```

**Key files used:**
- `references/identification-guide.md`
- `references/estimation-guide.md`

### Pattern 3: Journal-Specific Positioning

When a user needs to choose a target journal:

```
User: "I have a paper on digital advertising effectiveness using field experiments. Which journal should I target?"

Agent Actions:
1. Load journal-characteristics.md
2. Assess paper profile:
   - Method: Field experiment (A/B test)
   - Contribution: Causal estimate of ad effectiveness
   - Substantive: Digital marketing strategy
3. Match to journals:
   - JMR: Strong fit — empirical causal inference, substantive marketing
   - Marketing Science: Possible if structural model added
   - JM: Possible if broader strategic implications
   - JCR: No — not consumer psychology focused
4. Recommend JMR as primary target
5. Load reviewer-expectations.md for JMR
6. Output:
   - Positioning strategy: "Causal identification of ad effectiveness in digital markets"
   - Introduction structure: Hook with managerial problem → gap in causal estimates → experimental design → implications
   - Key reviewer expectations: Clean identification, external validity, heterogeneous effects
```

**Key files used:**
- `references/journal-characteristics.md`
- `references/reviewer-expectations.md`

### Pattern 4: Structural Estimation Code Generation

When a user needs working code for BLP estimation:

```python
# User: "Generate Python code for BLP demand estimation with random coefficients."

# Agent output (using examples/blp_estimation_example.py as template):

import numpy as np
from scipy.optimize import minimize
from scipy.stats import norm

class BLPModel:
    """
    Berry, Levinsohn, Pakes (1995) demand estimation.
    Random coefficients logit with aggregate data.
    """
    
    def __init__(self, data, instruments):
        """
        Args:
            data: DataFrame with columns [market_id, product_id, share, price, X1, X2, ...]
            instruments: DataFrame with columns [Z1, Z2, ...] (same length as data)
        """
        self.data = data
        self.Z = instruments.values
        self.N_markets = data['market_id'].nunique()
        self.N_products = len(data) // self.N_markets
        self.N_sim = 500  # Number of simulation draws
        
        # Precompute simulation draws (halton or random)
        self.nu = np.random.randn(self.N_sim, 2)  # 2 random coefficients (price, X1)
        
    def compute_shares(self, delta, sigma):
        """
        Compute predicted market shares given mean utilities (delta) and random coef SD (sigma).
        
        Args:
            delta: Mean utilities (N_markets * N_products,)
            sigma: Standard deviations of random coefficients (2,) for [price, X1]
        
        Returns:
            shares: Predicted market shares (N_markets * N_products,)
        """
        price = self.data['price'].values
        X1 = self.data['X1'].values
        
        # Compute utilities for each simulation draw
        # U_ijs = delta_j + sigma_price * nu_i * price_j + sigma_X1 * nu_i * X1_j
        utilities = delta.reshape(-1, 1)  # (J, 1)
        utilities += sigma[0] * self.nu[:, 0] * price.reshape(-1, 1)  # (J, S)
        utilities += sigma[1] * self.nu[:, 1] * X1.reshape(-1, 1)
        
        # Add outside option (utility = 0)
        exp_utilities = np.exp(utilities)
        exp_utilities_with_outside = np.vstack([exp_utilities, np.ones((1, self.N_sim))])
        
        # Choice probabilities
        probs = exp_utilities / exp_utilities_with_outside.sum(axis=0)
        
        # Aggregate over simulation draws
        shares = probs.mean(axis=1)
        
        return shares
    
    def invert_shares(self, sigma, max_iter=1000, tol=1e-6):
        """
        BLP contraction mapping: invert observed shares to get mean utilities (delta).
        
        Iterate: delta_{t+1} = delta_t + log(s_observed) - log(s_predicted(delta_t, sigma))
        
        Args:
            sigma: Random coefficient SDs
        
        Returns:
            delta: Mean utilities
        """
        delta = np.log(self.data['share'].values)  # Initial guess
        
        for _ in range(max_iter):
            shares_pred = self.compute_shares(delta, sigma)
            delta_new = delta + np.log(self.data['share'].values) - np.log(shares_pred)
            
            if np.max(np.abs(delta_new - delta)) < tol:
                return delta_new
            
            delta = delta_new
        
        raise ValueError("BLP contraction did not converge")
    
    def gmm_objective(self, theta):
        """
        GMM objective: Q(theta) = xi' * Z * W * Z' * xi
        
        where xi are demand shocks (unobserved quality) recovered from
        delta = X * beta + xi
        
        Args:
            theta: Parameter vector [beta (K,), sigma (2,)]
        
        Returns:
            Q: GMM objective value
        """
        K = self.data.shape[1] - 4  # Number of product characteristics
        beta = theta[:K]
        sigma = theta[K:]
        
        # Invert shares to get delta
        delta = self.invert_shares(sigma)
        
        # Recover structural errors: xi = delta - X * beta
        X = self.data[['price', 'X1', 'X2']].values
        xi = delta - X @ beta
        
        # GMM moments: E[Z' * xi] = 0
        moments = self.Z.T @ xi  # (L,) where L = number of instruments
        
        # Weighting matrix (2-step GMM: use optimal W from first stage)
        W = np.linalg.inv(self.Z.T @ self.Z)  # Identity for first stage
        
        # Objective
        Q = moments.T @ W @ moments
        
        return Q
    
    def estimate(self, initial_guess=None):
        """
        Estimate BLP model via GMM.
        
        Returns:
            results: dict with estimated parameters and standard errors
        """
        if initial_guess is None:
            K = self.data.shape[1] - 4
            initial_guess = np.concatenate([
                np.zeros(K),  # beta
                np.ones(2)    # sigma
            ])
        
        # Stage 1: Minimize GMM objective
        result = minimize(
            self.gmm_objective,
            initial_guess,
            method='Nelder-Mead',
            options={'maxiter': 10000}
        )
        
        theta_hat = result.x
        K = len(theta_hat) - 2
        
        # Stage 2: Update weighting matrix and re-estimate
        delta_hat = self.invert_shares(theta_hat[K:])
        X = self.data[['price', 'X1', 'X2']].values
        xi_hat = delta_hat - X @ theta_hat[:K]
        
        # Optimal weighting matrix
        Omega = (self.Z.T * xi_hat**2) @ self.Z / len(xi_hat)
        W_opt = np.linalg.inv(Omega)
        
        # Re-estimate with optimal W (modify gmm_objective to use W_opt)
        # [Code for stage 2 omitted for brevity — same structure]
        
        return {
            'beta': theta_hat[:K],
            'sigma': theta_hat[K:],
            'delta': delta_hat,
            'xi': xi_hat,
            'objective': result.fun
        }

# Usage example
if __name__ == '__main__':
    import pandas as pd
    
    # Load data (market_id, product_id, share, price, X1, X2)
    data = pd.read_csv('data/smartphone_data.csv')
    
    # Construct BLP instruments: sum of characteristics of other products
    data['blp_instrument_X1'] = data.groupby('market_id')['X1'].transform(
        lambda x: x.sum() - x
    )
    instruments = data[['blp_instrument_X1', 'cost_shifter']]
    
    # Estimate
    model = BLPModel(data, instruments)
    results = model.estimate()
    
    print("Price coefficient (beta_price):", results['beta'][0])
    print("Random coefficient SD (sigma_price):", results['sigma'][0])
```

**Key files used:**
- `examples/blp_estimation_example.py`
- `references/estimation-guide.md`
- `references/modeling-conventions.md`

### Pattern 5: Counterfactual Simulation Design

When a user needs to evaluate a marketing policy:

```
User: "I have BLP estimates for the laptop market. Design a counterfactual to evaluate a merger between Dell and HP."

Agent Actions:
1. Load counterfactual-guide.md
2. Specify counterfactual equilibrium problem:
   - Pre-merger: Dell and HP set prices non-cooperatively
   - Post-merger: Merged entity internalizes cannibalization
   
3. Formalize pricing FOCs:
   Pre-merger (firm f):
     ∂π_f / ∂p_j = s_j + Σ_{k ∈ J_f} (p_k - mc_k) * ∂s_k/∂p_j = 0
   
   Post-merger (merged firm):
     ∂π_merged / ∂p_j = s_j + Σ_{k ∈ J_Dell ∪ J_HP} (p_k - mc_k) * ∂s_k/∂p_j = 0

4. Compute counterfactual prices:
   - Solve system of FOCs numerically
   - Use estimated demand elasticities ∂s_k/∂p_j from BLP model
   
5. Welfare analysis:
   - Consumer surplus: CS = E[max_j U_ij] (log-sum formula)
   - Producer profit: π = Σ_j (p_j - mc_j) * s_j * M
   - Total welfare: W = CS + π

6. Output code:
```

```python
def counterfactual_merger(blp_model, firm1_products, firm2_products, marginal_costs):
    """
    Simulate post-merger equilibrium prices and welfare.
    
    Args:
        blp_model: Estimated BLPModel instance
        firm1_products: List of product indices for firm 1
        firm2_products: List of product indices for firm 2
        marginal_costs: Array of marginal costs for all products
    
    Returns:
        results: dict with pre/post prices, quantities, welfare
    """
    from scipy.optimize import fsolve
    
    # Pre-merger equilibrium (current prices)
    prices_pre = blp_model.data['price'].values
    shares_pre = blp_model.data['share'].values
    
    # Compute demand elasticities at current prices
    elasticities = blp_model.compute_elasticities(prices_pre)
    # elasticities[j, k] = (∂s_j / ∂p_k) * (p_k / s_j)
    
    # Post-merger pricing FOCs
    def pricing_focs(prices_post):
        """
        FOCs for merged firm: ∂π/∂p_j = 0
        """
        shares_post = blp_model.compute_shares_at_prices(prices_post)
        
        focs = []
        merged_products = firm1_products + firm2_products
        
        for j in merged_products:
            # FOC: s_j + Σ_{k in merged} (p_k - mc_k) * ∂s_k/∂p_j = 0
            foc_j = shares_post[j]
            for k in merged_products:
                markup = prices_post[k] - marginal_costs[k]
                elasticity_kj = elasticities[k, j]
                foc_j += markup * elasticity_kj * shares_post[k] / prices_post[j]
            focs.append(foc_j)
        
        # Other firms' FOCs unchanged
        for j in range(len(prices_post)):
            if j not in merged_products:
                foc_j = shares_post[j]
                firm_j_products = [j]  # Assuming single-product firms
                for k in firm_j_products:
                    markup = prices_post[k] - marginal_costs[k]
                    elasticity_kj = elasticities[k, j]
                    foc_j += markup * elasticity_kj * shares_post[k] / prices_post[j]
                focs.append(foc_j)
        
        return focs
    
    # Solve for post-merger prices
    prices_post = fsolve(pricing_focs, prices_pre)
    shares_post = blp_model.compute_shares_at_prices(prices_post)
    
    # Welfare analysis
    market_size = 1000000  # Assume 1M potential consumers
    
    # Consumer surplus (log-sum formula for logit)
    cs_pre = market_size * np.log(1 + np.sum(np.exp(blp_model.delta_pre)))
    cs_post = market_size * np.log(1 + np.sum(np.exp(blp_model.compute_delta(prices_post))))
    
    # Producer profit
    profit_pre = np.sum((prices_pre - marginal_costs) * shares_pre * market_size)
    profit_post = np.sum((prices_post - marginal_costs) * shares_post * market_size)
    
    # Total welfare
    welfare_pre = cs_pre + profit_pre
    welfare_post = cs_post + profit_post
    
    return {
        'prices_pre': prices_pre,
        'prices_post': prices_post,
        'shares_pre': shares_pre,
        'shares_post': shares_post,
        'consumer_surplus_change': cs_post - cs_pre,
        'profit_change': profit_post - profit_pre,
        'welfare_change': welfare_post - welfare_pre,
        'price_increase_pct': 100 * (prices_post - prices_pre) / prices_pre
    }

# Example usage
results = counterfactual_merger(
    blp_model=model,
    firm1_products=[0, 1, 2],  # Dell products
    firm2_products=[3, 4],      # HP products
    marginal_costs=np.array([300, 350, 400, 320, 380])  # Estimated from supply side
)

print(f"Average price increase: {results['price_increase_pct'].mean():.1f}%")
print(f"Consumer surplus change: ${results['consumer_surplus_change']/1e6:.1f}M")
print(f"Welfare change: ${results['welfare_change']/1e6:.1f}M")
```

**Key files used:**
- `references/counterfactual-guide.md`
- `references/estimation-guide.md`

## Key Reference Files

### `references/journal-characteristics.md`

Detailed profiles of 8 journals with:
- Typical methodology mix (structural vs. reduced-form vs. experiments)
- Positioning strategies for each journal
- Reviewer expectations (what they look for, common rejection reasons)
- Example papers for each journal type

**When to use:** Stage 1 (topic positioning), whenever user asks "which journal should I target?"

### `references/modeling-conventions.md`

Comprehensive guide to consumer utility models:
- **Discrete choice**: Multinomial logit, nested logit, random coefficients logit
- **Demand systems**: AIDS, linear demand, aggregate discrete choice
- **Utility specifications**: Price, product characteristics, unobserved quality, heterogeneity
- **Notation conventions**: Indexing (i=consumer, j=product, t=time), Greek letters (α for price, β for characteristics, ξ for unobserved quality)

**When to use:** Stage 2 (model building), when user asks "how do I specify utility?" or "what's the standard demand model?"

### `references/identification-guide.md`

Identification strategies for marketing:
- **Difference-in-differences**: Parallel trends, staggered adoption, event studies
- **Regression discontinuity**: Sharp vs. fuzzy RD, bandwidth selection, local polynomial estimation
- **Instrumental variables**: Hausman instruments, BLP instruments (sum of characteristics), cost shifters, shift-share instruments
- **Structural identification**: Exclusion restrictions in demand/supply systems, revealed preference conditions

**When to use:** Stage 3 (identification design), when user asks "how do I address endogeneity?" or "what instruments should I use?"

### `references/estimation-guide.md`

Estimation procedures:
- **BLP (Berry-Levinsohn-Pakes)**: Contraction mapping, GMM objective, 2-stage vs. 1-stage, micro moments
- **GMM**: Moment conditions, weighting matrix, standard errors (clustered, heteroskedasticity-robust)
- **MLE**: Likelihood function for discrete choice, computational issues, simulation-based estimation
- **Bayesian**: MCMC for hierarchical models, prior specification, convergence diagnostics

**When to use:** Stage 3 (estimation), when user asks "how do I estimate this?" or "what estimator should I use?"

### `references/counterfactual-guide.md`

Counterfactual simulation design:
- **Types**: Merger simulation, new product introduction, price discrimination, advertising regulation
- **Equilibrium computation**: Solving FOCs, fixed-point iteration, Newton-Raphson
- **Welfare analysis**: Consumer surplus (log-sum formula), producer profit, deadweight loss
- **Sensitivity analysis**: Robustness to demand elasticity, cost assumptions

**When to use:** Stage 4 (counterfactuals), when user asks "what counterfactuals should I run?" or "how do I compute welfare?"

### `references/conjoint-analysis.md`

Conjoint experiment design:
- **Attributes and levels**: Choice of product features, number of profiles
- **Experimental design**: Full factorial vs. fractional factorial, D-optimal design
- **Estimation**: Mixed logit, hierarchical Bayes, WTP computation
- **Validation**: Holdout prediction, external validity checks

**When to use:** Stage 2-4 (when user chooses conjoint as primary method), when user asks "how do I design a conjoint study?"

### `references/field-experiments.md`

Field experiment design and analysis:
- **Randomization**: Individual vs. cluster randomization, stratification, re-randomization
- **Power analysis**: Minimum detectable effect, sample size calculation
- **Estimation**: ATE, CATE (conditional average treatment effects), heterogeneous effects
- **Threats to validity**: Spillovers, non-compliance, attrition, Hawthorne effects

**When to use:** Stage 2-4 (when user chooses field experiment as primary method), when user asks "how do I design an A/B test?"

### `references/marketing-implications.md`

Framework for writing actionable marketing implications:
- **Structure**: What → Why → How (insight → mechanism → action)
- **Audience**: Brand managers, pricing analysts, product developers, policymakers
- **Specificity**: Quantify effects ("10% price increase → 5% demand decrease"), provide thresholds
- **Robustness**: Discuss boundary conditions, when implications hold vs. don't hold

**When to use:** Stage 5 (writing), specifically for "Marketing Implications" section

### `references/reviewer-expectations.md`

What reviewers look for (by journal):
- **Marketing Science**: Formal model correctness, identification rigor, computational transparency
- **JMR**: Clean causal inference, substantive contribution, managerial relevance
- **JM**: Broad strategic insights, multi-study validation, theoretical depth
- **JCR**: Psychological mechanisms, process evidence, conceptual novelty

**When to use:** Stage 1 (positioning), Stage 5 (revision), when user asks "what do reviewers care about?"

### `references/writing-patterns.md`

Reusable writing templates:
- **Introduction hook**: "Firms spend $X billion on Y, but little is known about Z"
- **Gap statement**: "Prior work has examined A, but overlooked B because of limitation C"
- **Contribution list**: "First, we show... Second, we document... Third, we demonstrate..."
- **Model intuition**: "Consumers choose j to maximize utility... This implies..."
- **Identification argument**: "We exploit exogenous variation in X driven by policy Y, which affects Z only through its impact on X"
- **Robustness discussion**: "Our results are robust to alternative specifications (Appendix Table A1)..."

**When to use:** Stage 5 (writing), for any section

## Configuration

### Environment Variables

If generating code that requires API access (e.g., OpenAI for text generation tasks), reference credentials as:

```python
import os

# Example: If using OpenAI API for auxiliary tasks (NOT for BLP estimation itself)
OPENAI_API_KEY = os.getenv('OPENAI_API_KEY')
if not OPENAI_API_KEY:
    raise ValueError("Set OPENAI_API_KEY environment variable")
```

**Never hardcode keys** — always use `os.getenv()`.

### LaTeX Dependencies

The skill assumes users have LaTeX installed for manuscript generation. Required packages:
```latex
\usepackage{amsmath, amssymb, amsthm}
\usepackage{natbib}
\usepackage{graphicx}
\usepackage{booktabs}
\usepackage{setspace}
```

Check installation:
```bash
pdflatex --version
```

If missing, recommend installation (e.g., `sudo apt install texlive-full` on Linux).

### Python Dependencies

For BLP estimation and counterfactuals:
```bash
pip install numpy scipy pandas matplotlib seaborn
```

For advanced structural estimation:
```bash
pip install pyblp  # PyBLP library (alternative to custom implementation)
```

## Troubleshooting

### "Contraction mapping not converging"

**Symptom:** BLP invert_shares() raises `ValueError("BLP contraction did not converge")`

**Causes:**
1. Initial guess for delta is too far from truth
2. sigma (random coefficient SD) is too large
3. Market shares have zeros or near-zeros

**Solutions:**
```python
# 1. Better initial guess
delta_init = np.log(data['share'].values) - np.log(data['share'].values.mean())

# 2. Constrain sigma during optimization
bounds = [(None, None)] * K + [(0.01, 2.0)] * 2  # sigma between 0.01 and 2

# 3. Add small constant to shares to avoid log(0)
data['share'] = data['share'].clip(lower=1e-6)
```

### "GMM objective is negative"

**Symptom:** `gmm_objective()` returns negative value

**Cause:** Weighting matrix W is not positive definite, or moments are scaled incorrectly.

**Solution:**
```python
# Regularize weighting matrix
W = np.linalg.inv(self.Z.T @ self.Z + 1e-6 * np.eye(L))

# Or use identity matrix for first stage
W = np.eye(L)
```

### "Counterfactual prices are unrealistic"

**Symptom:** Post-merger prices are 10x pre-merger prices, or negative

**Causes:**
1. Demand elasticities computed incorrectly
2. FOC system is poorly specified
3. Initial guess for fsolve is too far from equilibrium

**Solutions:**
```python
# 1. Verify elasticity matrix is negative diagonal
assert np.all(np.diag(elasticities) < 0), "Own-
