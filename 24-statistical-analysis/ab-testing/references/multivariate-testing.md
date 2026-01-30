# Multi-Variant and Multivariate Testing

## Overview

Multi-variant testing (MVT) tests multiple variants simultaneously, while multivariate testing examines interactions between multiple factors. This guide covers both approaches.

## Multi-Variant Testing (A/B/C/D...)

Testing 3+ variants of a single element:

### Sample Size Considerations

```python
from statsmodels.stats.power import FTestAnovaPower
import numpy as np

def sample_size_multivariant(n_variants, baseline_rate, mde, alpha=0.05, power=0.80):
    """
    Calculate sample size for multi-variant test
    
    Parameters:
    -----------
    n_variants : int
        Number of variants to test
    baseline_rate : float
        Expected baseline conversion rate
    mde : float
        Minimum detectable effect (relative)
    """
    # Convert to effect size (Cohen's f)
    treatment_rate = baseline_rate * (1 + mde)
    
    # Pooled standard deviation (for proportions)
    p_mean = (baseline_rate + treatment_rate) / 2
    sd_pooled = np.sqrt(p_mean * (1 - p_mean))
    
    # Cohen's f
    effect_size_f = abs(treatment_rate - baseline_rate) / sd_pooled
    
    # ANOVA power analysis
    power_analysis = FTestAnovaPower()
    n_per_group = power_analysis.solve_power(
        effect_size=effect_size_f,
        alpha=alpha,
        power=power,
        k_groups=n_variants
    )
    
    return {
        'n_per_variant': n_per_group,
        'total_sample_size': n_per_group * n_variants,
        'effect_size_f': effect_size_f
    }

# Example: 4 variants
result = sample_size_multivariant(
    n_variants=4,
    baseline_rate=0.10,
    mde=0.15,  # 15% relative lift
    alpha=0.05,
    power=0.80
)

print("Multi-Variant Test (4 variants):")
print(f"  Sample size per variant: {result['n_per_variant']:.0f}")
print(f"  Total sample size: {result['total_sample_size']:.0f}")

# Compare to pairwise A/B tests with Bonferroni correction
n_comparisons = 3  # A vs B, A vs C, A vs D
adjusted_alpha = 0.05 / n_comparisons

from statsmodels.stats.power import zt_ind_solve_power
from statsmodels.stats.proportion import proportion_effectsize

effect = proportion_effectsize(0.10, 0.115)
n_ab = zt_ind_solve_power(effect_size=effect, alpha=adjusted_alpha, power=0.80)

print(f"\nPairwise A/B with Bonferroni:")
print(f"  Sample size per variant: {n_ab:.0f}")
print(f"  Total for 3 tests: {n_ab * 2 * 3:.0f}")
```

### Analysis with ANOVA

```python
import numpy as np
import pandas as pd
from scipy import stats
from statsmodels.stats.multicomp import pairwise_tukeyhsd

# Simulate 4-variant test
np.random.seed(42)
n_per_variant = 1000

conversions = {
    'A': np.random.binomial(1, 0.10, n_per_variant),  # Control
    'B': np.random.binomial(1, 0.12, n_per_variant),  # +20% lift
    'C': np.random.binomial(1, 0.11, n_per_variant),  # +10% lift
    'D': np.random.binomial(1, 0.105, n_per_variant)  # +5% lift
}

df_multi = pd.DataFrame({
    'variant': np.repeat(list(conversions.keys()), n_per_variant),
    'converted': np.concatenate(list(conversions.values()))
})

# Summary statistics
summary = df_multi.groupby('variant')['converted'].agg([
    ('n', 'count'),
    ('conversions', 'sum'),
    ('rate', 'mean')
])

print("="*60)
print("MULTI-VARIANT TEST RESULTS")
print("="*60)
print(summary)

# Chi-square test of independence
contingency_table = pd.crosstab(df_multi['variant'], df_multi['converted'])
chi2, p_overall, dof, expected = stats.chi2_contingency(contingency_table)

print(f"\nOverall Test:")
print(f"  Chi-square statistic: {chi2:.4f}")
print(f"  p-value: {p_overall:.4f}")

if p_overall < 0.05:
    print("  ✓ At least one variant differs from others")
else:
    print("  ✗ No significant differences detected")

# Pairwise comparisons with Tukey HSD
print("\n" + "="*60)
print("PAIRWISE COMPARISONS (Tukey HSD)")
print("="*60)

tukey = pairwise_tukeyhsd(endog=df_multi['converted'], 
                          groups=df_multi['variant'], 
                          alpha=0.05)
print(tukey)

# Identify best variant
best_variant = summary['rate'].idxmax()
best_rate = summary.loc[best_variant, 'rate']

print(f"\n✓ Best variant: {best_variant} ({best_rate:.2%} conversion)")
```

### Winner Selection with Bayesian Approach

```python
import numpy as np

class BayesianMultiVariant:
    """Bayesian multi-variant test"""
    def __init__(self, variants):
        self.variants = variants
        self.data = {v: {'conversions': 0, 'trials': 0} for v in variants}
    
    def update(self, variant, conversions, trials):
        self.data[variant]['conversions'] += conversions
        self.data[variant]['trials'] += trials
    
    def prob_best(self, n_samples=10000):
        """Probability each variant is best"""
        samples = {}
        for variant in self.variants:
            alpha = 1 + self.data[variant]['conversions']
            beta = 1 + (self.data[variant]['trials'] - 
                       self.data[variant]['conversions'])
            samples[variant] = np.random.beta(alpha, beta, n_samples)
        
        # Find which variant is best in each sample
        samples_array = np.array([samples[v] for v in self.variants])
        best_indices = np.argmax(samples_array, axis=0)
        
        # Calculate probabilities
        probs = {}
        for i, variant in enumerate(self.variants):
            probs[variant] = (best_indices == i).mean()
        
        return probs
    
    def expected_loss(self, n_samples=10000):
        """Expected loss for choosing each variant"""
        samples = {}
        for variant in self.variants:
            alpha = 1 + self.data[variant]['conversions']
            beta = 1 + (self.data[variant]['trials'] - 
                       self.data[variant]['conversions'])
            samples[variant] = np.random.beta(alpha, beta, n_samples)
        
        samples_array = np.array([samples[v] for v in self.variants])
        best_in_each_sample = np.max(samples_array, axis=0)
        
        losses = {}
        for i, variant in enumerate(self.variants):
            losses[variant] = np.mean(
                np.maximum(0, best_in_each_sample - samples_array[i])
            )
        
        return losses

# Analyze with Bayesian approach
bayesian_test = BayesianMultiVariant(['A', 'B', 'C', 'D'])

for variant, conv in conversions.items():
    bayesian_test.update(variant, conv.sum(), len(conv))

# Probabilities
probs = bayesian_test.prob_best()
print("\nBayesian Analysis:")
print("Probability each variant is best:")
for variant, prob in sorted(probs.items(), key=lambda x: x[1], reverse=True):
    print(f"  {variant}: {prob:.2%}")

# Expected losses
losses = bayesian_test.expected_loss()
print("\nExpected loss (in conversion rate):")
for variant, loss in sorted(losses.items(), key=lambda x: x[1]):
    print(f"  {variant}: {loss:.6f}")

recommended = min(losses.items(), key=lambda x: x[1])[0]
print(f"\n✓ Recommended variant: {recommended}")
```

## Multivariate (Factorial) Testing

Testing multiple factors and their interactions:

### 2^k Factorial Design

```python
import numpy as np
import pandas as pd
import statsmodels.api as sm
from itertools import product

# Example: Test headline (2 levels) × CTA button (2 levels)
# 2^2 = 4 combinations

np.random.seed(42)
n_per_cell = 500

# Define factor levels
headline = ['Short', 'Long']
cta_button = ['Green', 'Blue']

# Generate all combinations
combinations = list(product(headline, cta_button))

# Simulate data with main effects and interaction
data = []

for h, cta in combinations:
    # Main effects
    headline_effect = 0.02 if h == 'Long' else 0
    cta_effect = 0.03 if cta == 'Green' else 0
    
    # Interaction: Long headline + Green CTA works best
    interaction = 0.04 if (h == 'Long' and cta == 'Green') else 0
    
    # Base rate
    base_rate = 0.10
    conversion_rate = base_rate + headline_effect + cta_effect + interaction
    
    conversions = np.random.binomial(1, conversion_rate, n_per_cell)
    
    for conv in conversions:
        data.append({
            'headline': h,
            'cta_button': cta,
            'converted': conv
        })

df_mvt = pd.DataFrame(data)

# Summary by combination
summary_mvt = df_mvt.groupby(['headline', 'cta_button'])['converted'].agg([
    ('n', 'count'),
    ('conversions', 'sum'),
    ('rate', 'mean')
]).reset_index()

print("="*60)
print("MULTIVARIATE TEST RESULTS")
print("="*60)
print(summary_mvt.to_string(index=False))

# Logistic regression to estimate main effects and interaction
df_mvt['headline_long'] = (df_mvt['headline'] == 'Long').astype(int)
df_mvt['cta_green'] = (df_mvt['cta_button'] == 'Green').astype(int)
df_mvt['interaction'] = df_mvt['headline_long'] * df_mvt['cta_green']

X = sm.add_constant(df_mvt[['headline_long', 'cta_green', 'interaction']])
model = sm.Logit(df_mvt['converted'], X).fit()

print("\n" + "="*60)
print("REGRESSION ANALYSIS")
print("="*60)
print(model.summary())

# Interpret effects
print("\nEffects on log-odds:")
print(f"  Baseline (Short headline, Blue CTA): {model.params['const']:.4f}")
print(f"  Long headline effect: {model.params['headline_long']:.4f}")
print(f"  Green CTA effect: {model.params['cta_green']:.4f}")
print(f"  Interaction (Long × Green): {model.params['interaction']:.4f}")

# Convert to probabilities
from scipy.special import expit

baseline_prob = expit(model.params['const'])
long_green_prob = expit(model.params['const'] + 
                        model.params['headline_long'] + 
                        model.params['cta_green'] + 
                        model.params['interaction'])

print(f"\nPredicted conversion rates:")
print(f"  Short + Blue: {baseline_prob:.2%}")
print(f"  Long + Green: {long_green_prob:.2%}")
print(f"  Lift: {(long_green_prob - baseline_prob)/baseline_prob:.1%}")
```

### 3-Factor Design

```python
# Example: 2×2×2 design (Headline × CTA × Image)
# Total of 8 combinations

def run_factorial_experiment(factors, n_per_cell=500):
    """
    Run factorial experiment with arbitrary number of factors
    
    Parameters:
    -----------
    factors : dict
        {factor_name: [level1, level2, ...], ...}
    """
    import itertools
    
    combinations = list(itertools.product(*factors.values()))
    factor_names = list(factors.keys())
    
    data = []
    
    for combo in combinations:
        # Simulate conversion rate based on combination
        # (In practice, observe real data)
        base_rate = 0.10
        
        # Add some random effect for each level
        combo_effect = np.random.normal(0, 0.02)
        conversion_rate = base_rate + combo_effect
        conversion_rate = np.clip(conversion_rate, 0, 1)
        
        conversions = np.random.binomial(1, conversion_rate, n_per_cell)
        
        for conv in conversions:
            row = dict(zip(factor_names, combo))
            row['converted'] = conv
            data.append(row)
    
    return pd.DataFrame(data)

# 3-factor experiment
factors = {
    'headline': ['Short', 'Long'],
    'cta': ['Green', 'Blue'],
    'image': ['Product', 'Lifestyle']
}

df_3factor = run_factorial_experiment(factors, n_per_cell=300)

# ANOVA
import statsmodels.api as sm
from statsmodels.formula.api import ols

# Fit ANOVA model
model_anova = ols('converted ~ C(headline) * C(cta) * C(image)', 
                  data=df_3factor).fit()
anova_table = sm.stats.anova_lm(model_anova, typ=2)

print("\nANOVA Table:")
print(anova_table)
```

## Fractional Factorial Designs

When testing many factors, use fractional factorial to reduce sample size:

```python
# Example: 5 factors, 2 levels each = 2^5 = 32 combinations
# Use 2^(5-2) = 8 combinations (1/4 fraction)

from pyDOE2 import fracfact

# Define fractional factorial design
design = fracfact("a b c d e")  # 2^5 full factorial

print("Fractional Factorial Design (2^(5-2)):")
print(design[:8])  # Show first 8 runs

# Convert to factor levels
factors = ['Headline', 'CTA', 'Image', 'Copy', 'Layout']
design_df = pd.DataFrame(design[:8], columns=factors)

# -1 = Level 1, +1 = Level 2
design_df = design_df.replace({-1: 'L1', 1: 'L2'})

print("\nExperiment Combinations:")
print(design_df)
```

## Sample Size Inflation

```python
def sample_size_inflation_multivariate(n_factors, levels_per_factor, 
                                       baseline_size):
    """
    Calculate sample size inflation for multivariate test
    
    For k factors with 2 levels each, need 2^k cells
    """
    n_cells = levels_per_factor ** n_factors
    total_sample = baseline_size * n_cells
    
    return {
        'n_cells': n_cells,
        'sample_per_cell': baseline_size,
        'total_sample': total_sample,
        'inflation_factor': n_cells
    }

# 3 factors, 2 levels each
result = sample_size_inflation_multivariate(3, 2, 1000)

print("\nSample Size for 2×2×2 Design:")
print(f"  Number of cells: {result['n_cells']}")
print(f"  Sample per cell: {result['sample_per_cell']}")
print(f"  Total sample needed: {result['total_sample']:,}")
print(f"  Inflation vs simple A/B: {result['inflation_factor']}x")
```

## When to Use Each Approach

| Method | Use When |
|--------|----------|
| **A/B Test** | Single factor, 2 levels, need fastest results |
| **Multi-Variant** | Single factor, 3+ levels, no interactions expected |
| **Full Factorial** | Multiple factors, interactions important, enough traffic |
| **Fractional Factorial** | Many factors, limited traffic, screening experiment |
| **Sequential Testing** | Limited traffic, need to find winner among many |

## Common Pitfalls

### 1. Interaction Neglect

Testing factors separately misses interactions:

```python
# BAD: Sequential A/B tests
# Week 1: Test headline → Choose Long
# Week 2: Test CTA → Choose Green
# Conclusion: Long + Green is best

# PROBLEM: Might miss that Short + Green is actually best
# Need multivariate test to detect interactions
```

### 2. Multiple Comparison Error

```python
# For k variants, number of pairwise comparisons = k(k-1)/2
# With 5 variants: 10 comparisons
# Without correction, false positive rate = 1 - (1-0.05)^10 = 40%!

from statsmodels.stats.multitest import multipletests

# Collect all pairwise p-values
p_values = [...]  # from pairwise tests

# Apply correction
reject, p_corrected, _, _ = multipletests(p_values, alpha=0.05, method='fdr_bh')
```

## Summary

**Multi-Variant Testing:**
- Single factor, multiple levels (A/B/C/D)
- Use ANOVA or chi-square for overall test
- Post-hoc Tukey HSD for pairwise comparisons
- Bayesian approach for winner selection

**Multivariate (Factorial) Testing:**
- Multiple factors simultaneously
- Can detect interactions
- Requires larger sample size (2^k cells)
- Use fractional factorial to reduce sample
- Logistic regression or ANOVA for analysis

**Key Tradeoffs:**
- **Simple A/B**: Fast, small sample, no interactions
- **Multi-variant**: Medium sample, one factor
- **Multivariate**: Large sample, multiple factors, interactions
