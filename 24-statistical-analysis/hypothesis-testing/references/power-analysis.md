# Power Analysis and Sample Size Calculation

## Overview

Statistical power is the probability of correctly rejecting a false null hypothesis (1 - β). Power analysis helps determine the required sample size for detecting an effect of a given size, or estimate the detectable effect size given a sample.

## Key Concepts

**Statistical Power Components:**
- **Alpha (α)**: Significance level (typically 0.05)
- **Beta (β)**: Type II error rate (typically 0.20)
- **Power (1-β)**: Probability of detecting true effect (typically 0.80)
- **Effect Size**: Magnitude of difference you want to detect
- **Sample Size**: Number of observations needed

## Using statsmodels for Power Analysis

### T-Test Power Analysis

```python
from statsmodels.stats.power import TTestIndPower

# Calculate required sample size
power_analysis = TTestIndPower()

# For Cohen's d = 0.5, alpha = 0.05, power = 0.80
sample_size = power_analysis.solve_power(
    effect_size=0.5,
    alpha=0.05,
    power=0.80,
    ratio=1.0,  # Equal group sizes
    alternative='two-sided'
)

print(f"Required sample size per group: {sample_size:.0f}")

# Calculate achievable power with given sample size
achieved_power = power_analysis.solve_power(
    effect_size=0.5,
    alpha=0.05,
    nobs1=50,  # Sample size per group
    ratio=1.0,
    alternative='two-sided'
)

print(f"Achieved power with n=50: {achieved_power:.4f}")

# Calculate detectable effect size
detectable_effect = power_analysis.solve_power(
    nobs1=50,
    alpha=0.05,
    power=0.80,
    ratio=1.0,
    alternative='two-sided'
)

print(f"Detectable effect size: {detectable_effect:.4f}")
```

### ANOVA Power Analysis

```python
from statsmodels.stats.power import FTestAnovaPower

power_analysis = FTestAnovaPower()

# Calculate sample size for ANOVA with 3 groups
# Effect size f = 0.25 (medium effect)
sample_size = power_analysis.solve_power(
    effect_size=0.25,
    alpha=0.05,
    power=0.80,
    k_groups=3
)

print(f"Required sample size per group: {sample_size:.0f}")
```

### Chi-Square Power Analysis

```python
from statsmodels.stats.power import GofChisquarePower

power_analysis = GofChisquarePower()

# For chi-square goodness-of-fit test
sample_size = power_analysis.solve_power(
    effect_size=0.3,  # w (effect size measure)
    alpha=0.05,
    power=0.80,
    n_bins=4  # Number of categories
)

print(f"Required total sample size: {sample_size:.0f}")
```

## Effect Size Conventions

### Cohen's d (T-tests)
- Small: 0.2
- Medium: 0.5
- Large: 0.8

### Eta-squared (ANOVA)
- Small: 0.01
- Medium: 0.06
- Large: 0.14

### Cramér's V (Chi-square)
- Small: 0.1
- Medium: 0.3
- Large: 0.5

## Power Curves

Visualize how power changes with sample size:

```python
import numpy as np
import matplotlib.pyplot as plt
from statsmodels.stats.power import TTestIndPower

power_analysis = TTestIndPower()
sample_sizes = np.arange(10, 200, 5)
effect_sizes = [0.2, 0.5, 0.8]

plt.figure(figsize=(10, 6))

for effect_size in effect_sizes:
    powers = [power_analysis.solve_power(
        effect_size=effect_size,
        nobs1=n,
        alpha=0.05,
        ratio=1.0,
        alternative='two-sided'
    ) for n in sample_sizes]
    
    plt.plot(sample_sizes, powers, label=f'd={effect_size}')

plt.axhline(y=0.80, color='r', linestyle='--', label='Target Power (0.80)')
plt.xlabel('Sample Size per Group')
plt.ylabel('Statistical Power')
plt.title('Power Curves for Independent T-Test')
plt.legend()
plt.grid(True, alpha=0.3)
plt.savefig('power_curve.png', dpi=300, bbox_inches='tight')
```

## Minimum Detectable Effect (MDE)

Calculate the smallest effect you can reliably detect:

```python
from statsmodels.stats.power import TTestIndPower

def calculate_mde(n_per_group, alpha=0.05, power=0.80):
    """Calculate minimum detectable effect size"""
    power_analysis = TTestIndPower()
    mde = power_analysis.solve_power(
        nobs1=n_per_group,
        alpha=alpha,
        power=power,
        ratio=1.0,
        alternative='two-sided'
    )
    return mde

# Example: with 100 samples per group
mde = calculate_mde(100)
print(f"MDE with n=100: Cohen's d = {mde:.4f}")

# Convert to percentage change
# If baseline mean = 100, sd = 15
baseline_mean = 100
baseline_std = 15
absolute_change = mde * baseline_std
percentage_change = (absolute_change / baseline_mean) * 100

print(f"This translates to a {percentage_change:.1f}% change")
```

## Sample Size for Equivalence Tests

When testing that two treatments are equivalent (not different):

```python
from statsmodels.stats.power import tt_ind_solve_power

# Equivalence margin (e.g., ±10% difference is considered equivalent)
equivalence_margin = 0.3  # Cohen's d units

# TOST (Two One-Sided Tests) requires more power
sample_size = tt_ind_solve_power(
    effect_size=equivalence_margin,
    alpha=0.05,
    power=0.80,
    ratio=1.0,
    alternative='two-sided'
)

# For TOST, use alpha/2
tost_sample_size = tt_ind_solve_power(
    effect_size=equivalence_margin,
    alpha=0.025,  # More stringent
    power=0.80,
    ratio=1.0,
    alternative='two-sided'
)

print(f"Sample size for superiority test: {sample_size:.0f}")
print(f"Sample size for equivalence test: {tost_sample_size:.0f}")
```

## Unequal Group Sizes

```python
from statsmodels.stats.power import TTestIndPower

power_analysis = TTestIndPower()

# If one group is 2x larger than the other
sample_size_small = power_analysis.solve_power(
    effect_size=0.5,
    alpha=0.05,
    power=0.80,
    ratio=2.0,  # Large group is 2x small group
    alternative='two-sided'
)

sample_size_large = sample_size_small * 2

print(f"Small group: {sample_size_small:.0f}")
print(f"Large group: {sample_size_large:.0f}")
print(f"Total: {sample_size_small + sample_size_large:.0f}")
```

## Post-Hoc Power Analysis

Calculate achieved power after study completion:

```python
from statsmodels.stats.power import TTestIndPower
import numpy as np

# Observed data
group_a = np.array([...])  # Your actual data
group_b = np.array([...])

# Calculate observed effect size
pooled_std = np.sqrt(((len(group_a)-1)*np.std(group_a, ddof=1)**2 + 
                      (len(group_b)-1)*np.std(group_b, ddof=1)**2) / 
                     (len(group_a) + len(group_b) - 2))
observed_d = (np.mean(group_b) - np.mean(group_a)) / pooled_std

# Calculate achieved power
power_analysis = TTestIndPower()
achieved_power = power_analysis.solve_power(
    effect_size=observed_d,
    nobs1=len(group_a),
    alpha=0.05,
    ratio=len(group_b)/len(group_a),
    alternative='two-sided'
)

print(f"Observed Cohen's d: {observed_d:.4f}")
print(f"Achieved power: {achieved_power:.4f}")

if achieved_power < 0.80:
    print("⚠️ Study was underpowered - risk of Type II error")
```

## Adaptive Sample Size (Sequential Testing)

For situations where you can add samples if needed:

```python
from statsmodels.stats.power import TTestIndPower

def sequential_power_check(current_n, alpha=0.05, target_power=0.80):
    """Check if current sample size achieves target power"""
    power_analysis = TTestIndPower()
    
    # Estimate effect size from current data
    # (In practice, use your observed effect size)
    estimated_effect = 0.5
    
    current_power = power_analysis.solve_power(
        effect_size=estimated_effect,
        nobs1=current_n,
        alpha=alpha,
        ratio=1.0,
        alternative='two-sided'
    )
    
    if current_power >= target_power:
        return True, current_power, 0
    else:
        # Calculate additional samples needed
        total_needed = power_analysis.solve_power(
            effect_size=estimated_effect,
            alpha=alpha,
            power=target_power,
            ratio=1.0,
            alternative='two-sided'
        )
        additional_needed = total_needed - current_n
        return False, current_power, additional_needed

# Check at interim analysis
sufficient, power, additional = sequential_power_check(current_n=50)
print(f"Current power: {power:.4f}")
if not sufficient:
    print(f"Need {additional:.0f} more samples per group")
```

## Summary

Power analysis is critical for:
- **Pre-study planning**: Determine required sample size
- **Resource allocation**: Avoid underpowered or wastefully large studies
- **Interpretation**: Understand risk of Type II errors
- **Grant proposals**: Justify sample size requirements

Always conduct power analysis BEFORE data collection, not after.
