# Bayesian Hypothesis Testing

## Overview

Bayesian hypothesis testing provides an alternative to frequentist p-values by calculating the probability of hypotheses given the data, rather than the probability of data given the null hypothesis.

## Key Differences from Frequentist Testing

| Frequentist | Bayesian |
|-------------|----------|
| P(data \| H₀) | P(H₀ \| data) |
| Fixed parameters, random data | Random parameters, observed data |
| p-value (probability of data) | Bayes Factor (evidence ratio) |
| Cannot accept null | Can support null or alternative |
| No prior information | Incorporates prior beliefs |

## Bayes Factor

The Bayes Factor (BF₁₀) quantifies evidence for H₁ vs H₀:

**Interpretation:**
- BF₁₀ > 10: Strong evidence for H₁
- BF₁₀ = 3-10: Moderate evidence for H₁
- BF₁₀ = 1/3-3: Weak/anecdotal evidence
- BF₁₀ < 1/10: Strong evidence for H₀

## Installation

```bash
pip install arviz pymc pingouin
```

## Bayesian T-Test with Pingouin

```python
import pingouin as pg
import numpy as np
import pandas as pd

# Generate sample data
np.random.seed(42)
group_a = np.random.normal(100, 15, 50)
group_b = np.random.normal(110, 15, 50)

# Bayesian independent t-test
bf = pg.bayesfactor_ttest(group_a, group_b, paired=False)

print(f"Bayes Factor (BF₁₀): {bf:.4f}")

# Interpretation
if bf > 10:
    print("Strong evidence for difference between groups")
elif bf > 3:
    print("Moderate evidence for difference")
elif bf > 1:
    print("Weak evidence for difference")
elif bf > 1/3:
    print("Anecdotal evidence (inconclusive)")
else:
    print("Evidence supports null hypothesis (no difference)")

# Bayesian paired t-test
before = np.random.normal(100, 15, 30)
after = before + np.random.normal(5, 10, 30)

bf_paired = pg.bayesfactor_ttest(before, after, paired=True)
print(f"\nPaired t-test BF₁₀: {bf_paired:.4f}")
```

## Bayesian ANOVA with PyMC

```python
import pymc as pm
import arviz as az
import numpy as np
import matplotlib.pyplot as plt

# Generate data for 3 groups
np.random.seed(42)
group_means = [100, 110, 105]
groups_data = [np.random.normal(mean, 15, 30) for mean in group_means]

# Combine into dataframe
import pandas as pd
df = pd.DataFrame({
    'value': np.concatenate(groups_data),
    'group': np.repeat(['A', 'B', 'C'], 30)
})

# Create categorical codes
group_codes = pd.Categorical(df['group']).codes

with pm.Model() as model:
    # Priors
    mu = pm.Normal('mu', mu=100, sigma=20)  # Grand mean
    sigma = pm.HalfNormal('sigma', sigma=20)  # Within-group std
    
    # Group effects
    group_effect = pm.Normal('group_effect', mu=0, sigma=10, shape=3)
    
    # Expected value
    expected = mu + group_effect[group_codes]
    
    # Likelihood
    y = pm.Normal('y', mu=expected, sigma=sigma, observed=df['value'])
    
    # Sample posterior
    trace = pm.sample(2000, tune=1000, return_inferencedata=True, 
                      random_seed=42, progressbar=False)

# Summarize results
print(az.summary(trace, var_names=['mu', 'group_effect', 'sigma']))

# Plot posterior distributions
az.plot_posterior(trace, var_names=['group_effect'])
plt.savefig('bayesian_anova_posterior.png', dpi=300, bbox_inches='tight')

# Calculate probability that groups differ
samples = trace.posterior.stack(sample=('chain', 'draw'))
prob_b_greater_a = (samples['group_effect'].sel(group_effect_dim_0=1) > 
                    samples['group_effect'].sel(group_effect_dim_0=0)).mean()

print(f"\nProbability Group B > Group A: {prob_b_greater_a.values:.4f}")
```

## Bayesian Correlation Test

```python
import pingouin as pg
import numpy as np

# Generate correlated data
np.random.seed(42)
x = np.random.normal(0, 1, 50)
y = 0.6 * x + np.random.normal(0, 0.8, 50)

# Bayesian Pearson correlation
bf = pg.bayesfactor_pearson(x, y)

print(f"Bayes Factor for correlation: {bf:.4f}")

# Compare with frequentist
from scipy import stats
r, p = stats.pearsonr(x, y)
print(f"Frequentist: r={r:.4f}, p={p:.4f}")
```

## Credible Intervals vs Confidence Intervals

```python
import pymc as pm
import arviz as az
import numpy as np

# Sample data
np.random.seed(42)
data = np.random.normal(105, 15, 50)

# Bayesian estimation
with pm.Model() as model:
    mu = pm.Normal('mu', mu=100, sigma=20)
    sigma = pm.HalfNormal('sigma', sigma=20)
    y = pm.Normal('y', mu=mu, sigma=sigma, observed=data)
    
    trace = pm.sample(2000, tune=1000, return_inferencedata=True,
                      random_seed=42, progressbar=False)

# 95% Credible Interval (Bayesian)
credible_interval = az.hdi(trace, hdi_prob=0.95)['mu'].values
print(f"95% Credible Interval: [{credible_interval[0]:.2f}, {credible_interval[1]:.2f}]")
print("Interpretation: 95% probability the true mean is in this range")

# 95% Confidence Interval (Frequentist)
from scipy import stats
ci = stats.t.interval(0.95, len(data)-1, 
                      loc=np.mean(data), 
                      scale=stats.sem(data))
print(f"\n95% Confidence Interval: [{ci[0]:.2f}, {ci[1]:.2f}]")
print("Interpretation: If we repeated the experiment, 95% of intervals would contain true mean")
```

## Sequential Bayesian Testing

Unlike frequentist tests, Bayesian methods allow continuous monitoring without inflating error rates:

```python
import numpy as np
import pingouin as pg
import matplotlib.pyplot as plt

np.random.seed(42)
true_effect = 0.5  # True Cohen's d

# Simulate sequential data collection
sample_sizes = range(10, 201, 10)
bayes_factors = []

for n in sample_sizes:
    # Generate data
    group_a = np.random.normal(0, 1, n)
    group_b = np.random.normal(true_effect, 1, n)
    
    # Calculate BF
    bf = pg.bayesfactor_ttest(group_a, group_b, paired=False)
    bayes_factors.append(bf)

# Plot sequential evidence accumulation
plt.figure(figsize=(10, 6))
plt.plot(sample_sizes, bayes_factors, marker='o')
plt.axhline(y=10, color='g', linestyle='--', label='Strong evidence threshold')
plt.axhline(y=3, color='orange', linestyle='--', label='Moderate evidence threshold')
plt.axhline(y=1/3, color='orange', linestyle='--')
plt.axhline(y=1/10, color='r', linestyle='--')
plt.xlabel('Sample Size per Group')
plt.ylabel('Bayes Factor (BF₁₀)')
plt.title('Sequential Bayesian Evidence Accumulation')
plt.yscale('log')
plt.legend()
plt.grid(True, alpha=0.3)
plt.savefig('sequential_bayes.png', dpi=300, bbox_inches='tight')

print(f"Strong evidence reached at n={sample_sizes[np.argmax(np.array(bayes_factors) > 10)]}")
```

## Model Comparison with WAIC/LOO

```python
import pymc as pm
import arviz as az
import numpy as np

# Generate data
np.random.seed(42)
x = np.linspace(0, 10, 50)
y = 2 * x + np.random.normal(0, 2, 50)

# Model 1: Linear
with pm.Model() as linear_model:
    intercept = pm.Normal('intercept', mu=0, sigma=10)
    slope = pm.Normal('slope', mu=0, sigma=10)
    sigma = pm.HalfNormal('sigma', sigma=5)
    
    mu = intercept + slope * x
    y_obs = pm.Normal('y_obs', mu=mu, sigma=sigma, observed=y)
    
    trace_linear = pm.sample(1000, tune=500, return_inferencedata=True,
                            random_seed=42, progressbar=False)

# Model 2: Quadratic
with pm.Model() as quadratic_model:
    intercept = pm.Normal('intercept', mu=0, sigma=10)
    slope1 = pm.Normal('slope1', mu=0, sigma=10)
    slope2 = pm.Normal('slope2', mu=0, sigma=10)
    sigma = pm.HalfNormal('sigma', sigma=5)
    
    mu = intercept + slope1 * x + slope2 * x**2
    y_obs = pm.Normal('y_obs', mu=mu, sigma=sigma, observed=y)
    
    trace_quad = pm.sample(1000, tune=500, return_inferencedata=True,
                          random_seed=42, progressbar=False)

# Compare models
comparison = az.compare({
    'linear': trace_linear,
    'quadratic': trace_quad
}, ic='waic')

print(comparison)
print("\nLower WAIC = better model")
print("dWAIC > 10 indicates strong preference")
```

## Prior Sensitivity Analysis

```python
import pymc as pm
import arviz as az
import numpy as np

data = np.random.normal(105, 15, 30)

# Test different priors
priors = {
    'weak': {'mu': 20, 'sigma': 50},
    'medium': {'mu': 10, 'sigma': 20},
    'strong': {'mu': 5, 'sigma': 10}
}

results = {}

for prior_name, prior_params in priors.items():
    with pm.Model() as model:
        mu = pm.Normal('mu', mu=100, sigma=prior_params['mu'])
        sigma = pm.HalfNormal('sigma', sigma=prior_params['sigma'])
        y = pm.Normal('y', mu=mu, sigma=sigma, observed=data)
        
        trace = pm.sample(1000, tune=500, return_inferencedata=True,
                         random_seed=42, progressbar=False)
        
        results[prior_name] = trace

# Compare posterior means
for prior_name, trace in results.items():
    posterior_mean = trace.posterior['mu'].mean().values
    print(f"{prior_name.capitalize()} prior → Posterior mean: {posterior_mean:.2f}")

# Visual comparison
import matplotlib.pyplot as plt

fig, axes = plt.subplots(1, 3, figsize=(15, 4))
for idx, (prior_name, trace) in enumerate(results.items()):
    az.plot_posterior(trace, var_names=['mu'], ax=axes[idx])
    axes[idx].set_title(f'{prior_name.capitalize()} Prior')

plt.tight_layout()
plt.savefig('prior_sensitivity.png', dpi=300, bbox_inches='tight')
```

## Reporting Bayesian Results

**Template:**

```
We conducted a Bayesian independent t-test comparing Group A (M = 100.5, SD = 14.8) 
and Group B (M = 110.2, SD = 15.3). The Bayes Factor (BF₁₀ = 12.5) provided strong 
evidence for a difference between groups. The posterior distribution of the difference 
had a 95% credible interval of [4.2, 15.1], suggesting Group B scores were 
4.2-15.1 points higher than Group A with 95% probability.
```

## Advantages of Bayesian Testing

1. **Interpretability**: Direct probability statements about hypotheses
2. **Sequential testing**: Can stop early or continue without penalty
3. **Evidence for null**: Can support H₀, not just fail to reject
4. **Prior knowledge**: Incorporate previous studies
5. **Small samples**: More robust with limited data

## Disadvantages

1. **Prior dependence**: Results influenced by prior choice
2. **Computational cost**: MCMC sampling is slower
3. **Complexity**: Harder to implement and explain
4. **No consensus**: Multiple BF interpretation thresholds

## When to Use Bayesian vs Frequentist

| Use Bayesian | Use Frequentist |
|--------------|----------------|
| Sequential testing/early stopping | Fixed sample size |
| Want evidence FOR null | Only reject/fail to reject |
| Prior knowledge available | No prior information |
| Small sample sizes | Large samples (n > 100) |
| Complex hierarchical models | Simple comparisons |
| Publication allows | Journal requires p-values |

## Resources

- **Books**: "Statistical Rethinking" by Richard McElreath
- **Software**: PyMC, ArviZ, Pingouin, JASP
- **Prior selection**: jasp-stats.org/jasp-materials/
