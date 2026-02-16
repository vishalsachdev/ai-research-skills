# Robust Statistical Tests

## Overview

Robust tests maintain validity when assumptions (normality, homogeneity of variance) are violated. They provide reliable inference even with outliers, heavy-tailed distributions, or non-normal data.

## When to Use Robust Tests

**Use robust methods when:**
- Data contains outliers that shouldn't be removed
- Distributions are heavy-tailed or skewed
- Sample sizes are small (n < 30)
- Variance heterogeneity exists
- You want inference that generalizes better

## Robust Location Tests

### Trimmed Means

Remove extreme values before calculating mean:

```python
from scipy import stats
import numpy as np

# Data with outliers
np.random.seed(42)
data = np.concatenate([
    np.random.normal(100, 15, 45),
    [200, 210]  # Outliers
])

# Regular mean (influenced by outliers)
regular_mean = np.mean(data)

# Trimmed mean (10% trimming on each side)
trimmed_mean = stats.trim_mean(data, proportiontocut=0.1)

# Median (50% trimming, most robust)
median = np.median(data)

print(f"Regular mean: {regular_mean:.2f}")
print(f"10% trimmed mean: {trimmed_mean:.2f}")
print(f"Median: {median:.2f}")
```

### Yuen's Test (Robust T-Test)

T-test using trimmed means:

```python
from scipy import stats
import numpy as np

def yuens_test(group1, group2, trim_percent=0.2):
    """
    Yuen's test for trimmed means
    
    Parameters:
    -----------
    trim_percent : float
        Proportion to trim from each end (0.2 = 20% from each end)
    """
    # Trim data
    n1, n2 = len(group1), len(group2)
    g1_trim = int(n1 * trim_percent)
    g2_trim = int(n2 * trim_percent)
    
    g1_sorted = np.sort(group1)[g1_trim:n1-g1_trim]
    g2_sorted = np.sort(group2)[g2_trim:n2-g2_trim]
    
    # Trimmed means
    mean1 = np.mean(g1_sorted)
    mean2 = np.mean(g2_sorted)
    
    # Winsorized variances
    def winsorized_var(data, trim_n):
        sorted_data = np.sort(data)
        winsorized = sorted_data.copy()
        winsorized[:trim_n] = sorted_data[trim_n]
        winsorized[-trim_n:] = sorted_data[-trim_n-1]
        return np.var(winsorized, ddof=1)
    
    var1 = winsorized_var(group1, g1_trim)
    var2 = winsorized_var(group2, g2_trim)
    
    # Effective sample sizes
    h1 = len(g1_sorted)
    h2 = len(g2_sorted)
    
    # Test statistic
    se_diff = np.sqrt(var1/h1 + var2/h2)
    t_stat = (mean1 - mean2) / se_diff
    
    # Degrees of freedom (Welch-Satterthwaite)
    df = (var1/h1 + var2/h2)**2 / (
        (var1/h1)**2/(h1-1) + (var2/h2)**2/(h2-1)
    )
    
    p_value = 2 * (1 - stats.t.cdf(abs(t_stat), df))
    
    return t_stat, p_value, mean1, mean2

# Example with outliers
np.random.seed(42)
group_a = np.concatenate([np.random.normal(100, 15, 45), [200]])
group_b = np.concatenate([np.random.normal(110, 15, 45), [90]])

# Standard t-test
t_standard, p_standard = stats.ttest_ind(group_a, group_b)

# Yuen's robust test
t_yuen, p_yuen, mean_a, mean_b = yuens_test(group_a, group_b, trim_percent=0.2)

print(f"Standard t-test: t={t_standard:.4f}, p={p_standard:.4f}")
print(f"Yuen's test (20% trim): t={t_yuen:.4f}, p={p_yuen:.4f}")
print(f"  Trimmed means: {mean_a:.2f} vs {mean_b:.2f}")
```

## Bootstrapping

Generate empirical sampling distributions without assumptions:

```python
from scipy import stats
import numpy as np

def bootstrap_ttest(group1, group2, n_bootstrap=10000, alpha=0.05):
    """
    Bootstrap two-sample t-test
    """
    # Observed difference
    observed_diff = np.mean(group2) - np.mean(group1)
    
    # Bootstrap distribution under null (no difference)
    combined = np.concatenate([group1, group2])
    n1, n2 = len(group1), len(group2)
    
    bootstrap_diffs = []
    rng = np.random.default_rng(42)
    
    for _ in range(n_bootstrap):
        # Resample
        perm = rng.permutation(combined)
        boot_g1 = perm[:n1]
        boot_g2 = perm[n1:]
        
        # Calculate difference
        diff = np.mean(boot_g2) - np.mean(boot_g1)
        bootstrap_diffs.append(diff)
    
    bootstrap_diffs = np.array(bootstrap_diffs)
    
    # P-value (two-tailed)
    p_value = np.mean(np.abs(bootstrap_diffs) >= np.abs(observed_diff))
    
    # Confidence interval
    ci_lower = np.percentile(bootstrap_diffs, 100 * alpha/2)
    ci_upper = np.percentile(bootstrap_diffs, 100 * (1 - alpha/2))
    
    return observed_diff, p_value, (ci_lower, ci_upper)

# Example
np.random.seed(42)
group_a = np.random.exponential(2, 40)  # Non-normal
group_b = np.random.exponential(2.5, 40)

diff, p_boot, ci = bootstrap_ttest(group_a, group_b)

print(f"Bootstrap test:")
print(f"  Observed difference: {diff:.4f}")
print(f"  p-value: {p_boot:.4f}")
print(f"  95% CI: [{ci[0]:.4f}, {ci[1]:.4f}]")
```

## Permutation Tests

Exact non-parametric tests:

```python
from scipy import stats
import numpy as np

def permutation_test(group1, group2, n_permutations=10000):
    """
    Permutation test for difference in means
    """
    # Observed test statistic
    observed_stat = abs(np.mean(group2) - np.mean(group1))
    
    # Combine groups
    combined = np.concatenate([group1, group2])
    n1 = len(group1)
    
    # Generate null distribution
    rng = np.random.default_rng(42)
    perm_stats = []
    
    for _ in range(n_permutations):
        # Randomly assign to groups
        perm = rng.permutation(combined)
        perm_g1 = perm[:n1]
        perm_g2 = perm[n1:]
        
        # Calculate statistic
        stat = abs(np.mean(perm_g2) - np.mean(perm_g1))
        perm_stats.append(stat)
    
    perm_stats = np.array(perm_stats)
    
    # P-value
    p_value = np.mean(perm_stats >= observed_stat)
    
    return observed_stat, p_value, perm_stats

# Example
np.random.seed(42)
group_a = np.random.normal(100, 15, 30)
group_b = np.random.normal(110, 15, 30)

stat, p_perm, null_dist = permutation_test(group_a, group_b)

print(f"Permutation test:")
print(f"  Test statistic: {stat:.4f}")
print(f"  p-value: {p_perm:.4f}")

# Visualize null distribution
import matplotlib.pyplot as plt

plt.figure(figsize=(10, 6))
plt.hist(null_dist, bins=50, alpha=0.7, edgecolor='black')
plt.axvline(stat, color='red', linestyle='--', linewidth=2, 
            label=f'Observed ({stat:.2f})')
plt.xlabel('Test Statistic (Absolute Difference in Means)')
plt.ylabel('Frequency')
plt.title(f'Permutation Test Null Distribution (p={p_perm:.4f})')
plt.legend()
plt.grid(True, alpha=0.3)
plt.savefig('permutation_test.png', dpi=300, bbox_inches='tight')
```

## Welch's T-Test (Unequal Variances)

More robust than Student's t-test:

```python
from scipy import stats
import numpy as np

# Groups with unequal variances
np.random.seed(42)
group_a = np.random.normal(100, 10, 50)  # Small variance
group_b = np.random.normal(110, 25, 50)  # Large variance

# Student's t-test (assumes equal variances)
t_student, p_student = stats.ttest_ind(group_a, group_b, equal_var=True)

# Welch's t-test (allows unequal variances)
t_welch, p_welch = stats.ttest_ind(group_a, group_b, equal_var=False)

print(f"Student's t-test: t={t_student:.4f}, p={p_student:.4f}")
print(f"Welch's t-test: t={t_welch:.4f}, p={p_welch:.4f}")

# Test variance equality
levene_stat, levene_p = stats.levene(group_a, group_b)
print(f"\nLevene's test: p={levene_p:.4f}")

if levene_p < 0.05:
    print("→ Use Welch's t-test (variances are unequal)")
else:
    print("→ Either test appropriate (variances are equal)")
```

## Brunner-Munzel Test

More powerful alternative to Mann-Whitney U:

```python
from scipy import stats
import numpy as np

# Ordinal or non-normal data
np.random.seed(42)
group_a = np.random.exponential(2, 40)
group_b = np.random.exponential(2.5, 40)

# Mann-Whitney U (ranks only, assumes similar distributions)
u_stat, p_mw = stats.mannwhitneyu(group_a, group_b, alternative='two-sided')

# Brunner-Munzel (more general, doesn't assume distribution shape)
bm_stat, p_bm = stats.brunnermunzel(group_a, group_b)

print(f"Mann-Whitney U: U={u_stat:.4f}, p={p_mw:.4f}")
print(f"Brunner-Munzel: W={bm_stat:.4f}, p={p_bm:.4f}")
```

## Robust ANOVA Alternatives

### Kruskal-Wallis (Rank-based)

```python
from scipy import stats
import numpy as np

# 3 groups with outliers
np.random.seed(42)
group_a = np.concatenate([np.random.normal(100, 15, 28), [200, 210]])
group_b = np.concatenate([np.random.normal(110, 15, 28), [90, 80]])
group_c = np.random.normal(105, 15, 30)

# Standard ANOVA
f_stat, p_anova = stats.f_oneway(group_a, group_b, group_c)

# Kruskal-Wallis (rank-based, robust to outliers)
h_stat, p_kw = stats.kruskal(group_a, group_b, group_c)

print(f"ANOVA: F={f_stat:.4f}, p={p_anova:.4f}")
print(f"Kruskal-Wallis: H={h_stat:.4f}, p={p_kw:.4f}")
```

### Welch's ANOVA (Unequal Variances)

```python
import numpy as np
from scipy import stats

def welch_anova(*groups):
    """
    Welch's ANOVA for groups with unequal variances
    """
    k = len(groups)  # Number of groups
    n_i = np.array([len(group) for group in groups])
    mean_i = np.array([np.mean(group) for group in groups])
    var_i = np.array([np.var(group, ddof=1) for group in groups])
    
    # Weights
    w_i = n_i / var_i
    
    # Grand mean (weighted)
    grand_mean = np.sum(w_i * mean_i) / np.sum(w_i)
    
    # Test statistic
    f_numerator = np.sum(w_i * (mean_i - grand_mean)**2) / (k - 1)
    
    # Denominator correction
    lambda_val = 3 * np.sum((1 - w_i/np.sum(w_i))**2 / (n_i - 1)) / (k**2 - 1)
    f_denominator = 1 + (2 * (k - 2) * lambda_val) / (k**2 - 1)
    
    f_stat = f_numerator / f_denominator
    
    # Degrees of freedom
    df1 = k - 1
    df2 = 1 / (3 * lambda_val)
    
    p_value = 1 - stats.f.cdf(f_stat, df1, df2)
    
    return f_stat, p_value, df1, df2

# Example with unequal variances
np.random.seed(42)
group_a = np.random.normal(100, 10, 30)
group_b = np.random.normal(110, 25, 30)
group_c = np.random.normal(105, 15, 30)

f_welch, p_welch, df1, df2 = welch_anova(group_a, group_b, group_c)

print(f"Welch's ANOVA: F({df1:.0f}, {df2:.1f}) = {f_welch:.4f}, p={p_welch:.4f}")
```

## M-Estimators

Robust regression-style estimators:

```python
import numpy as np
from scipy import stats
from scipy.optimize import minimize

def huber_mean(data, k=1.345):
    """
    Huber M-estimator of location
    
    Parameters:
    -----------
    k : float
        Tuning constant (1.345 for 95% efficiency at normal)
    """
    def huber_loss(mu, x, k):
        residuals = x - mu
        loss = np.where(
            np.abs(residuals) <= k,
            0.5 * residuals**2,
            k * np.abs(residuals) - 0.5 * k**2
        )
        return np.sum(loss)
    
    result = minimize(huber_loss, x0=np.median(data), 
                     args=(data, k), method='BFGS')
    return result.x[0]

# Data with outliers
np.random.seed(42)
data = np.concatenate([
    np.random.normal(100, 15, 45),
    [200, 210, 220]  # Outliers
])

regular_mean = np.mean(data)
median = np.median(data)
huber = huber_mean(data)

print(f"Regular mean: {regular_mean:.2f}")
print(f"Median: {median:.2f}")
print(f"Huber M-estimator: {huber:.2f}")
```

## Winsorization

Replace extreme values with less extreme values:

```python
from scipy.stats.mstats import winsorize
import numpy as np

# Data with outliers
np.random.seed(42)
data = np.concatenate([
    np.random.normal(100, 15, 47),
    [200, 210, 220]
])

# Winsorize at 5% on each tail
winsorized_data = winsorize(data, limits=[0.05, 0.05])

print(f"Original mean: {np.mean(data):.2f}")
print(f"Winsorized mean: {np.mean(winsorized_data):.2f}")
print(f"\nOriginal range: [{data.min():.2f}, {data.max():.2f}]")
print(f"Winsorized range: [{winsorized_data.min():.2f}, {winsorized_data.max():.2f}]")
```

## Robust Effect Sizes

### Probability of Superiority

```python
import numpy as np

def prob_superiority(group1, group2):
    """
    Probability that random value from group2 > random value from group1
    Robust alternative to Cohen's d
    """
    n1, n2 = len(group1), len(group2)
    
    # Count pairs where group2 > group1
    count = 0
    for x1 in group1:
        for x2 in group2:
            if x2 > x1:
                count += 1
            elif x2 == x1:
                count += 0.5
    
    ps = count / (n1 * n2)
    return ps

# Example
np.random.seed(42)
group_a = np.random.exponential(2, 40)
group_b = np.random.exponential(2.5, 40)

ps = prob_superiority(group_a, group_b)
print(f"Probability of Superiority: {ps:.4f}")
print(f"Interpretation: {ps*100:.1f}% chance random value from B > A")
```

## Summary: Choosing Robust Tests

| Scenario | Robust Alternative |
|----------|-------------------|
| T-test with outliers | Yuen's test, permutation test |
| T-test, unequal variances | Welch's t-test |
| T-test, non-normal | Mann-Whitney U, Brunner-Munzel |
| ANOVA with outliers | Kruskal-Wallis |
| ANOVA, unequal variances | Welch's ANOVA |
| Small sample CI | Bootstrap confidence intervals |
| Effect size with outliers | Probability of superiority |

**General principle**: Robust tests sacrifice a small amount of power under ideal conditions for much better performance when assumptions are violated.
