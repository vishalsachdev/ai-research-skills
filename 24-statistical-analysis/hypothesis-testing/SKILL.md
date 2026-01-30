---
name: hypothesis-testing
description: Provides expert guidance for conducting statistical hypothesis tests including t-tests, ANOVA, chi-square, and non-parametric tests using scipy and statsmodels for rigorous statistical inference
version: 1.0.0
author: Orchestra Research
license: MIT
tags: [Statistical Analysis, Hypothesis Testing, Scipy, Statsmodels, Inference, T-Test, ANOVA, Chi-Square, Non-Parametric]
dependencies: [scipy>=1.11.0, statsmodels>=0.14.0, numpy>=1.24.0, pandas>=2.0.0]
---

# Hypothesis Testing

Expert-level guidance for conducting statistical hypothesis tests in Python. This skill provides battle-tested patterns for t-tests, ANOVA, chi-square tests, and non-parametric alternatives using scipy and statsmodels.

## When to Use This Skill

Use hypothesis testing when you need to:
- **Compare group means** (t-tests, ANOVA) to determine if differences are statistically significant
- **Test categorical associations** (chi-square, Fisher's exact) between variables
- **Validate assumptions** before applying parametric tests
- **Handle non-normal data** with non-parametric alternatives (Mann-Whitney, Kruskal-Wallis)
- **Report p-values and effect sizes** for scientific publications or business decisions

**Do NOT use hypothesis testing for:**
- Exploratory data analysis without clear hypotheses (use descriptive statistics instead)
- Causal inference without proper experimental design (correlation ≠ causation)
- Multiple comparisons without corrections (inflates Type I error)
- When sample sizes are too small (n < 30 for parametric tests typically)

---

## Core Concepts

### 1. Hypothesis Testing Fundamentals

**Null Hypothesis (H₀)**: The default assumption (e.g., "no difference between groups")
**Alternative Hypothesis (H₁)**: What you're trying to prove (e.g., "groups differ")

**P-value**: Probability of observing results at least as extreme as yours, assuming H₀ is true
- p < 0.05: Reject H₀ (statistically significant)
- p ≥ 0.05: Fail to reject H₀ (not significant)

**Type I Error (α)**: False positive (rejecting true H₀)
**Type II Error (β)**: False negative (failing to reject false H₀)
**Power (1-β)**: Probability of correctly rejecting false H₀

### 2. Test Selection Decision Tree

```
Data Type?
├── Continuous (means)
│   ├── Two groups → Independent t-test (or Mann-Whitney if non-normal)
│   ├── Paired samples → Paired t-test (or Wilcoxon if non-normal)
│   └── 3+ groups → One-way ANOVA (or Kruskal-Wallis if non-normal)
│
└── Categorical (frequencies)
    ├── 2x2 table → Chi-square (or Fisher's exact if n < 5 in cells)
    └── Larger table → Chi-square test of independence
```

---

## Workflow 1: Two-Sample T-Test (Comparing Means)

**Use Case**: Compare mean outcomes between two independent groups (e.g., treatment vs control)

### Checklist

- [ ] Load and prepare data
- [ ] Check normality assumptions (Shapiro-Wilk test)
- [ ] Check variance homogeneity (Levene's test)
- [ ] Perform appropriate t-test (Welch's if variances unequal)
- [ ] Calculate effect size (Cohen's d)
- [ ] Visualize distributions
- [ ] Interpret results with confidence intervals

### Implementation

```python
import numpy as np
import pandas as pd
from scipy import stats
import matplotlib.pyplot as plt
import seaborn as sns

# Step 1: Prepare data
np.random.seed(42)
group_a = np.random.normal(100, 15, 50)  # Control group
group_b = np.random.normal(110, 15, 50)  # Treatment group

df = pd.DataFrame({
    'value': np.concatenate([group_a, group_b]),
    'group': ['A'] * 50 + ['B'] * 50
})

# Step 2: Check normality assumption
shapiro_a = stats.shapiro(group_a)
shapiro_b = stats.shapiro(group_b)

print(f"Normality Tests:")
print(f"  Group A: W={shapiro_a.statistic:.4f}, p={shapiro_a.pvalue:.4f}")
print(f"  Group B: W={shapiro_b.statistic:.4f}, p={shapiro_b.pvalue:.4f}")

# If p > 0.05, data is approximately normal
if shapiro_a.pvalue > 0.05 and shapiro_b.pvalue > 0.05:
    print("✓ Normality assumption satisfied")
else:
    print("✗ Consider non-parametric test (Mann-Whitney)")

# Step 3: Check variance homogeneity
levene_stat, levene_p = stats.levene(group_a, group_b)
print(f"\nLevene's Test: F={levene_stat:.4f}, p={levene_p:.4f}")

# Step 4: Perform t-test
if levene_p > 0.05:
    # Equal variances - use standard t-test
    t_stat, p_value = stats.ttest_ind(group_a, group_b)
    print("\n✓ Using standard t-test (equal variances)")
else:
    # Unequal variances - use Welch's t-test
    t_stat, p_value = stats.ttest_ind(group_a, group_b, equal_var=False)
    print("\n✓ Using Welch's t-test (unequal variances)")

print(f"t-statistic: {t_stat:.4f}")
print(f"p-value: {p_value:.4f}")

# Step 5: Calculate effect size (Cohen's d)
pooled_std = np.sqrt(((len(group_a)-1)*np.std(group_a, ddof=1)**2 + 
                      (len(group_b)-1)*np.std(group_b, ddof=1)**2) / 
                     (len(group_a) + len(group_b) - 2))
cohens_d = (np.mean(group_b) - np.mean(group_a)) / pooled_std

print(f"\nEffect Size (Cohen's d): {cohens_d:.4f}")
print(f"Interpretation: ", end="")
if abs(cohens_d) < 0.2:
    print("Negligible")
elif abs(cohens_d) < 0.5:
    print("Small")
elif abs(cohens_d) < 0.8:
    print("Medium")
else:
    print("Large")

# Step 6: Calculate confidence intervals
ci_a = stats.t.interval(0.95, len(group_a)-1, 
                        loc=np.mean(group_a), 
                        scale=stats.sem(group_a))
ci_b = stats.t.interval(0.95, len(group_b)-1, 
                        loc=np.mean(group_b), 
                        scale=stats.sem(group_b))

print(f"\n95% Confidence Intervals:")
print(f"  Group A: [{ci_a[0]:.2f}, {ci_a[1]:.2f}]")
print(f"  Group B: [{ci_b[0]:.2f}, {ci_b[1]:.2f}]")

# Step 7: Visualize
fig, axes = plt.subplots(1, 2, figsize=(12, 5))

# Violin plot
sns.violinplot(data=df, x='group', y='value', ax=axes[0])
axes[0].set_title('Distribution Comparison')
axes[0].set_ylabel('Value')

# Box plot with points
sns.boxplot(data=df, x='group', y='value', ax=axes[1])
sns.stripplot(data=df, x='group', y='value', alpha=0.3, ax=axes[1])
axes[1].set_title(f't-test: p={p_value:.4f}')
axes[1].set_ylabel('Value')

plt.tight_layout()
plt.savefig('ttest_results.png', dpi=300, bbox_inches='tight')
print("\n✓ Visualization saved to ttest_results.png")
```

---

## Workflow 2: One-Way ANOVA (Comparing 3+ Groups)

**Use Case**: Compare means across three or more independent groups

### Checklist

- [ ] Load data with 3+ groups
- [ ] Check normality for each group
- [ ] Check variance homogeneity (Bartlett or Levene test)
- [ ] Perform one-way ANOVA
- [ ] If significant, perform post-hoc tests (Tukey HSD)
- [ ] Calculate effect size (eta-squared)
- [ ] Visualize group differences

### Implementation

```python
from scipy import stats
from statsmodels.stats.multicomp import pairwise_tukeyhsd
import pandas as pd
import numpy as np

# Step 1: Prepare data (3 groups)
np.random.seed(42)
group_1 = np.random.normal(100, 15, 30)
group_2 = np.random.normal(110, 15, 30)
group_3 = np.random.normal(105, 15, 30)

df = pd.DataFrame({
    'value': np.concatenate([group_1, group_2, group_3]),
    'group': ['A']*30 + ['B']*30 + ['C']*30
})

# Step 2: Check assumptions
print("Normality Tests:")
for group_name in ['A', 'B', 'C']:
    group_data = df[df['group'] == group_name]['value']
    stat, p = stats.shapiro(group_data)
    print(f"  Group {group_name}: p={p:.4f}")

# Variance homogeneity
stat, p = stats.levene(group_1, group_2, group_3)
print(f"\nLevene's Test: p={p:.4f}")

# Step 3: Perform one-way ANOVA
f_stat, p_value = stats.f_oneway(group_1, group_2, group_3)
print(f"\nOne-Way ANOVA:")
print(f"  F-statistic: {f_stat:.4f}")
print(f"  p-value: {p_value:.4f}")

# Step 4: Calculate effect size (eta-squared)
# Total sum of squares
grand_mean = df['value'].mean()
ss_total = np.sum((df['value'] - grand_mean)**2)

# Between-group sum of squares
group_means = df.groupby('group')['value'].mean()
group_sizes = df.groupby('group').size()
ss_between = np.sum(group_sizes * (group_means - grand_mean)**2)

eta_squared = ss_between / ss_total
print(f"\nEffect Size (η²): {eta_squared:.4f}")

# Step 5: Post-hoc tests (if ANOVA is significant)
if p_value < 0.05:
    print("\n✓ ANOVA significant - performing Tukey HSD post-hoc test")
    tukey = pairwise_tukeyhsd(endog=df['value'], groups=df['group'], alpha=0.05)
    print("\n" + str(tukey))
else:
    print("\n✗ ANOVA not significant - no post-hoc tests needed")

# Visualization
import matplotlib.pyplot as plt
import seaborn as sns

fig, ax = plt.subplots(figsize=(10, 6))
sns.boxplot(data=df, x='group', y='value', ax=ax)
sns.stripplot(data=df, x='group', y='value', alpha=0.4, color='black', ax=ax)
ax.set_title(f'One-Way ANOVA: F={f_stat:.2f}, p={p_value:.4f}')
ax.set_ylabel('Value')
plt.tight_layout()
plt.savefig('anova_results.png', dpi=300, bbox_inches='tight')
```

---

## Workflow 3: Chi-Square Test (Categorical Data)

**Use Case**: Test association between two categorical variables

### Checklist

- [ ] Create contingency table
- [ ] Check expected frequencies (all cells ≥ 5)
- [ ] Perform chi-square test of independence
- [ ] Calculate effect size (Cramér's V)
- [ ] Examine standardized residuals for patterns
- [ ] If 2x2 table with small n, use Fisher's exact test

### Implementation

```python
import numpy as np
import pandas as pd
from scipy import stats
from scipy.stats.contingency import association

# Step 1: Create contingency table
data = {
    'Treatment': ['A', 'A', 'A', 'B', 'B', 'B', 'C', 'C', 'C'],
    'Outcome': ['Success', 'Failure', 'Success', 'Success', 'Success', 'Failure', 
                'Failure', 'Failure', 'Success']
}

df = pd.DataFrame({
    'Treatment': ['A']*60 + ['B']*60 + ['C']*60,
    'Outcome': ['Success']*40 + ['Failure']*20 +  # Treatment A
               ['Success']*50 + ['Failure']*10 +  # Treatment B
               ['Success']*30 + ['Failure']*30    # Treatment C
})

contingency_table = pd.crosstab(df['Treatment'], df['Outcome'])
print("Contingency Table:")
print(contingency_table)

# Step 2: Check expected frequencies
chi2, p, dof, expected = stats.chi2_contingency(contingency_table)
print("\nExpected Frequencies:")
print(pd.DataFrame(expected, 
                   index=contingency_table.index, 
                   columns=contingency_table.columns))

# Check if all expected frequencies ≥ 5
min_expected = expected.min()
print(f"\nMinimum expected frequency: {min_expected:.2f}")
if min_expected >= 5:
    print("✓ Chi-square test appropriate")
else:
    print("✗ Consider Fisher's exact test (expected frequencies < 5)")

# Step 3: Perform chi-square test
print(f"\nChi-Square Test of Independence:")
print(f"  χ² statistic: {chi2:.4f}")
print(f"  p-value: {p:.4f}")
print(f"  Degrees of freedom: {dof}")

# Step 4: Calculate effect size (Cramér's V)
n = contingency_table.sum().sum()
min_dim = min(contingency_table.shape[0], contingency_table.shape[1]) - 1
cramers_v = np.sqrt(chi2 / (n * min_dim))

print(f"\nEffect Size (Cramér's V): {cramers_v:.4f}")
print(f"Interpretation: ", end="")
if cramers_v < 0.1:
    print("Negligible")
elif cramers_v < 0.3:
    print("Small")
elif cramers_v < 0.5:
    print("Medium")
else:
    print("Large")

# Step 5: Standardized residuals (identify which cells contribute most)
observed = contingency_table.values
residuals = (observed - expected) / np.sqrt(expected)
print("\nStandardized Residuals:")
print(pd.DataFrame(residuals, 
                   index=contingency_table.index,
                   columns=contingency_table.columns))
print("(|residual| > 2 indicates significant deviation)")

# Visualization
import matplotlib.pyplot as plt
import seaborn as sns

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Stacked bar chart
contingency_table.plot(kind='bar', stacked=True, ax=axes[0])
axes[0].set_title('Observed Frequencies')
axes[0].set_ylabel('Count')
axes[0].legend(title='Outcome')

# Heatmap of standardized residuals
sns.heatmap(residuals, annot=True, fmt='.2f', cmap='RdBu_r', 
            center=0, ax=axes[1], cbar_kws={'label': 'Standardized Residual'})
axes[1].set_title(f'Chi-Square Test: p={p:.4f}')

plt.tight_layout()
plt.savefig('chisquare_results.png', dpi=300, bbox_inches='tight')
```

---

## Workflow 4: Non-Parametric Tests (When Assumptions Violated)

**Use Case**: Compare groups when data is not normally distributed

### Implementation

```python
from scipy import stats
import numpy as np

# Example: Right-skewed data
np.random.seed(42)
group_a = np.random.exponential(scale=2.0, size=40)
group_b = np.random.exponential(scale=2.5, size=40)

# Mann-Whitney U test (alternative to independent t-test)
u_stat, p_value = stats.mannwhitneyu(group_a, group_b, alternative='two-sided')
print("Mann-Whitney U Test:")
print(f"  U-statistic: {u_stat:.4f}")
print(f"  p-value: {p_value:.4f}")

# Wilcoxon signed-rank test (alternative to paired t-test)
# For paired samples
differences = group_b - group_a
stat, p = stats.wilcoxon(differences)
print(f"\nWilcoxon Signed-Rank Test:")
print(f"  Statistic: {stat:.4f}")
print(f"  p-value: {p:.4f}")

# Kruskal-Wallis H-test (alternative to one-way ANOVA)
group_c = np.random.exponential(scale=3.0, size=40)
h_stat, p_kw = stats.kruskal(group_a, group_b, group_c)
print(f"\nKruskal-Wallis H-Test:")
print(f"  H-statistic: {h_stat:.4f}")
print(f"  p-value: {p_kw:.4f}")
```

---

## Common Issues and Solutions

### Issue 1: Multiple Comparisons Inflating Type I Error

**Problem**: Testing multiple hypotheses increases false positive rate

**Solution**: Apply correction methods

```python
from statsmodels.stats.multitest import multipletests

# Multiple p-values from different tests
p_values = [0.01, 0.04, 0.03, 0.12, 0.002]

# Bonferroni correction (conservative)
reject_bonf, pvals_bonf, _, _ = multipletests(p_values, alpha=0.05, method='bonferroni')

# Benjamini-Hochberg (FDR control, less conservative)
reject_fdr, pvals_fdr, _, _ = multipletests(p_values, alpha=0.05, method='fdr_bh')

print("Original p-values:", p_values)
print("Bonferroni adjusted:", pvals_bonf)
print("FDR adjusted:", pvals_fdr)
print("\nSignificant (Bonferroni):", reject_bonf)
print("Significant (FDR):", reject_fdr)
```

### Issue 2: Small Sample Sizes

**Problem**: Parametric tests require n ≥ 30 for reliable results

**Solution**: Use exact tests or bootstrap methods

```python
from scipy import stats

# For 2x2 contingency tables with small n
table = [[8, 2], [1, 5]]
odds_ratio, p_value = stats.fisher_exact(table)
print(f"Fisher's Exact Test p-value: {p_value:.4f}")

# Bootstrap confidence intervals for small samples
from scipy.stats import bootstrap

data = np.array([23, 25, 28, 22, 24, 26, 29])
rng = np.random.default_rng(42)
res = bootstrap((data,), np.mean, confidence_level=0.95, 
                random_state=rng, n_resamples=10000)
print(f"Bootstrap 95% CI: [{res.confidence_interval.low:.2f}, {res.confidence_interval.high:.2f}]")
```

### Issue 3: Assumption Violations

**Problem**: Data doesn't meet normality or variance assumptions

**Solution**: Transform data or use non-parametric alternatives

```python
import numpy as np
from scipy import stats

# Right-skewed data
data = np.random.exponential(2, 100)

# Check normality
_, p_before = stats.shapiro(data)
print(f"Normality before transform: p={p_before:.4f}")

# Log transformation
data_log = np.log(data)
_, p_after = stats.shapiro(data_log)
print(f"Normality after log transform: p={p_after:.4f}")

# If transformation doesn't work, use non-parametric test
if p_after < 0.05:
    print("→ Use Mann-Whitney U test instead of t-test")
```

### Issue 4: Unequal Sample Sizes

**Problem**: ANOVA assumes equal group sizes for best power

**Solution**: Use Welch's ANOVA for unequal variances and sizes

```python
from scipy import stats

group_a = np.random.normal(100, 15, 25)
group_b = np.random.normal(110, 20, 50)
group_c = np.random.normal(105, 10, 35)

# Welch's ANOVA (doesn't assume equal variances)
# Note: scipy doesn't have built-in Welch ANOVA, use alternative:
# For two groups: use Welch's t-test
stat, p = stats.ttest_ind(group_a, group_b, equal_var=False)
print(f"Welch's t-test: p={p:.4f}")

# For 3+ groups with unequal variances, use Kruskal-Wallis
h, p = stats.kruskal(group_a, group_b, group_c)
print(f"Kruskal-Wallis (robust to unequal variances): p={p:.4f}")
```

---

## When to Use vs Alternatives

| Use Hypothesis Testing | Use Alternative |
|------------------------|-----------------|
| Confirmatory analysis with clear hypotheses | Exploratory data analysis → Use descriptive stats, visualization |
| Comparing specific groups or variables | Building predictive models → Use regression/ML |
| Need statistical significance for publication | Estimating effect sizes → Use confidence intervals |
| Testing assumptions or validating theories | Causal inference from observational data → Use causal inference methods |
| Simple group comparisons | Complex multi-variable relationships → Use regression analysis |

---

## Advanced Features

**Power Analysis**: See [references/power-analysis.md](references/power-analysis.md)
**Bayesian Hypothesis Testing**: See [references/bayesian-tests.md](references/bayesian-tests.md)
**Robust Statistical Tests**: See [references/robust-tests.md](references/robust-tests.md)

---

## Quick Reference

### Test Selection Guide

```python
# Two groups (continuous outcome)
stats.ttest_ind(group1, group2)  # Independent samples
stats.ttest_rel(before, after)   # Paired samples
stats.mannwhitneyu(group1, group2)  # Non-parametric

# 3+ groups (continuous outcome)
stats.f_oneway(group1, group2, group3)  # ANOVA
stats.kruskal(group1, group2, group3)   # Non-parametric

# Categorical data
stats.chi2_contingency(contingency_table)  # Chi-square
stats.fisher_exact(table_2x2)              # Fisher's exact

# Normality test
stats.shapiro(data)  # Shapiro-Wilk

# Variance homogeneity
stats.levene(group1, group2, group3)  # Levene's test
stats.bartlett(group1, group2, group3)  # Bartlett's test (sensitive to non-normality)
```

---

## Summary

This skill provides production-ready workflows for statistical hypothesis testing:
- **T-tests and ANOVA** for comparing group means
- **Chi-square tests** for categorical associations
- **Non-parametric alternatives** when assumptions violated
- **Effect sizes and confidence intervals** for proper interpretation
- **Multiple comparison corrections** to control false positives

**Key Principle**: Always check assumptions before selecting a test, and report both statistical significance (p-value) and practical significance (effect size).
