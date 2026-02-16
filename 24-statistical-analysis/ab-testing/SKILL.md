---
name: ab-testing
description: Provides expert guidance for designing, analyzing, and interpreting A/B tests and experiments using statistical methods including power analysis, sequential testing, and multi-armed bandits for data-driven decision making
version: 1.0.0
author: Orchestra Research
license: MIT
tags: [Statistical Analysis, A/B Testing, Experimental Design, Causal Inference, Hypothesis Testing, Sequential Testing, Bayesian Optimization]
dependencies: [scipy>=1.11.0, statsmodels>=0.14.0, numpy>=1.24.0, pandas>=2.0.0, scikit-learn>=1.3.0]
---

# A/B Testing and Experimental Design

Expert-level guidance for designing, executing, and analyzing A/B tests. This skill covers sample size calculation, randomization, statistical testing, sequential analysis, and multi-armed bandits for optimal experimentation.

## When to Use This Skill

Use A/B testing when you need to:
- **Compare product variants** (e.g., UI changes, pricing, algorithms) with statistical rigor
- **Make causal inferences** about treatment effects in controlled experiments
- **Optimize conversion rates** or other business metrics through experimentation
- **Validate hypotheses** before full product rollout
- **Balance exploration vs exploitation** with bandit algorithms

**Do NOT use A/B testing for:**
- Observational data (use causal inference methods instead)
- When randomization is impossible or unethical
- Metrics with extreme latency (>1 month to observe outcome)
- Very small sample sizes (n < 100 per variant typically)

---

## Core Concepts

### 1. A/B Test Fundamentals

**Key Components:**
- **Control (A)**: Baseline variant
- **Treatment (B)**: New variant being tested
- **Randomization**: Random assignment to A or B
- **Metric**: Outcome you're trying to improve (conversion rate, revenue, etc.)
- **Significance Level (α)**: Typically 0.05 (5% false positive rate)
- **Statistical Power (1-β)**: Typically 0.80 (80% chance to detect true effect)

**Types of Metrics:**
- **Binary**: Conversion, click-through rate (use proportion test)
- **Continuous**: Revenue per user, time on site (use t-test)
- **Count**: Number of purchases, page views (use Poisson/negative binomial)
- **Ratio**: Average order value, revenue per session (use delta method)

### 2. Minimum Detectable Effect (MDE)

The smallest effect size you want to reliably detect:
- MDE = 10% relative lift is common for product metrics
- Smaller MDE requires larger sample sizes
- Trade-off between statistical power and experiment duration

---

## Workflow 1: Designing an A/B Test

**Use Case**: Plan experiment before launching

### Checklist

- [ ] Define hypothesis and primary metric
- [ ] Calculate required sample size
- [ ] Determine experiment duration
- [ ] Design randomization strategy
- [ ] Set up tracking and instrumentation
- [ ] Define success criteria and decision rules
- [ ] Plan for multiple testing corrections (if testing multiple metrics)

### Implementation

```python
import numpy as np
from statsmodels.stats.power import zt_ind_solve_power
from statsmodels.stats.proportion import proportion_effectsize
import pandas as pd

# Step 1: Define experiment parameters
baseline_conversion = 0.10  # Current conversion rate: 10%
mde = 0.15  # Minimum detectable effect: 15% relative lift
alpha = 0.05  # Significance level
power = 0.80  # Statistical power

# Calculate absolute effect size
treatment_conversion = baseline_conversion * (1 + mde)
print(f"Baseline conversion: {baseline_conversion:.2%}")
print(f"Treatment conversion (if MDE achieved): {treatment_conversion:.2%}")
print(f"Absolute lift: {treatment_conversion - baseline_conversion:.2%}")

# Step 2: Calculate required sample size
effect_size = proportion_effectsize(baseline_conversion, treatment_conversion)

sample_size_per_variant = zt_ind_solve_power(
    effect_size=effect_size,
    alpha=alpha,
    power=power,
    ratio=1.0,  # Equal allocation to A and B
    alternative='two-sided'
)

total_sample_size = 2 * sample_size_per_variant

print(f"\nSample Size Calculation:")
print(f"  Effect size (Cohen's h): {effect_size:.4f}")
print(f"  Sample size per variant: {sample_size_per_variant:.0f}")
print(f"  Total sample size: {total_sample_size:.0f}")

# Step 3: Calculate experiment duration
daily_users = 10000  # Average daily users
allocation_rate = 1.0  # 100% of users enter experiment

days_required = total_sample_size / (daily_users * allocation_rate)

print(f"\nExperiment Duration:")
print(f"  Daily users: {daily_users:,}")
print(f"  Allocation rate: {allocation_rate:.0%}")
print(f"  Days required: {days_required:.1f}")

# Step 4: Power analysis for different scenarios
print("\nPower Analysis for Different Effect Sizes:")
print("="*60)
print(f"{'Relative Lift':<15} {'Sample Size':<15} {'Days Required':<15}")
print("-"*60)

for relative_lift in [0.05, 0.10, 0.15, 0.20, 0.25]:
    treatment_rate = baseline_conversion * (1 + relative_lift)
    es = proportion_effectsize(baseline_conversion, treatment_rate)
    n = zt_ind_solve_power(effect_size=es, alpha=alpha, power=power, 
                           ratio=1.0, alternative='two-sided')
    days = (2 * n) / daily_users
    
    print(f"{relative_lift:.0%}             {n:.0f}             {days:.1f}")

# Step 5: Create tracking plan
tracking_plan = pd.DataFrame({
    'Metric': ['Primary: Conversion Rate', 'Secondary: Revenue per User', 
               'Guardrail: Bounce Rate'],
    'Type': ['Binary', 'Continuous', 'Binary'],
    'Success Criteria': ['Increase >0%', 'No decrease', '<5% increase'],
    'Statistical Test': ['Proportion test', 'T-test', 'Proportion test']
})

print("\n" + "="*60)
print("TRACKING PLAN")
print("="*60)
print(tracking_plan.to_string(index=False))
```

---

## Workflow 2: Running and Analyzing an A/B Test

**Use Case**: Analyze results after experiment completes

### Checklist

- [ ] Verify randomization (check covariate balance)
- [ ] Check sample ratio mismatch
- [ ] Perform statistical test on primary metric
- [ ] Calculate confidence intervals
- [ ] Test secondary and guardrail metrics
- [ ] Adjust for multiple comparisons
- [ ] Interpret results and make decision

### Implementation

```python
import numpy as np
import pandas as pd
from scipy import stats
from statsmodels.stats.proportion import proportions_ztest, proportion_confint

# Step 1: Load experiment data
np.random.seed(42)

# Simulate A/B test data
n_control = 5000
n_treatment = 5000

# Control group (10% conversion)
control_conversions = np.random.binomial(1, 0.10, n_control)

# Treatment group (12% conversion - 20% relative lift)
treatment_conversions = np.random.binomial(1, 0.12, n_treatment)

df = pd.DataFrame({
    'variant': ['A']*n_control + ['B']*n_treatment,
    'converted': np.concatenate([control_conversions, treatment_conversions])
})

# Step 2: Calculate summary statistics
summary = df.groupby('variant')['converted'].agg([
    ('users', 'count'),
    ('conversions', 'sum'),
    ('rate', 'mean')
])

print("="*60)
print("A/B TEST RESULTS")
print("="*60)
print(summary)

control_rate = summary.loc['A', 'rate']
treatment_rate = summary.loc['B', 'rate']
absolute_lift = treatment_rate - control_rate
relative_lift = (treatment_rate - control_rate) / control_rate

print(f"\nAbsolute lift: {absolute_lift:.4f} ({absolute_lift*100:.2f} percentage points)")
print(f"Relative lift: {relative_lift:.2%}")

# Step 3: Perform two-proportion z-test
conversions = np.array([summary.loc['B', 'conversions'], 
                        summary.loc['A', 'conversions']])
users = np.array([summary.loc['B', 'users'], 
                  summary.loc['A', 'users']])

z_stat, p_value = proportions_ztest(conversions, users, alternative='larger')

print(f"\nStatistical Test (Two-Proportion Z-Test):")
print(f"  Z-statistic: {z_stat:.4f}")
print(f"  P-value: {p_value:.4f}")

if p_value < 0.05:
    print(f"  ✓ Result is statistically significant (p < 0.05)")
    print(f"  → Treatment B performs better than control A")
else:
    print(f"  ✗ Result is not statistically significant (p ≥ 0.05)")
    print(f"  → Cannot conclude that B is better than A")

# Step 4: Calculate confidence intervals
ci_control = proportion_confint(summary.loc['A', 'conversions'], 
                                summary.loc['A', 'users'], 
                                alpha=0.05, method='wilson')
ci_treatment = proportion_confint(summary.loc['B', 'conversions'], 
                                  summary.loc['B', 'users'], 
                                  alpha=0.05, method='wilson')

print(f"\n95% Confidence Intervals:")
print(f"  Control (A): [{ci_control[0]:.4f}, {ci_control[1]:.4f}]")
print(f"  Treatment (B): [{ci_treatment[0]:.4f}, {ci_treatment[1]:.4f}]")

# Confidence interval for difference
se_diff = np.sqrt(
    control_rate*(1-control_rate)/n_control + 
    treatment_rate*(1-treatment_rate)/n_treatment
)
ci_diff = (absolute_lift - 1.96*se_diff, absolute_lift + 1.96*se_diff)
print(f"  Difference: [{ci_diff[0]:.4f}, {ci_diff[1]:.4f}]")

# Step 5: Check sample ratio mismatch (SRM)
expected_ratio = 0.5  # Equal allocation
observed_ratio = n_treatment / (n_control + n_treatment)
chi2_stat = ((n_control + n_treatment) * 
             (observed_ratio - expected_ratio)**2 / expected_ratio)
srm_p_value = 1 - stats.chi2.cdf(chi2_stat, df=1)

print(f"\nSample Ratio Mismatch Check:")
print(f"  Expected ratio: {expected_ratio:.2%}")
print(f"  Observed ratio: {observed_ratio:.4f}")
print(f"  SRM p-value: {srm_p_value:.4f}")

if srm_p_value < 0.01:
    print(f"  ✗ WARNING: Sample ratio mismatch detected!")
    print(f"  → Check randomization implementation")
else:
    print(f"  ✓ No sample ratio mismatch")

# Step 6: Visualize results
import matplotlib.pyplot as plt
import seaborn as sns

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Bar chart with confidence intervals
variants = ['Control (A)', 'Treatment (B)']
rates = [control_rate, treatment_rate]
cis = [ci_control, ci_treatment]
errors = [[rates[i] - cis[i][0], cis[i][1] - rates[i]] for i in range(2)]

axes[0].bar(variants, rates, alpha=0.7, color=['steelblue', 'coral'])
axes[0].errorbar(variants, rates, 
                yerr=np.array(errors).T,
                fmt='none', color='black', capsize=5)
axes[0].set_ylabel('Conversion Rate')
axes[0].set_title(f'A/B Test Results (p={p_value:.4f})')
axes[0].grid(True, alpha=0.3, axis='y')

# Distribution plot
axes[1].hist(control_conversions, bins=2, alpha=0.5, label='Control', 
            density=True, edgecolor='black')
axes[1].hist(treatment_conversions, bins=2, alpha=0.5, label='Treatment', 
            density=True, edgecolor='black')
axes[1].set_xlabel('Converted (0 = No, 1 = Yes)')
axes[1].set_ylabel('Density')
axes[1].set_title('Conversion Distribution')
axes[1].legend()
axes[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('ab_test_results.png', dpi=300, bbox_inches='tight')
```

---

## Workflow 3: Sequential Testing (Early Stopping)

**Use Case**: Monitor experiment continuously and stop early if conclusive

### Implementation

```python
import numpy as np
from scipy import stats
import matplotlib.pyplot as plt

def sequential_probability_ratio_test(control_conv, treatment_conv, 
                                      alpha=0.05, beta=0.20):
    """
    Sequential Probability Ratio Test (SPRT)
    
    Returns: (decision, log_likelihood_ratio)
    decision: 'continue', 'reject_null', 'accept_null'
    """
    n_control = len(control_conv)
    n_treatment = len(treatment_conv)
    
    # Observed conversion rates
    p_control = np.mean(control_conv)
    p_treatment = np.mean(treatment_conv)
    
    # Log likelihood ratio
    if p_control > 0 and p_control < 1 and p_treatment > 0 and p_treatment < 1:
        llr = (np.sum(control_conv) * np.log(p_treatment / p_control) + 
               np.sum(1 - control_conv) * np.log((1-p_treatment) / (1-p_control)))
    else:
        llr = 0
    
    # Decision boundaries
    upper_bound = np.log((1 - beta) / alpha)
    lower_bound = np.log(beta / (1 - alpha))
    
    if llr >= upper_bound:
        return 'reject_null', llr  # Treatment is better
    elif llr <= lower_bound:
        return 'accept_null', llr  # No difference
    else:
        return 'continue', llr

# Simulate sequential monitoring
np.random.seed(42)
max_n = 10000
check_intervals = np.arange(100, max_n+1, 100)

control_rate = 0.10
treatment_rate = 0.12  # 20% relative lift

decisions = []
llrs = []

for n in check_intervals:
    # Generate data up to this point
    control = np.random.binomial(1, control_rate, n)
    treatment = np.random.binomial(1, treatment_rate, n)
    
    # Perform SPRT
    decision, llr = sequential_probability_ratio_test(control, treatment)
    decisions.append(decision)
    llrs.append(llr)
    
    print(f"n={n:5d}: {decision:15s} (LLR={llr:.2f})")
    
    if decision != 'continue':
        print(f"\n✓ Experiment concluded at n={n}")
        print(f"  Decision: {decision}")
        break

# Plot sequential analysis
plt.figure(figsize=(12, 6))
plt.plot(check_intervals[:len(llrs)], llrs, 'o-', label='Log Likelihood Ratio')
plt.axhline(y=np.log((1-0.20)/0.05), color='g', linestyle='--', 
           label='Upper boundary (reject H₀)')
plt.axhline(y=np.log(0.20/(1-0.05)), color='r', linestyle='--', 
           label='Lower boundary (accept H₀)')
plt.xlabel('Sample Size per Variant')
plt.ylabel('Log Likelihood Ratio')
plt.title('Sequential Probability Ratio Test')
plt.legend()
plt.grid(True, alpha=0.3)
plt.savefig('sequential_test.png', dpi=300, bbox_inches='tight')
```

---

## Workflow 4: Multi-Armed Bandits (Exploration vs Exploitation)

**Use Case**: Optimize metric while learning which variant is best

### Implementation

```python
import numpy as np
import matplotlib.pyplot as plt

class ThompsonSamplingBandit:
    """
    Thompson Sampling for Bernoulli bandits
    Balances exploration and exploitation automatically
    """
    def __init__(self, n_arms):
        self.n_arms = n_arms
        self.successes = np.ones(n_arms)  # Prior: Beta(1,1)
        self.failures = np.ones(n_arms)
        
    def select_arm(self):
        """Sample from posterior and select arm with highest sample"""
        samples = np.random.beta(self.successes, self.failures)
        return np.argmax(samples)
    
    def update(self, arm, reward):
        """Update posterior based on observed reward"""
        if reward == 1:
            self.successes[arm] += 1
        else:
            self.failures[arm] += 1
    
    def get_probabilities(self):
        """Return posterior mean conversion rates"""
        return self.successes / (self.successes + self.failures)

# Simulate multi-armed bandit experiment
np.random.seed(42)
n_arms = 3
true_rates = [0.10, 0.12, 0.09]  # Variant B is best
n_rounds = 5000

# Initialize bandit
bandit = ThompsonSamplingBandit(n_arms)

# Track metrics
arm_counts = np.zeros(n_arms)
cumulative_reward = 0
rewards_over_time = []
arm_selection_over_time = []

for t in range(n_rounds):
    # Select arm
    arm = bandit.select_arm()
    arm_counts[arm] += 1
    arm_selection_over_time.append(arm)
    
    # Observe reward
    reward = np.random.binomial(1, true_rates[arm])
    
    # Update bandit
    bandit.update(arm, reward)
    cumulative_reward += reward
    rewards_over_time.append(cumulative_reward / (t + 1))

# Final results
print("="*60)
print("MULTI-ARMED BANDIT RESULTS")
print("="*60)
print(f"\nTrue conversion rates: {true_rates}")
print(f"Estimated rates: {bandit.get_probabilities()}")
print(f"\nArm selection counts:")
for i in range(n_arms):
    print(f"  Variant {chr(65+i)}: {arm_counts[i]:.0f} ({arm_counts[i]/n_rounds:.1%})")

print(f"\nTotal conversions: {cumulative_reward:.0f}")
print(f"Overall conversion rate: {cumulative_reward/n_rounds:.4f}")

# Compare to fixed A/B test
fixed_ab_conversions = n_rounds * (0.5 * true_rates[0] + 0.5 * true_rates[1])
regret = cumulative_reward - fixed_ab_conversions

print(f"\nComparison to fixed 50/50 A/B test:")
print(f"  Bandit conversions: {cumulative_reward:.0f}")
print(f"  Fixed A/B conversions: {fixed_ab_conversions:.0f}")
print(f"  Advantage: {regret:.0f} additional conversions")

# Visualize
fig, axes = plt.subplots(2, 1, figsize=(12, 10))

# Arm selection over time
window = 100
arm_selection_smoothed = pd.Series(arm_selection_over_time).rolling(window).apply(
    lambda x: (x == 1).sum() / len(x)
)

axes[0].plot(arm_selection_smoothed, label='Variant B Selection Rate (smoothed)')
axes[0].axhline(y=1.0, color='g', linestyle='--', alpha=0.5, label='Optimal (100%)')
axes[0].set_xlabel('Round')
axes[0].set_ylabel('Selection Probability')
axes[0].set_title('Thompson Sampling: Convergence to Best Arm')
axes[0].legend()
axes[0].grid(True, alpha=0.3)

# Cumulative conversion rate
axes[1].plot(rewards_over_time, label='Bandit Strategy')
axes[1].axhline(y=np.max(true_rates), color='g', linestyle='--', 
               label='Optimal (always choose best)')
axes[1].axhline(y=np.mean(true_rates[:2]), color='r', linestyle='--', 
               label='Fixed 50/50 A/B')
axes[1].set_xlabel('Round')
axes[1].set_ylabel('Cumulative Conversion Rate')
axes[1].set_title('Cumulative Performance')
axes[1].legend()
axes[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('bandit_results.png', dpi=300, bbox_inches='tight')
```

---

## Common Issues and Solutions

### Issue 1: Peeking (Repeated Testing)

**Problem**: Looking at results multiple times inflates Type I error

**Solution**: Use sequential testing or adjust alpha with Bonferroni correction

```python
# Bonferroni correction for k looks
k_looks = 5
adjusted_alpha = 0.05 / k_looks
print(f"Adjusted alpha for {k_looks} looks: {adjusted_alpha:.4f}")
```

### Issue 2: Novelty/Primacy Effects

**Problem**: Users react differently to change initially

**Solution**: Run for at least 2 business cycles or exclude first few days

```python
# Exclude first 3 days of data
df_filtered = df[df['days_since_start'] > 3]
```

### Issue 3: Multiple Testing

**Problem**: Testing many metrics increases false positives

**Solution**: Designate primary metric, use FDR correction for others

```python
from statsmodels.stats.multitest import multipletests

p_values = [0.01, 0.04, 0.06, 0.12]  # Multiple metric p-values
reject, pvals_corrected, _, _ = multipletests(p_values, alpha=0.05, 
                                              method='fdr_bh')

print("FDR-corrected decisions:", reject)
```

### Issue 4: Unequal Sample Sizes

**Problem**: Imbalanced randomization reduces power

**Solution**: Check for SRM, use appropriate tests

```python
# Welch's t-test handles unequal sample sizes
from scipy import stats

# Different sample sizes
group_a = np.random.normal(100, 15, 1000)
group_b = np.random.normal(105, 15, 1200)

t_stat, p_value = stats.ttest_ind(group_a, group_b, equal_var=False)
print(f"Welch's t-test p-value: {p_value:.4f}")
```

---

## When to Use vs Alternatives

| Use A/B Testing | Use Alternative |
|-----------------|-----------------|
| Can randomize users | Cannot randomize → Observational causal inference (DID, RDD) |
| Simple A vs B comparison | Many variants → Multi-armed bandits |
| Low switching cost | High cost to change → Bayesian optimization |
| Immediate feedback | Long feedback loops → Cohort analysis |
| Binary/continuous metrics | Complex user journeys → Funnel analysis |

---

## Advanced Features

**Bayesian A/B Testing**: See [references/bayesian-ab-testing.md](references/bayesian-ab-testing.md)
**CUPED (Variance Reduction)**: See [references/variance-reduction.md](references/variance-reduction.md)
**Multi-Variant Testing**: See [references/multivariate-testing.md](references/multivariate-testing.md)

---

## Quick Reference

```python
# Sample size calculation
from statsmodels.stats.power import zt_ind_solve_power
from statsmodels.stats.proportion import proportion_effectsize

effect = proportion_effectsize(0.10, 0.12)
n = zt_ind_solve_power(effect_size=effect, alpha=0.05, power=0.80)

# Proportion test
from statsmodels.stats.proportion import proportions_ztest

z_stat, p_value = proportions_ztest([conv_b, conv_a], [n_b, n_a])

# T-test for continuous metrics
from scipy import stats

t_stat, p_value = stats.ttest_ind(group_a, group_b)

# Sequential testing
# Use SPRT or alpha-spending functions (see references)

# Multi-armed bandit
# Use Thompson Sampling or UCB algorithms
```

---

## Summary

This skill provides production-ready workflows for:
- **Experiment design** with power analysis and sample size calculation
- **Statistical testing** for binary and continuous metrics
- **Sequential analysis** for early stopping
- **Multi-armed bandits** for exploration-exploitation tradeoff

**Key Principle**: Always pre-register your hypothesis, primary metric, and sample size. Avoid p-hacking by not peeking at results before reaching target sample size, or use proper sequential testing methods.
