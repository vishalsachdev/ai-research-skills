# Variance Reduction Techniques (CUPED)

## Overview

Variance reduction increases statistical power without increasing sample size by controlling for pre-experiment covariates. CUPED (Controlled-experiment Using Pre-Experiment Data) is the most common technique.

## Why Variance Reduction Matters

**Problem**: High variance in metrics reduces power to detect effects

**Solution**: Use pre-experiment data to reduce variance

**Benefits:**
- Detect smaller effects with same sample size
- Shorter experiments (reach significance faster)
- More cost-effective testing

## CUPED (Controlled-experiment Using Pre-Experiment Data)

Basic idea: Adjust metric using pre-experiment values of the same metric

```python
import numpy as np
import pandas as pd
from scipy import stats

def cuped_adjustment(y_post, y_pre, treatment):
    """
    Apply CUPED variance reduction
    
    Parameters:
    -----------
    y_post : array
        Post-experiment metric values
    y_pre : array
        Pre-experiment metric values (same metric, before experiment)
    treatment : array
        Treatment assignment (0 = control, 1 = treatment)
    
    Returns:
    --------
    y_adjusted : array
        Variance-reduced metric
    variance_reduction : float
        Proportion of variance reduced
    """
    # Overall mean of pre-experiment metric
    theta = np.mean(y_pre)
    
    # Covariance and variance
    cov = np.cov(y_post, y_pre)[0, 1]
    var_pre = np.var(y_pre, ddof=1)
    
    # Optimal coefficient
    alpha = cov / var_pre
    
    # Adjusted metric
    y_adjusted = y_post - alpha * (y_pre - theta)
    
    # Variance reduction
    var_original = np.var(y_post, ddof=1)
    var_adjusted = np.var(y_adjusted, ddof=1)
    variance_reduction = 1 - (var_adjusted / var_original)
    
    return y_adjusted, variance_reduction

# Simulate experiment with pre-experiment data
np.random.seed(42)
n = 1000

# Pre-experiment metric (correlated with post-experiment)
y_pre = np.random.normal(100, 20, n)

# Post-experiment metric (correlated with pre-experiment)
treatment = np.random.binomial(1, 0.5, n)
noise = np.random.normal(0, 15, n)

# Treatment effect: +5 points
y_post = 0.7 * y_pre + 5 * treatment + noise

# Unadjusted analysis
control_post = y_post[treatment == 0]
treatment_post = y_post[treatment == 1]

t_unadjusted, p_unadjusted = stats.ttest_ind(treatment_post, control_post)

print("="*60)
print("UNADJUSTED ANALYSIS")
print("="*60)
print(f"Control mean: {control_post.mean():.2f}")
print(f"Treatment mean: {treatment_post.mean():.2f}")
print(f"Difference: {treatment_post.mean() - control_post.mean():.2f}")
print(f"t-statistic: {t_unadjusted:.4f}")
print(f"p-value: {p_unadjusted:.4f}")
print(f"Standard error: {np.sqrt(np.var(control_post, ddof=1)/len(control_post) + np.var(treatment_post, ddof=1)/len(treatment_post)):.4f}")

# CUPED adjustment
y_adjusted, var_reduction = cuped_adjustment(y_post, y_pre, treatment)

control_adjusted = y_adjusted[treatment == 0]
treatment_adjusted = y_adjusted[treatment == 1]

t_adjusted, p_adjusted = stats.ttest_ind(treatment_adjusted, control_adjusted)

print("\n" + "="*60)
print("CUPED-ADJUSTED ANALYSIS")
print("="*60)
print(f"Control mean: {control_adjusted.mean():.2f}")
print(f"Treatment mean: {treatment_adjusted.mean():.2f}")
print(f"Difference: {treatment_adjusted.mean() - control_adjusted.mean():.2f}")
print(f"t-statistic: {t_adjusted:.4f}")
print(f"p-value: {p_adjusted:.4f}")
print(f"Standard error: {np.sqrt(np.var(control_adjusted, ddof=1)/len(control_adjusted) + np.var(treatment_adjusted, ddof=1)/len(treatment_adjusted)):.4f}")

print(f"\nVariance reduction: {var_reduction:.1%}")
print(f"Effective sample size increase: {1/(1-var_reduction):.2f}x")
```

## Variance Reduction with statsmodels

Using regression-based CUPED:

```python
import statsmodels.api as sm
import numpy as np
import pandas as pd

def cuped_regression(df, metric_post, metric_pre, treatment_col):
    """
    CUPED using regression (equivalent to covariance adjustment)
    """
    # Fit regression: y_post ~ y_pre (on full data)
    X_pre = sm.add_constant(df[metric_pre])
    model_pre = sm.OLS(df[metric_post], X_pre).fit()
    
    # Get residuals (variance-reduced metric)
    df['residuals'] = model_pre.resid
    
    # Test treatment effect on residuals
    control_resid = df[df[treatment_col] == 0]['residuals']
    treatment_resid = df[df[treatment_col] == 1]['residuals']
    
    # T-test
    from scipy import stats
    t_stat, p_value = stats.ttest_ind(treatment_resid, control_resid)
    
    # Alternatively, use regression with treatment indicator
    df['treatment'] = df[treatment_col]
    X = sm.add_constant(df[[metric_pre, 'treatment']])
    model = sm.OLS(df[metric_post], X).fit()
    
    return {
        'treatment_effect': model.params['treatment'],
        'se': model.bse['treatment'],
        'p_value': model.pvalues['treatment'],
        't_stat': model.tvalues['treatment'],
        'ci_lower': model.conf_int().loc['treatment', 0],
        'ci_upper': model.conf_int().loc['treatment', 1],
        'r_squared': model.rsquared
    }

# Example
df = pd.DataFrame({
    'metric_pre': y_pre,
    'metric_post': y_post,
    'treatment': treatment
})

results = cuped_regression(df, 'metric_post', 'metric_pre', 'treatment')

print("\nRegression-based CUPED:")
print(f"  Treatment effect: {results['treatment_effect']:.2f}")
print(f"  Standard error: {results['se']:.4f}")
print(f"  95% CI: [{results['ci_lower']:.2f}, {results['ci_upper']:.2f}]")
print(f"  p-value: {results['p_value']:.4f}")
print(f"  R²: {results['r_squared']:.4f}")
```

## Multiple Covariates

Use multiple pre-experiment metrics:

```python
import statsmodels.api as sm

# Multiple covariates
df['metric_pre_2'] = np.random.normal(50, 10, n)
df['metric_pre_3'] = np.random.normal(30, 5, n)

# Regression with multiple covariates
X = sm.add_constant(df[['metric_pre', 'metric_pre_2', 'metric_pre_3', 'treatment']])
model_multi = sm.OLS(df['metric_post'], X).fit()

print("\nMultiple Covariate Adjustment:")
print(model_multi.summary())

# Variance reduction
from sklearn.metrics import r2_score
y_pred_no_treatment = model_multi.predict(
    sm.add_constant(df[['metric_pre', 'metric_pre_2', 'metric_pre_3']])
)
r2_without_treatment = r2_score(df['metric_post'], y_pred_no_treatment)

print(f"\nVariance explained by covariates: {r2_without_treatment:.1%}")
```

## Stratified Sampling

Pre-stratify randomization based on covariates:

```python
import numpy as np

def stratified_randomization(df, strata_col, n_strata=4):
    """
    Randomize within strata (e.g., deciles of pre-experiment metric)
    """
    # Create strata based on pre-experiment metric
    df['stratum'] = pd.qcut(df[strata_col], q=n_strata, labels=False, duplicates='drop')
    
    # Randomize within each stratum
    def assign_treatment(group):
        n = len(group)
        treatment = np.random.permutation([0]*(n//2) + [1]*(n - n//2))
        return pd.Series(treatment, index=group.index)
    
    df['treatment'] = df.groupby('stratum').apply(assign_treatment).values
    
    return df

# Example
df_strat = pd.DataFrame({
    'metric_pre': np.random.normal(100, 20, 1000)
})

df_strat = stratified_randomization(df_strat, 'metric_pre', n_strata=4)

# Check balance
print("Treatment assignment by stratum:")
print(df_strat.groupby('stratum')['treatment'].value_counts().unstack())
```

## Post-Stratification

Adjust after randomization:

```python
def post_stratification_adjustment(df, metric_col, treatment_col, strata_col, n_strata=5):
    """
    Weight observations by stratum to reduce variance
    """
    # Create strata
    df['stratum'] = pd.qcut(df[strata_col], q=n_strata, labels=False, duplicates='drop')
    
    # Calculate stratum proportions in population
    stratum_props = df['stratum'].value_counts(normalize=True)
    
    # Calculate treatment effect within each stratum
    stratum_effects = []
    stratum_vars = []
    
    for stratum in df['stratum'].unique():
        stratum_data = df[df['stratum'] == stratum]
        
        control = stratum_data[stratum_data[treatment_col] == 0][metric_col]
        treatment = stratum_data[stratum_data[treatment_col] == 1][metric_col]
        
        effect = treatment.mean() - control.mean()
        
        # Variance of difference
        var_diff = (control.var(ddof=1)/len(control) + 
                   treatment.var(ddof=1)/len(treatment))
        
        stratum_effects.append(effect)
        stratum_vars.append(var_diff)
    
    # Weighted average
    weights = [stratum_props[s] for s in df['stratum'].unique()]
    weighted_effect = np.average(stratum_effects, weights=weights)
    weighted_var = np.average(stratum_vars, weights=np.array(weights)**2)
    weighted_se = np.sqrt(weighted_var)
    
    return {
        'effect': weighted_effect,
        'se': weighted_se,
        't_stat': weighted_effect / weighted_se,
        'p_value': 2 * (1 - stats.t.cdf(abs(weighted_effect / weighted_se), 
                                         df=len(df) - 2))
    }

results_ps = post_stratification_adjustment(df, 'metric_post', 'treatment', 'metric_pre')

print("\nPost-Stratification Results:")
print(f"  Effect: {results_ps['effect']:.2f}")
print(f"  SE: {results_ps['se']:.4f}")
print(f"  p-value: {results_ps['p_value']:.4f}")
```

## Difference-in-Differences

Control for time trends:

```python
import pandas as pd
import statsmodels.api as sm

# Simulate DID data
np.random.seed(42)
n_per_group = 500

# Pre-period
df_pre = pd.DataFrame({
    'group': ['control']*n_per_group + ['treatment']*n_per_group,
    'period': 'pre',
    'metric': np.concatenate([
        np.random.normal(100, 20, n_per_group),  # Control pre
        np.random.normal(100, 20, n_per_group)   # Treatment pre (same)
    ])
})

# Post-period (with time trend and treatment effect)
time_trend = 5
treatment_effect = 10

df_post = pd.DataFrame({
    'group': ['control']*n_per_group + ['treatment']*n_per_group,
    'period': 'post',
    'metric': np.concatenate([
        np.random.normal(100 + time_trend, 20, n_per_group),  # Control post
        np.random.normal(100 + time_trend + treatment_effect, 20, n_per_group)  # Treatment post
    ])
})

df_did = pd.concat([df_pre, df_post], ignore_index=True)

# Create dummy variables
df_did['treatment'] = (df_did['group'] == 'treatment').astype(int)
df_did['post'] = (df_did['period'] == 'post').astype(int)
df_did['treatment_x_post'] = df_did['treatment'] * df_did['post']

# DID regression
X = sm.add_constant(df_did[['treatment', 'post', 'treatment_x_post']])
model_did = sm.OLS(df_did['metric'], X).fit()

print("\nDifference-in-Differences:")
print(model_did.summary())

# The coefficient on treatment_x_post is the DID estimator
did_effect = model_did.params['treatment_x_post']
print(f"\nDID Treatment Effect: {did_effect:.2f}")
print(f"  (True effect: {treatment_effect})")
```

## Comparing Methods

```python
def compare_variance_reduction_methods(df):
    """Compare different variance reduction techniques"""
    
    methods = {}
    
    # 1. Unadjusted
    control = df[df['treatment'] == 0]['metric_post']
    treatment = df[df['treatment'] == 1]['metric_post']
    t, p = stats.ttest_ind(treatment, control)
    se = np.sqrt(control.var(ddof=1)/len(control) + treatment.var(ddof=1)/len(treatment))
    
    methods['Unadjusted'] = {'effect': treatment.mean() - control.mean(), 
                             'se': se, 'p': p}
    
    # 2. CUPED
    y_adj, var_red = cuped_adjustment(df['metric_post'].values, 
                                      df['metric_pre'].values, 
                                      df['treatment'].values)
    control_adj = y_adj[df['treatment'] == 0]
    treatment_adj = y_adj[df['treatment'] == 1]
    t, p = stats.ttest_ind(treatment_adj, control_adj)
    se = np.sqrt(control_adj.var(ddof=1)/len(control_adj) + 
                 treatment_adj.var(ddof=1)/len(treatment_adj))
    
    methods['CUPED'] = {'effect': treatment_adj.mean() - control_adj.mean(), 
                       'se': se, 'p': p}
    
    # 3. Regression adjustment
    X = sm.add_constant(df[['metric_pre', 'treatment']])
    model = sm.OLS(df['metric_post'], X).fit()
    
    methods['Regression'] = {'effect': model.params['treatment'],
                            'se': model.bse['treatment'],
                            'p': model.pvalues['treatment']}
    
    # Compare
    results_df = pd.DataFrame(methods).T
    results_df['relative_se'] = results_df['se'] / results_df.loc['Unadjusted', 'se']
    
    return results_df

comparison = compare_variance_reduction_methods(df)
print("\n" + "="*60)
print("COMPARISON OF VARIANCE REDUCTION METHODS")
print("="*60)
print(comparison)
```

## When to Use Variance Reduction

**Best for:**
- Metrics correlated with pre-experiment values (retention, revenue)
- Long experiments (user behavior stable over time)
- Small effect sizes (need extra power)

**Not useful for:**
- New user metrics (no pre-experiment data)
- Uncorrelated metrics
- Very large effects (already have high power)

## Summary

**Variance Reduction Techniques:**
1. **CUPED**: Use pre-experiment values of same metric
2. **Regression adjustment**: Control for multiple covariates
3. **Stratification**: Randomize within groups
4. **Post-stratification**: Weight by strata after randomization
5. **Difference-in-differences**: Control for time trends

**Power Gain:**
- Typical variance reduction: 30-70%
- Equivalent to 1.5x - 3x sample size increase
- Enables shorter experiments or detection of smaller effects
