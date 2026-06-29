---
name: marketing-science-writing
description: AI agent skill for writing marketing science academic papers from topic selection through structural modeling, identification, estimation, and full draft assembly.
triggers:
  - "write a marketing science paper"
  - "help me design a consumer utility model"
  - "identify causal effects in marketing data"
  - "design counterfactual simulations for structural model"
  - "write a paper for Marketing Science journal"
  - "help with BLP demand estimation"
  - "design a conjoint analysis study"
  - "set up identification strategy with instrumental variables"
---

# Marketing Science Academic Writing Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive skill for AI coding agents to guide researchers through the complete pipeline of writing marketing science academic papers — from topic positioning and consumer utility modeling through identification design, structural estimation, counterfactual simulations, and full manuscript assembly.

Covers 8 flagship marketing journals: **Marketing Science, JMR, JM, JCR** (UTD-24 tier) and **JAMS, IJRM, QME, Marketing Letters**.

## What This Skill Enables

This skill teaches AI agents to help researchers with:

- **Topic Positioning**: Gap analysis, journal selection, contribution framing
- **Consumer Utility Modeling**: Micro-founded demand systems (logit, nested logit, random coefficients)
- **Identification Strategy**: DID, RDD, IV, structural identification for causal inference
- **Estimation Methods**: BLP, GMM, MLE, Bayesian estimation workflows
- **Counterfactual Simulations**: Merger analysis, policy simulations, welfare calculations
- **Experimental Design**: Field experiments, conjoint analysis, A/B testing
- **Full Draft Assembly**: LaTeX manuscript generation with INFORMS formatting

The skill is **domain-specific** for marketing journals, encoding conventions that generic academic writing skills don't cover (e.g., structural model presentation, identification justification, counterfactual design, reviewer expectations).

## Installation

### 1. Clone Repository

```bash
git clone https://github.com/liyuanbo1024/marketing-science-writing.git
cd marketing-science-writing
```

### 2. Install for Your AI Agent

#### OpenCode

```bash
# Linux/macOS
mkdir -p ~/.config/opencode/skills
cp -r . ~/.config/opencode/skills/marketing-science-writing

# Windows PowerShell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.config\opencode\skills"
Copy-Item -Recurse . "$env:USERPROFILE\.config\opencode\skills\marketing-science-writing"
```

Or add to `~/.config/opencode/opencode.json`:

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

#### Claude Code

```bash
# Linux/macOS
mkdir -p ~/.claude/skills
cp -r . ~/.claude/skills/marketing-science-writing

# Windows PowerShell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills"
Copy-Item -Recurse . "$env:USERPROFILE\.claude\skills\marketing-science-writing"
```

#### Cursor / Windsurf / Other Agents

```bash
# Cursor
cp -r . ~/.cursor/skills/marketing-science-writing

# Windsurf
cp -r . ~/.windsurf/skills/marketing-science-writing

# Generic agent with custom skills directory
cp -r . /your/agent/skills/path/marketing-science-writing
```

## The 5-Stage Pipeline

The skill guides agents through a **0-to-Draft Pipeline**:

| Stage | Output | Key File Reference |
|-------|--------|-------------------|
| **1. Topic Positioning** | Gap table, journal choice, contribution list | `references/journal-characteristics.md` |
| **2. Consumer Utility Model** | Utility specification, demand derivation, notation | `references/modeling-conventions.md` |
| **3. Identification & Estimation** | Identification strategy, estimator, specification tests | `references/identification-guide.md`, `references/estimation-guide.md` |
| **4. Counterfactuals & Experiments** | Counterfactual design, conjoint/field experiment | `references/counterfactual-guide.md`, `references/conjoint-analysis.md` |
| **5. Full Draft Assembly** | Complete LaTeX manuscript | `examples/manuscript_template.tex` |

## Usage Patterns

### Pattern 1: Full Pipeline Mode

User says: *"Take me through the full marketing science pipeline for a paper on dynamic pricing in ride-sharing markets."*

Agent workflow:

```python
# Stage 1: Topic Positioning
# Read: references/journal-characteristics.md
# Output: Gap table, recommend Marketing Science or QME
# Ask user: "Confirm target journal: Marketing Science?"

# Stage 2: Consumer Utility Model
# Read: references/modeling-conventions.md
# Design: Random coefficients logit with time-varying price coefficients
# Output: §3 Model section draft with notation table

# Stage 3: Identification
# Read: references/identification-guide.md
# Design: IV strategy using weather shocks + cost shifters
# Output: §4 Identification and Estimation section

# Stage 4: Counterfactuals
# Read: references/counterfactual-guide.md
# Design: Simulate merger, price cap, surge pricing ban
# Output: §5 Counterfactual Results section

# Stage 5: Assembly
# Read: examples/manuscript_template.tex
# Generate: Full LaTeX draft with all sections
```

### Pattern 2: Stage-Specific Help

User says: *"I have my structural model. Help me design the identification strategy."*

Agent action:
1. Read `references/identification-guide.md`
2. Ask user for endogenous variables, available instruments
3. Propose identification strategy (e.g., BLP instruments + cost shifters)
4. Draft §4.1 Identification subsection

### Pattern 3: Code Generation for Estimation

User says: *"Generate Python code for BLP estimation with random coefficients."*

Agent uses `examples/blp_estimation_example.py`:

```python
import numpy as np
import pandas as pd
from scipy.optimize import minimize
from scipy.stats import norm

class BLPModel:
    """
    BLP (1995) Random Coefficients Logit Demand Estimation
    
    Environment variables needed:
    - DATA_PATH: path to market-level data CSV
    """
    
    def __init__(self, data_path=None):
        if data_path is None:
            import os
            data_path = os.getenv('DATA_PATH', 'data/market_data.csv')
        self.data = pd.read_csv(data_path)
        
    def compute_shares(self, delta, sigma, nu):
        """
        Compute predicted market shares via simulation
        
        Args:
            delta: mean utilities (J×1)
            sigma: std dev of random coefficients (K×1)
            nu: simulation draws (N×K)
        
        Returns:
            shares: predicted shares (J×1)
        """
        J = len(delta)
        N = nu.shape[0]
        
        # Individual-level utilities
        # u_ij = delta_j + sigma · x_j · nu_i
        X = self.data[['price', 'horsepower', 'mpg']].values
        u = delta[:, None] + (X @ np.diag(sigma)) @ nu.T  # J×N
        
        # Choice probabilities
        exp_u = np.exp(u)
        denom = 1 + exp_u.sum(axis=0)  # 1×N
        probs = exp_u / denom  # J×N
        
        # Average across simulation draws
        shares = probs.mean(axis=1)
        return shares
    
    def contraction_mapping(self, delta_init, observed_shares, sigma, nu, tol=1e-8, max_iter=1000):
        """
        BLP contraction mapping to invert shares → delta
        
        delta^{t+1} = delta^t + log(s_observed) - log(s_predicted)
        """
        delta = delta_init.copy()
        for iteration in range(max_iter):
            s_pred = self.compute_shares(delta, sigma, nu)
            delta_new = delta + np.log(observed_shares) - np.log(s_pred)
            
            if np.abs(delta_new - delta).max() < tol:
                return delta_new
            delta = delta_new
        
        raise ValueError(f"Contraction mapping did not converge in {max_iter} iterations")
    
    def gmm_objective(self, theta, Z, W):
        """
        GMM objective: g(theta)' W g(theta)
        
        Args:
            theta: [sigma_price, sigma_hp, sigma_mpg]
            Z: instruments (T×M matrix)
            W: weighting matrix (M×M)
        
        Returns:
            GMM objective value
        """
        sigma = theta
        nu = np.random.randn(500, 3)  # 500 simulation draws
        
        # Solve for delta via contraction mapping
        observed_shares = self.data['market_share'].values
        delta_init = np.log(observed_shares) - np.log(1 - observed_shares.sum())
        delta = self.contraction_mapping(delta_init, observed_shares, sigma, nu)
        
        # Compute structural errors: xi = delta - X*beta
        X = self.data[['price', 'horsepower', 'mpg']].values
        beta = np.linalg.lstsq(X, delta, rcond=None)[0]
        xi = delta - X @ beta
        
        # Moment conditions: E[Z'*xi] = 0
        moments = Z.T @ xi  # M×1
        
        # GMM objective
        obj = moments.T @ W @ moments
        return obj
    
    def estimate(self, instruments_cols, initial_sigma=None):
        """
        Run two-step GMM estimation
        
        Args:
            instruments_cols: list of column names in self.data
            initial_sigma: starting values for [sigma_price, sigma_hp, sigma_mpg]
        
        Returns:
            theta_hat: estimated random coefficient std devs
            se: standard errors
        """
        Z = self.data[instruments_cols].values
        
        if initial_sigma is None:
            initial_sigma = np.array([0.5, 0.5, 0.5])
        
        # Step 1: Identity weighting matrix
        W = np.eye(Z.shape[1])
        result1 = minimize(self.gmm_objective, initial_sigma, args=(Z, W),
                          method='Nelder-Mead', options={'maxiter': 100})
        theta1 = result1.x
        
        # Step 2: Optimal weighting matrix
        # (In practice: estimate Omega = E[Z'*xi*xi'*Z] then W = inv(Omega))
        # Simplified here for demonstration
        W_optimal = np.eye(Z.shape[1])  # Replace with actual Omega^{-1}
        result2 = minimize(self.gmm_objective, theta1, args=(Z, W_optimal),
                          method='BFGS')
        theta_hat = result2.x
        
        # Standard errors (from inverse Hessian)
        se = np.sqrt(np.diag(result2.hess_inv))
        
        return theta_hat, se

# Usage example
if __name__ == '__main__':
    model = BLPModel()  # Reads from $DATA_PATH
    
    # Instruments: BLP instruments (sum of other firms' characteristics)
    # + cost shifters (e.g., steel_price, labor_cost)
    instruments = ['blp_price_sum', 'blp_hp_sum', 'blp_mpg_sum', 
                   'steel_price', 'labor_cost']
    
    theta_hat, se = model.estimate(instruments)
    
    print("Estimated Random Coefficient Std Devs:")
    print(f"  σ_price = {theta_hat[0]:.4f} (SE: {se[0]:.4f})")
    print(f"  σ_hp    = {theta_hat[1]:.4f} (SE: {se[1]:.4f})")
    print(f"  σ_mpg   = {theta_hat[2]:.4f} (SE: {se[2]:.4f})")
```

**Key points**:
- Use environment variables for data paths: `os.getenv('DATA_PATH')`
- Contraction mapping to invert shares → mean utilities
- Two-step GMM with optimal weighting matrix
- BLP instruments: sum of rivals' characteristics + cost shifters

### Pattern 4: Counterfactual Simulation

User says: *"Design counterfactual simulations for a merger between Firm A and Firm B."*

Agent reads `references/counterfactual-guide.md` and generates:

```python
def simulate_merger_counterfactual(model, firm_a_products, firm_b_products):
    """
    Simulate post-merger equilibrium prices and welfare
    
    Args:
        model: estimated BLPModel instance
        firm_a_products: list of product indices owned by Firm A
        firm_b_products: list of product indices owned by Firm B
    
    Returns:
        results: dict with pre/post prices, quantities, consumer surplus, profits
    """
    # Pre-merger: Solve for Nash equilibrium prices
    pre_prices = solve_bertrand_equilibrium(model, ownership_matrix_pre)
    pre_shares = model.compute_shares(model.delta, model.sigma, model.nu)
    pre_cs = compute_consumer_surplus(model, pre_prices)
    pre_profits = compute_profits(model, pre_prices, pre_shares, ownership_matrix_pre)
    
    # Post-merger: Update ownership matrix
    ownership_matrix_post = ownership_matrix_pre.copy()
    # Merge Firm A and Firm B
    for i in firm_a_products:
        for j in firm_b_products:
            ownership_matrix_post[i, j] = 1
            ownership_matrix_post[j, i] = 1
    
    # Solve for new Nash equilibrium
    post_prices = solve_bertrand_equilibrium(model, ownership_matrix_post)
    post_shares = model.compute_shares(
        model.delta + model.beta_price * (post_prices - pre_prices),
        model.sigma, model.nu
    )
    post_cs = compute_consumer_surplus(model, post_prices)
    post_profits = compute_profits(model, post_prices, post_shares, ownership_matrix_post)
    
    return {
        'pre_prices': pre_prices,
        'post_prices': post_prices,
        'price_change_pct': (post_prices - pre_prices) / pre_prices * 100,
        'consumer_surplus_change': post_cs - pre_cs,
        'profit_change': post_profits - pre_profits,
        'total_welfare_change': (post_cs - pre_cs) + (post_profits - pre_profits)
    }

def solve_bertrand_equilibrium(model, ownership, tol=1e-6, max_iter=1000):
    """
    Solve for Bertrand-Nash equilibrium prices given ownership structure
    """
    prices = model.data['price'].values.copy()
    marginal_costs = model.data['marginal_cost'].values
    
    for iteration in range(max_iter):
        # Compute demand elasticities
        shares = model.compute_shares(model.delta, model.sigma, model.nu)
        elasticities = compute_elasticity_matrix(model, prices)
        
        # First-order conditions: p_j - mc_j = -s_j / (∂s_j/∂p_j) [single-product firm]
        # Multi-product firm: (P - MC) = -Δ^{-1} * s, where Δ_jk = ∂s_j/∂p_k * ownership_jk
        Delta = elasticities * ownership
        prices_new = marginal_costs - np.linalg.solve(Delta, shares)
        
        if np.abs(prices_new - prices).max() < tol:
            return prices_new
        prices = 0.5 * prices + 0.5 * prices_new  # Damping for stability
    
    raise ValueError("Bertrand equilibrium did not converge")
```

## Journal-Specific Guidance

The skill includes detailed profiles for 8 journals. Quick reference:

| Journal | Best For | Key Requirement |
|---------|----------|----------------|
| **Marketing Science** | Structural models, formal theory | Rigorous identification + model microfoundations |
| **JMR** | Empirical marketing, experiments | Clean identification + substantive insights |
| **JM** | Broad marketing strategy | Substantive contribution + managerial relevance |
| **JCR** | Consumer psychology | Psychological mechanism + process evidence |
| **QME** | IO-style structural models | Structural econometrics + counterfactuals |
| **JAMS** | Broad marketing science | Conceptual + empirical rigor |
| **IJRM** | Diverse methodologies | European audience + methodological innovation |
| **Marketing Letters** | Concise empirical findings | Sharp result + 6000-word limit |

**Selection rule encoded in the skill**:
- Structural model with counterfactuals → Marketing Science or QME
- Field experiment with causal effects → JMR or JM
- Consumer behavior with psychological mechanism → JCR
- Broad strategic question + substantive data → JM or JAMS

## Configuration

### Environment Variables

Set these for data access and API usage:

```bash
# Data paths
export DATA_PATH="/path/to/market_data.csv"
export INSTRUMENTS_PATH="/path/to/instruments.csv"

# LaTeX output
export OUTPUT_DIR="./output"
export MANUSCRIPT_TEMPLATE="./examples/manuscript_template.tex"

# Optional: API keys for literature search
export SEMANTIC_SCHOLAR_API_KEY="your_key_here"
```

### Skill Configuration

The skill reads from `SKILL.md` and all files in `references/`. No additional configuration needed.

For custom journal profiles, add to `references/journal-characteristics.md`:

```markdown
## Custom Journal Name

**Focus**: Brief description

**Typical Length**: X-Y pages

**Methodology**: Quantitative / Qualitative / Mixed

**Key Expectations**:
- Expectation 1
- Expectation 2
```

## Key Commands & Patterns

### Trigger Phrases

The agent activates this skill when you say:

- "Write a marketing science paper on [topic]"
- "Help me design a consumer utility model for [problem]"
- "Set up identification strategy with instrumental variables"
- "Generate BLP estimation code"
- "Design counterfactual simulations for [scenario]"
- "Write a conjoint analysis study for [product]"
- "Position my paper for Marketing Science / JMR / JM / JCR"

### Quick Commands

| What You Say | What Agent Does |
|-------------|----------------|
| "Run Stage 1" | Topic positioning + journal selection |
| "Draft the model section" | Generate §3 with utility model + notation |
| "Design identification" | Propose DID/RDD/IV strategy + draft §4 |
| "Generate estimation code" | Output Python/R code for BLP/GMM |
| "Simulate counterfactual" | Design counterfactual analysis + welfare calculations |
| "Write full draft" | Assemble complete LaTeX manuscript |

## LaTeX Manuscript Generation

The skill uses `examples/manuscript_template.tex` to generate INFORMS-style manuscripts:

```latex
\documentclass[12pt]{article}
\usepackage{informs3}  % Marketing Science style
\usepackage{amsmath, amssymb, amsthm}
\usepackage{natbib}
\usepackage{graphicx}
\usepackage{booktabs}

\title{Your Paper Title}
\author{Author Names \\ Affiliations}

\begin{document}
\maketitle

\begin{abstract}
% Auto-generated from Stage 5
\end{abstract}

\section{Introduction}
% Auto-generated: motivation, gap, contributions, roadmap

\section{Literature Review}
% Auto-generated: gap table, positioning

\section{Model}
\subsection{Consumer Utility}
% From Stage 2: utility specification, notation

\subsection{Demand System}
% From Stage 2: choice probabilities, aggregate demand

\subsection{Supply Side}
% From Stage 2: pricing game, marginal costs

\section{Identification and Estimation}
\subsection{Identification Strategy}
% From Stage 3: IV justification, exogeneity discussion

\subsection{Estimation}
% From Stage 3: GMM/MLE/Bayesian method

\subsection{Data}
% User provides, agent structures

\section{Results}
\subsection{Parameter Estimates}
% From Stage 3: coefficient table, elasticities

\subsection{Model Fit}
% From Stage 3: specification tests, predicted vs. actual

\section{Counterfactual Analysis}
% From Stage 4: counterfactual scenarios, welfare tables

\section{Managerial Implications}
% From references/marketing-implications.md

\section{Conclusion}
% Auto-generated: summary, limitations, future research

\bibliographystyle{informs2014}
\bibliography{references}
\end{document}
```

## Troubleshooting

### Issue: "Skill not activating"

**Solution**: Ensure the skill directory name matches the `name` field in YAML frontmatter (`marketing-science-writing`). Check your agent's skills path:

```bash
# OpenCode
cat ~/.config/opencode/opencode.json | grep skills

# Claude Code
ls ~/.claude/skills/

# Cursor
ls ~/.cursor/skills/
```

### Issue: "Agent doesn't use references"

**Solution**: The agent should auto-load reference files when the skill activates. Explicitly mention the stage or topic:

```
"Read references/identification-guide.md and design an IV strategy for my pricing problem."
```

### Issue: "BLP code fails with singular matrix error"

**Cause**: Poor starting values or weak instruments in `gmm_objective`.

**Solution**:
1. Check instrument strength (F-stat > 10)
2. Use better starting values: `initial_sigma = [0.1, 0.1, 0.1]` (closer to zero)
3. Add regularization to avoid numerical singularities:

```python
# In contraction_mapping, add damping
delta_new = 0.7 * delta + 0.3 * (delta + np.log(observed_shares) - np.log(s_pred))
```

### Issue: "Counterfactual equilibrium not converging"

**Cause**: Bertrand equilibrium solver stuck in non-convex region.

**Solution**:
1. Use adaptive damping: `prices = alpha * prices + (1-alpha) * prices_new`, where `alpha` decreases over iterations
2. Try multiple starting points: pre-merger prices, marginal costs, uniform markups

```python
# Multi-start approach
best_prices = None
best_obj = np.inf
for start in [pre_prices, marginal_costs * 1.2, marginal_costs * 1.5]:
    try:
        prices = solve_bertrand_equilibrium(model, ownership, start)
        obj = compute_nash_objective(prices, model, ownership)
        if obj < best_obj:
            best_prices, best_obj = prices, obj
    except:
        continue
```

### Issue: "Journal reviewers want more robustness checks"

**Solution**: The skill includes a checklist in `references/reviewer-expectations.md`. Common requests:

- **Alternative specifications**: Nested logit, probit, heterogeneous coefficients
- **Instrument validity**: Sargan-Hansen test, reduced-form F-stats
- **Placebo tests**: Falsification tests on unaffected outcomes
- **Subsample analysis**: By market size, time period, product category
- **Alternative identification**: Different IV sets, DID with alternative control groups

The skill guides the agent to pre-emptively draft these in §5.3 Robustness Checks.

## Reference Files Summary

| File | Content |
|------|---------|
| `references/journal-characteristics.md` | Detailed profiles of 8 journals (scope, length, expectations) |
| `references/modeling-conventions.md` | Utility specifications, demand systems (logit, nested logit, RC), notation |
| `references/identification-guide.md` | DID, RDD, IV strategies with marketing examples |
| `references/estimation-guide.md` | BLP, GMM, MLE, Bayesian methods with code templates |
| `references/counterfactual-guide.md` | Counterfactual design, equilibrium computation, welfare analysis |
| `references/conjoint-analysis.md` | CBC design, WTP estimation, hierarchical Bayes |
| `references/field-experiments.md` | Randomization, power analysis, heterogeneous effects |
| `references/marketing-implications.md` | Framework for managerial implications section |
| `references/reviewer-expectations.md` | What reviewers look for, rebuttal tips, revision strategies |
| `references/writing-patterns.md` | Reusable sentence patterns for common sections |

## Advanced: Custom Pipelines

You can extend the skill for non-standard workflows:

### Custom: Add Meta-Analysis Section

Edit `SKILL.md` to add a Stage 2.5:

```markdown
### Stage 2.5: Meta-Analysis of Prior Effects

**Goal**: Synthesize effect sizes from prior literature.

**Agent actions**:
- Search for papers reporting similar treatment effects
- Extract effect sizes and standard errors
- Run random-effects meta-analysis
- Output forest plot and summary estimate
- Use summary estimate as prior for Bayesian estimation
```

Then reference from `examples/meta_analysis.R`:

```r
library(meta)

# Load extracted effect sizes
effects <- read.csv(Sys.getenv('META_DATA_PATH'))

# Random-effects meta-analysis
ma <- metagen(TE = effect_size, seTE = std_error, data = effects,
              studlab = study_name, sm = "MD")

# Forest plot
pdf(file.path(Sys.getenv('OUTPUT_DIR'), 'forest_plot.pdf'))
forest(ma, sortvar = TE, xlim = c(-2, 2), 
       xlab = "Standardized Effect Size")
dev.off()

# Use pooled estimate as Bayesian prior
prior_mean <- ma$TE.random
prior_se <- ma$seTE.random
```

### Custom: Multi-Method Papers

For papers combining structural + reduced-form:

1. Run Stage 3 twice: once for IV (reduced-form), once for BLP (structural)
2. Cross-reference estimates: "The structural price elasticity of -2.3 is consistent with the IV estimate of -2.1 in Table 3."
3. Use reduced-form as validation of structural model assumptions

## Real-World Example Workflow

**Scenario**: You're writing a paper on the effect of influencer marketing on consumer demand in the beauty industry.

**Step 1**: Topic positioning

```
You: "I'm studying how Instagram influencer endorsements affect demand for beauty products. Help me position this for a top marketing journal."

Agent (activates skill):
- Reads references/journal-characteristics.md
- Searches for related papers (influencer marketing, social media, beauty industry)
- Builds gap table:
  | Paper | Method | Gap |
  |-------|--------|-----|
  | Smith (2020, JMR) | Survey experiment | Doesn't measure actual sales |
  | Jones (2021, MS) | Structural model of fashion | Different product category |
  | ...
- Recommends: "Marketing Science (structural model + counterfactuals) or JMR (field experiment + causal ID)"
- You choose: Marketing Science
```

**Step 2**: Consumer utility model

```
You: "Design the consumer utility model."

Agent:
- Reads references/modeling-conventions.md
- Proposes random coefficients logit:
  u_ijt = α_i · price_jt + β_1 · quality_jt + β_2 · influencer_endorsement_jt + ξ_jt + ε_ijt
  where α_i ~ N(α̅, σ_α²), heterogeneous price sensitivity
- Derives choice probabilities via simulation
- Specifies supply side: Bertrand pricing with marginal cost = production cost + influencer payment
- Outputs §3 Model section (15 pages with derivations)
```

**Step 3**: Identification

```
You: "I have data on influencer payments and Instagram follower counts. Design the identification strategy."

Agent:
- Reads references/identification-guide.md
- Proposes instruments:
  1. Excluded IV: lagged follower counts (affects endorsement but not demand directly)
  2. Cost shifter: platform algorithm changes (affects influencer payment but not ξ_jt)
- Drafts §4.1 Identification:
  - Discusses endogeneity of influencer_endorsement (brands select influencers based on unobserved ξ)
  - Justifies exogeneity of instruments (lagged followers determined before campaign)
  - First-stage F-statistic calculation
```

**Step 4**: Estimation code

```
You: "Generate the BLP estimation code."

Agent:
- Uses examples/blp_estimation_example.py as template
- Customizes for your model (adds influencer_endorsement variable)
- Outputs 200-line Python script with:
  - Data loading from $DATA_PATH
  - Contraction mapping with damping
  - GMM objective with your instruments
  - Two-step estimation
  - Standard errors via bootstrap
```

**Step 5**: Counterfactuals

```
You: "Design counterfactual: what if influencer payments were banned?"

Agent:
- Reads references/counterfactual-guide.md
- Designs simulation:
  1. Set influencer_endorsement = 0 for all products
  2. Resolve Bertrand equilibrium (firms optimize only price, not influencer budget)
  3. Compute welfare change:
     - Consumer surplus: higher (lower prices due to less spending on influencers)
     - Producer profit: lower (less differentiation)
     - Total welfare: ambiguous (depends on influencer effectiveness β_2)
- Outputs §6 Counterfactual Analysis with 3 tables
```

**Step 6**: Full draft assembly

```
You: "Write the full paper."

Agent:
- Reads examples/manuscript_template.tex
- Assembles:
  - §1 Introduction (motivation: $X billion influencer market, policy debates)
  - §2 Literature (gap table from Stage 1)
  - §3 Model (from Stage 2)
  - §4 Identification & Estimation (from Stage 3)
  - §5 Data (you provide descriptive stats, agent formats tables)
  - §6 Results (parameter estimates + model fit from Stage 3)
  - §7 Counterfactual (from Stage 5)
  - §8 Managerial Implications (reads references/marketing-implications.md)
  - §9 Conclusion
- Outputs: manuscript.tex (50 pages), compiles to PDF
```

## Additional Resources

- **BLP Estimation Tutorial**: `examples/blp_estimation_example.py` includes full working code
- **LaTeX Template**: `examples/manuscript_template.tex` INFORMS-ready
- **Notation Guide**: `references/modeling-conventions.md` §2 "Standard Notation System"
- **Reviewer Checklist**: `references/reviewer-expectations.md` use before submission

## Contributing

To extend this skill:

1. Add new reference files to `references/` (e.g., `references/bayesian
