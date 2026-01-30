# Bayesian A/B Testing

## Overview

Bayesian A/B testing provides an alternative to frequentist hypothesis testing by calculating probabilities that one variant is better than another, rather than p-values. This approach is often more intuitive for business decision-making.

## Key Advantages

1. **Direct probability statements**: "95% probability that B is better than A"
2. **No fixed sample size**: Can stop whenever decision is clear
3. **Incorporates prior knowledge**: Use historical data
4. **Decision-focused**: Built-in loss functions for business decisions

## Beta-Binomial Model for Conversion Rates

For binary outcomes (converted/not converted):

```python
import numpy as np
import matplotlib.pyplot as plt
from scipy import stats

class BayesianABTest:
    """
    Bayesian A/B test for conversion rates using Beta-Binomial model
    """
    def __init__(self, prior_alpha=1, prior_beta=1):
        # Prior: Beta(alpha, beta)
        # Uninformative prior: Beta(1, 1) = Uniform(0, 1)
        self.prior_alpha = prior_alpha
        self.prior_beta = prior_beta
        
        # Data
        self.data = {
            'A': {'conversions': 0, 'trials': 0},
            'B': {'conversions': 0, 'trials': 0}
        }
    
    def update(self, variant, conversions, trials):
        """Update with new data"""
        self.data[variant]['conversions'] += conversions
        self.data[variant]['trials'] += trials
    
    def get_posterior(self, variant):
        """Get posterior distribution parameters"""
        alpha = self.prior_alpha + self.data[variant]['conversions']
        beta = self.prior_beta + (self.data[variant]['trials'] - 
                                  self.data[variant]['conversions'])
        return alpha, beta
    
    def sample_posterior(self, variant, n_samples=10000):
        """Draw samples from posterior distribution"""
        alpha, beta = self.get_posterior(variant)
        return np.random.beta(alpha, beta, n_samples)
    
    def prob_b_better_than_a(self, n_samples=10000):
        """Calculate P(B > A | data)"""
        samples_a = self.sample_posterior('A', n_samples)
        samples_b = self.sample_posterior('B', n_samples)
        return (samples_b > samples_a).mean()
    
    def expected_loss(self, n_samples=10000):
        """Calculate expected loss for choosing each variant"""
        samples_a = self.sample_posterior('A', n_samples)
        samples_b = self.sample_posterior('B', n_samples)
        
        # Expected loss if we choose A
        loss_a = np.maximum(0, samples_b - samples_a).mean()
        
        # Expected loss if we choose B
        loss_b = np.maximum(0, samples_a - samples_b).mean()
        
        return {'A': loss_a, 'B': loss_b}
    
    def credible_interval(self, variant, alpha=0.05):
        """Calculate credible interval"""
        post_alpha, post_beta = self.get_posterior(variant)
        lower = stats.beta.ppf(alpha/2, post_alpha, post_beta)
        upper = stats.beta.ppf(1 - alpha/2, post_alpha, post_beta)
        return lower, upper

# Example usage
test = BayesianABTest()

# Simulate data
np.random.seed(42)
test.update('A', conversions=480, trials=5000)  # 9.6% conversion
test.update('B', conversions=588, trials=5000)  # 11.76% conversion

# Results
prob_b_wins = test.prob_b_better_than_a()
print(f"P(B > A | data) = {prob_b_wins:.4f}")

if prob_b_wins > 0.95:
    print("→ Strong evidence that B is better")
elif prob_b_wins > 0.90:
    print("→ Moderate evidence that B is better")
elif prob_b_wins < 0.10:
    print("→ Strong evidence that A is better")
else:
    print("→ Inconclusive")

# Expected loss
losses = test.expected_loss()
print(f"\nExpected Loss:")
print(f"  If choose A: {losses['A']:.6f} (loss in conversion rate)")
print(f"  If choose B: {losses['B']:.6f}")

# Credible intervals
ci_a = test.credible_interval('A')
ci_b = test.credible_interval('B')
print(f"\n95% Credible Intervals:")
print(f"  A: [{ci_a[0]:.4f}, {ci_a[1]:.4f}]")
print(f"  B: [{ci_b[0]:.4f}, {ci_b[1]:.4f}]")

# Visualize posterior distributions
samples_a = test.sample_posterior('A', 10000)
samples_b = test.sample_posterior('B', 10000)

plt.figure(figsize=(12, 6))
plt.hist(samples_a, bins=50, alpha=0.5, label='Variant A', density=True)
plt.hist(samples_b, bins=50, alpha=0.5, label='Variant B', density=True)
plt.xlabel('Conversion Rate')
plt.ylabel('Density')
plt.title(f'Posterior Distributions (P(B > A) = {prob_b_wins:.3f})')
plt.legend()
plt.grid(True, alpha=0.3)
plt.savefig('bayesian_posteriors.png', dpi=300, bbox_inches='tight')
```

## Informative Priors

Use historical data to set priors:

```python
# Historical data: 10% conversion from 10,000 users
historical_conversions = 1000
historical_trials = 10000

# Set prior based on historical data
test_informed = BayesianABTest(
    prior_alpha=historical_conversions,
    prior_beta=historical_trials - historical_conversions
)

# This prior means: "We believe conversion rate is ~10%"
# But we're still open to updating based on new data

# Compare uninformed vs informed prior
test_uninformed = BayesianABTest(prior_alpha=1, prior_beta=1)

# Small amount of new data
test_informed.update('A', conversions=5, trials=50)
test_uninformed.update('A', conversions=5, trials=50)

# Posteriors will be different
print("With informed prior:")
alpha_i, beta_i = test_informed.get_posterior('A')
print(f"  Posterior mean: {alpha_i / (alpha_i + beta_i):.4f}")

print("\nWith uninformed prior:")
alpha_u, beta_u = test_uninformed.get_posterior('A')
print(f"  Posterior mean: {alpha_u / (alpha_u + beta_u):.4f}")
```

## Normal Model for Continuous Metrics

For revenue, time on site, etc.:

```python
import numpy as np
from scipy import stats

class BayesianABTestContinuous:
    """
    Bayesian A/B test for continuous metrics using Normal model
    """
    def __init__(self):
        self.data = {
            'A': [],
            'B': []
        }
    
    def update(self, variant, observations):
        """Add observations"""
        self.data[variant].extend(observations)
    
    def get_posterior_params(self, variant):
        """
        Get posterior parameters for Normal-Gamma model
        Assumes unknown mean and variance
        """
        data = np.array(self.data[variant])
        n = len(data)
        
        if n == 0:
            return None
        
        mean = data.mean()
        std = data.std(ddof=1)
        se = std / np.sqrt(n)
        
        # Use t-distribution for posterior (conjugate prior)
        df = n - 1
        
        return {'mean': mean, 'std': std, 'se': se, 'df': df, 'n': n}
    
    def prob_b_better_than_a(self, n_samples=10000):
        """Monte Carlo estimate of P(B > A)"""
        params_a = self.get_posterior_params('A')
        params_b = self.get_posterior_params('B')
        
        # Sample from t-distributions
        samples_a = stats.t.rvs(
            df=params_a['df'], 
            loc=params_a['mean'], 
            scale=params_a['se'],
            size=n_samples
        )
        
        samples_b = stats.t.rvs(
            df=params_b['df'], 
            loc=params_b['mean'], 
            scale=params_b['se'],
            size=n_samples
        )
        
        return (samples_b > samples_a).mean()
    
    def credible_interval(self, variant, alpha=0.05):
        """Calculate credible interval"""
        params = self.get_posterior_params(variant)
        
        lower = stats.t.ppf(
            alpha/2, 
            df=params['df'], 
            loc=params['mean'], 
            scale=params['se']
        )
        
        upper = stats.t.ppf(
            1 - alpha/2, 
            df=params['df'], 
            loc=params['mean'], 
            scale=params['se']
        )
        
        return lower, upper

# Example: Revenue per user
np.random.seed(42)

test_revenue = BayesianABTestContinuous()

# Simulate revenue data
revenue_a = np.random.gamma(shape=2, scale=25, size=500)
revenue_b = np.random.gamma(shape=2, scale=28, size=500)  # 12% higher

test_revenue.update('A', revenue_a)
test_revenue.update('B', revenue_b)

prob_b_wins = test_revenue.prob_b_better_than_a()
print(f"P(B revenue > A revenue | data) = {prob_b_wins:.4f}")

# Credible intervals
ci_a = test_revenue.credible_interval('A')
ci_b = test_revenue.credible_interval('B')

params_a = test_revenue.get_posterior_params('A')
params_b = test_revenue.get_posterior_params('B')

print(f"\nRevenue per User:")
print(f"  A: ${params_a['mean']:.2f}, 95% CI=[${ci_a[0]:.2f}, ${ci_a[1]:.2f}]")
print(f"  B: ${params_b['mean']:.2f}, 95% CI=[${ci_b[0]:.2f}, ${ci_b[1]:.2f}]")
```

## Decision Rules with Loss Functions

Incorporate business costs:

```python
def economic_value_decision(test, cost_of_switching=100, 
                           users_per_day=10000, days_deployed=365):
    """
    Calculate expected economic value of choosing each variant
    
    Parameters:
    -----------
    cost_of_switching : float
        One-time cost to switch to new variant (e.g., engineering time)
    users_per_day : int
        Number of users per day
    days_deployed : int
        How long variant will be deployed
    """
    # Sample from posteriors
    samples_a = test.sample_posterior('A', 10000)
    samples_b = test.sample_posterior('B', 10000)
    
    # Total users over deployment period
    total_users = users_per_day * days_deployed
    
    # Expected conversions
    expected_conv_a = samples_a.mean() * total_users
    expected_conv_b = samples_b.mean() * total_users
    
    # Expected value (assuming $1 per conversion)
    value_stay_a = expected_conv_a
    value_switch_b = expected_conv_b - cost_of_switching
    
    print(f"Expected Value Analysis:")
    print(f"  Stay with A: ${value_stay_a:,.0f}")
    print(f"  Switch to B: ${value_switch_b:,.0f}")
    print(f"  Expected gain: ${value_switch_b - value_stay_a:,.0f}")
    
    # Probability of positive ROI
    gains = (samples_b - samples_a) * total_users - cost_of_switching
    prob_positive_roi = (gains > 0).mean()
    
    print(f"\nProbability of positive ROI: {prob_positive_roi:.2%}")
    
    if prob_positive_roi > 0.90:
        return "SWITCH to B"
    else:
        return "STAY with A"

decision = economic_value_decision(test)
print(f"\nDecision: {decision}")
```

## Sequential Bayesian Testing

Monitor experiment continuously:

```python
def bayesian_stopping_rule(test, threshold=0.95, min_samples=100):
    """
    Stop when P(B > A) > threshold or P(A > B) > threshold
    """
    total_samples_a = test.data['A']['trials']
    total_samples_b = test.data['B']['trials']
    
    if total_samples_a < min_samples or total_samples_b < min_samples:
        return 'CONTINUE', None
    
    prob_b_wins = test.prob_b_better_than_a()
    
    if prob_b_wins > threshold:
        return 'STOP', 'B is better'
    elif prob_b_wins < (1 - threshold):
        return 'STOP', 'A is better'
    else:
        return 'CONTINUE', None

# Simulate sequential testing
test_seq = BayesianABTest()

np.random.seed(42)
batch_size = 100
max_batches = 50

for batch in range(max_batches):
    # Collect data
    conv_a = np.random.binomial(batch_size, 0.10)
    conv_b = np.random.binomial(batch_size, 0.12)
    
    test_seq.update('A', conv_a, batch_size)
    test_seq.update('B', conv_b, batch_size)
    
    # Check stopping rule
    decision, reason = bayesian_stopping_rule(test_seq, threshold=0.95)
    
    n_total = test_seq.data['A']['trials'] + test_seq.data['B']['trials']
    prob_b = test_seq.prob_b_better_than_a()
    
    print(f"Batch {batch+1:2d} (n={n_total:5d}): P(B>A)={prob_b:.3f} - {decision}")
    
    if decision == 'STOP':
        print(f"\n✓ Stopped early: {reason}")
        print(f"  Total samples: {n_total} (vs {max_batches * batch_size * 2} if ran to completion)")
        break
```

## Bayesian vs Frequentist Comparison

```python
import pandas as pd

def compare_approaches(conv_a, n_a, conv_b, n_b):
    """Compare Bayesian and Frequentist results"""
    
    # Frequentist
    from statsmodels.stats.proportion import proportions_ztest
    z_stat, p_value = proportions_ztest([conv_b, conv_a], [n_b, n_a])
    
    # Bayesian
    test = BayesianABTest()
    test.update('A', conv_a, n_a)
    test.update('B', conv_b, n_b)
    prob_b_wins = test.prob_b_better_than_a()
    
    results = pd.DataFrame({
        'Approach': ['Frequentist', 'Bayesian'],
        'Metric': ['P-value', 'P(B > A)'],
        'Value': [p_value, prob_b_wins],
        'Decision Threshold': [0.05, 0.95],
        'Interpretation': [
            'Reject H₀' if p_value < 0.05 else 'Fail to reject H₀',
            'B is better' if prob_b_wins > 0.95 else 'Inconclusive'
        ]
    })
    
    return results

# Example
results = compare_approaches(conv_a=480, n_a=5000, conv_b=588, n_b=5000)
print(results.to_string(index=False))
```

## Multi-Variant Bayesian Testing

```python
class BayesianMultiVariantTest:
    """Bayesian test for multiple variants"""
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
        
        # For each sample, find which variant is best
        samples_array = np.array([samples[v] for v in self.variants])
        best_indices = np.argmax(samples_array, axis=0)
        
        # Calculate probability each is best
        probs = {}
        for i, variant in enumerate(self.variants):
            probs[variant] = (best_indices == i).mean()
        
        return probs

# Example with 3 variants
test_multi = BayesianMultiVariantTest(['A', 'B', 'C'])

test_multi.update('A', 95, 1000)   # 9.5%
test_multi.update('B', 115, 1000)  # 11.5%
test_multi.update('C', 105, 1000)  # 10.5%

probs = test_multi.prob_best()

print("Probability each variant is best:")
for variant, prob in sorted(probs.items(), key=lambda x: x[1], reverse=True):
    print(f"  {variant}: {prob:.2%}")
```

## Summary

**When to Use Bayesian A/B Testing:**
- Want interpretable probabilities (not p-values)
- Need to make decisions with business context (loss functions)
- Sequential testing without inflating error rates
- Have informative prior knowledge
- Stakeholders prefer "probability B is better" over "reject null hypothesis"

**Key Differences:**
- Frequentist: Fixed sample size, p-values, hypothesis tests
- Bayesian: Flexible stopping, probabilities, decision-focused

**Implementation:**
- Binary metrics: Beta-Binomial model
- Continuous metrics: Normal model with t-distribution
- Stop when P(B > A) > 95% or economic value is clear
