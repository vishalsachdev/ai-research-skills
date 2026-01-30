# Generalized Linear Models (GLMs)

## Overview

Generalized Linear Models extend linear regression to non-normal response distributions via link functions. GLMs unify many regression types under a common framework.

## GLM Components

1. **Random Component**: Distribution of Y (Normal, Binomial, Poisson, Gamma, etc.)
2. **Systematic Component**: Linear predictor η = β₀ + β₁X₁ + ... + βₚXₚ
3. **Link Function**: Connects E(Y) to η via g(μ) = η

## Common GLM Families

| Family | Distribution | Link | Use Case |
|--------|--------------|------|----------|
| Gaussian | Normal | Identity | Continuous outcomes (standard linear regression) |
| Binomial | Binomial | Logit | Binary/proportion outcomes |
| Poisson | Poisson | Log | Count data |
| Gamma | Gamma | Log | Positive continuous, right-skewed |
| Inverse Gaussian | Inv. Gaussian | 1/μ² | Skewed positive data |

## Poisson Regression (Count Data)

For non-negative integer outcomes:

```python
import statsmodels.api as sm
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

# Generate count data
np.random.seed(42)
n = 200

X1 = np.random.uniform(0, 10, n)
X2 = np.random.normal(0, 1, n)

# Poisson process: log(λ) = β₀ + β₁X₁ + β₂X₂
log_lambda = -1 + 0.5*X1 + 0.3*X2
lambda_param = np.exp(log_lambda)
y = np.random.poisson(lambda_param)

df = pd.DataFrame({
    'y': y,
    'X1': X1,
    'X2': X2
})

print(f"Count data statistics:")
print(f"  Mean: {df['y'].mean():.2f}")
print(f"  Variance: {df['y'].var():.2f}")
print(f"  Zero counts: {(df['y'] == 0).sum()}")

# Fit Poisson regression
X = sm.add_constant(df[['X1', 'X2']])
poisson_model = sm.GLM(df['y'], X, family=sm.families.Poisson())
poisson_results = poisson_model.fit()

print("\n" + poisson_results.summary().as_text())

# Interpret coefficients (exponentiated = incidence rate ratios)
print("\nIncidence Rate Ratios (IRR):")
for var, coef in poisson_results.params.items():
    if var == 'const':
        continue
    irr = np.exp(coef)
    ci = np.exp(poisson_results.conf_int().loc[var])
    print(f"{var}: IRR={irr:.3f}, 95% CI=[{ci[0]:.3f}, {ci[1]:.3f}]")
    print(f"  → 1-unit increase in {var} multiplies expected count by {irr:.3f}")

# Check overdispersion
pearson_chi2 = poisson_results.pearson_chi2
df_resid = poisson_results.df_resid
dispersion = pearson_chi2 / df_resid

print(f"\nDispersion parameter: {dispersion:.3f}")
if dispersion > 1.5:
    print("⚠ Overdispersion detected - consider Negative Binomial model")
else:
    print("✓ No overdispersion")
```

## Negative Binomial Regression (Overdispersed Counts)

When variance > mean in count data:

```python
import statsmodels.api as sm
import numpy as np

# Generate overdispersed count data
np.random.seed(42)
n = 200
X = np.random.uniform(0, 10, n)

# Negative binomial process (overdispersed)
mu = np.exp(-1 + 0.5*X)
alpha = 2  # Dispersion parameter
p = 1 / (1 + alpha * mu)
r = mu / alpha
y = np.random.negative_binomial(r, p)

X_const = sm.add_constant(X)

# Fit Poisson (misspecified)
poisson_model = sm.GLM(y, X_const, family=sm.families.Poisson())
poisson_results = poisson_model.fit()

# Fit Negative Binomial (correct)
nb_model = sm.GLM(y, X_const, family=sm.families.NegativeBinomial())
nb_results = nb_model.fit()

print("Model Comparison:")
print(f"  Poisson AIC: {poisson_results.aic:.2f}")
print(f"  Negative Binomial AIC: {nb_results.aic:.2f}")

if nb_results.aic < poisson_results.aic:
    print("  → Negative Binomial fits better (lower AIC)")
```

## Gamma Regression (Positive Continuous Data)

For right-skewed positive outcomes:

```python
import statsmodels.api as sm
import numpy as np
import matplotlib.pyplot as plt

# Generate gamma-distributed data (e.g., insurance claims, survival times)
np.random.seed(42)
n = 200
X = np.random.uniform(0, 10, n)

# Gamma regression: log(μ) = β₀ + β₁X
log_mu = 2 + 0.3*X
mu = np.exp(log_mu)
shape = 4  # Shape parameter
scale = mu / shape
y = np.random.gamma(shape, scale)

X_const = sm.add_constant(X)

# Fit Gamma GLM with log link
gamma_model = sm.GLM(y, X_const, family=sm.families.Gamma())
gamma_results = gamma_model.fit()

print(gamma_results.summary())

# Plot fit
X_plot = np.linspace(0, 10, 100)
X_plot_const = sm.add_constant(X_plot)
y_pred = gamma_results.predict(X_plot_const)

plt.figure(figsize=(10, 6))
plt.scatter(X, y, alpha=0.6, label='Data')
plt.plot(X_plot, y_pred, 'r-', linewidth=2, label='Gamma GLM')
plt.xlabel('X')
plt.ylabel('y (Positive Continuous)')
plt.title('Gamma Regression with Log Link')
plt.legend()
plt.grid(True, alpha=0.3)
plt.savefig('gamma_regression.png', dpi=300, bbox_inches='tight')
```

## Quasi-Poisson (Alternative to Negative Binomial)

```python
import statsmodels.api as sm

# Fit with Quasi-Poisson family
quasipoisson_model = sm.GLM(y, X_const, 
                            family=sm.families.Poisson())
quasipoisson_results = quasipoisson_model.fit(scale='X2')  # Estimate dispersion

print(quasipoisson_results.summary())
print(f"\nEstimated dispersion: {quasipoisson_results.scale:.3f}")
```

## Zero-Inflated Models

When data has excess zeros:

```python
from statsmodels.discrete.count_model import ZeroInflatedPoisson
import numpy as np

# Generate zero-inflated data
np.random.seed(42)
n = 300
X = np.random.uniform(0, 10, n)

# Two-component process:
# 1. Bernoulli: probability of being "always zero" group
# 2. Poisson: count for non-zero group
always_zero_prob = 0.3
always_zero = np.random.binomial(1, always_zero_prob, n)

lambda_param = np.exp(-1 + 0.5*X)
poisson_counts = np.random.poisson(lambda_param)

y = np.where(always_zero, 0, poisson_counts)

print(f"Proportion of zeros: {(y == 0).mean():.2%}")

# Fit Zero-Inflated Poisson
X_const = sm.add_constant(X)
zip_model = ZeroInflatedPoisson(y, X_const, exog_infl=X_const)
zip_results = zip_model.fit()

print("\n" + zip_results.summary().as_text())

# Compare to standard Poisson
poisson_model = sm.GLM(y, X_const, family=sm.families.Poisson())
poisson_results = poisson_model.fit()

print(f"\nModel Comparison:")
print(f"  Standard Poisson AIC: {poisson_results.aic:.2f}")
print(f"  Zero-Inflated Poisson AIC: {zip_results.aic:.2f}")
```

## Ordinal Logistic Regression

For ordered categorical outcomes:

```python
from statsmodels.miscmodels.ordinal_model import OrderedModel
import numpy as np
import pandas as pd

# Generate ordinal data (e.g., survey ratings: 1-5)
np.random.seed(42)
n = 300
X1 = np.random.normal(0, 1, n)
X2 = np.random.normal(0, 1, n)

# Latent continuous variable
z = -1 + 0.8*X1 + 0.5*X2 + np.random.normal(0, 1, n)

# Convert to ordinal categories
y = pd.cut(z, bins=[-np.inf, -1, 0, 1, 2, np.inf], 
           labels=[0, 1, 2, 3, 4]).astype(int)

df = pd.DataFrame({
    'y': y,
    'X1': X1,
    'X2': X2
})

# Fit ordinal logistic regression
model = OrderedModel(df['y'], df[['X1', 'X2']], distr='logit')
results = model.fit(method='bfgs')

print(results.summary())

# Predicted probabilities for each category
probs = results.model.predict(results.params, exog=df[['X1', 'X2']])
print("\nPredicted probabilities (first 5 observations):")
print(pd.DataFrame(probs[:5], columns=[f'P(Y={i})' for i in range(5)]))
```

## Tweedie Regression (Insurance Claims)

For data that's mix of zeros and continuous positive values:

```python
from sklearn.linear_model import TweedieRegressor
import numpy as np
import matplotlib.pyplot as plt

# Generate insurance claim data
np.random.seed(42)
n = 500
X = np.random.uniform(0, 10, n).reshape(-1, 1)

# Many zero claims, some large claims
claim_occurs = np.random.binomial(1, 0.3, n)  # 30% have claims
claim_amount = np.where(claim_occurs, 
                        np.random.gamma(2, 1000*np.exp(0.1*X.ravel())),
                        0)

# Tweedie power parameter p:
# p = 0: Normal
# p = 1: Poisson
# p = 2: Gamma
# 1 < p < 2: Compound Poisson-Gamma (insurance claims)

model = TweedieRegressor(power=1.5, alpha=0.5, max_iter=100)
model.fit(X, claim_amount)

# Predict
X_plot = np.linspace(0, 10, 100).reshape(-1, 1)
y_pred = model.predict(X_plot)

plt.figure(figsize=(10, 6))
plt.scatter(X, claim_amount, alpha=0.4, label='Data')
plt.plot(X_plot, y_pred, 'r-', linewidth=2, label='Tweedie Regression')
plt.xlabel('Risk Factor')
plt.ylabel('Claim Amount')
plt.title('Tweedie Regression for Insurance Claims')
plt.legend()
plt.grid(True, alpha=0.3)
plt.savefig('tweedie_regression.png', dpi=300, bbox_inches='tight')

print(f"Proportion of zero claims: {(claim_amount == 0).mean():.2%}")
print(f"Mean claim (all): ${claim_amount.mean():.2f}")
print(f"Mean claim (non-zero): ${claim_amount[claim_amount > 0].mean():.2f}")
```

## Multinomial Logistic Regression

For unordered multi-class outcomes:

```python
from sklearn.linear_model import LogisticRegression
import numpy as np
import pandas as pd

# Generate 3-class data
np.random.seed(42)
n = 300
X1 = np.random.normal(0, 1, n)
X2 = np.random.normal(0, 1, n)

# Multinomial logit
z1 = -1 + 2*X1 + 0.5*X2
z2 = 0.5 + 0.5*X1 + 2*X2
z3 = 0  # Reference category

exp_z1 = np.exp(z1)
exp_z2 = np.exp(z2)
exp_z3 = np.exp(z3)

prob_class0 = exp_z1 / (exp_z1 + exp_z2 + exp_z3)
prob_class1 = exp_z2 / (exp_z1 + exp_z2 + exp_z3)
prob_class2 = exp_z3 / (exp_z1 + exp_z2 + exp_z3)

# Sample class
rand = np.random.random(n)
y = np.where(rand < prob_class0, 0,
             np.where(rand < prob_class0 + prob_class1, 1, 2))

X = np.column_stack([X1, X2])

# Fit multinomial logistic regression
clf = LogisticRegression(multi_class='multinomial', solver='lbfgs', max_iter=1000)
clf.fit(X, y)

print("Multinomial Logistic Regression Coefficients:")
print("Class 0 vs Class 2 (reference):", clf.coef_[0])
print("Class 1 vs Class 2 (reference):", clf.coef_[1])

# Predict probabilities
probs = clf.predict_proba(X)
print("\nPredicted probabilities (first 5 observations):")
print(pd.DataFrame(probs[:5], columns=['P(Class 0)', 'P(Class 1)', 'P(Class 2)']))
```

## Model Diagnostics for GLMs

```python
import statsmodels.api as sm
import numpy as np
import matplotlib.pyplot as plt

# Fit Poisson model
X_const = sm.add_constant(df[['X1', 'X2']])
model = sm.GLM(df['y'], X_const, family=sm.families.Poisson())
results = model.fit()

# Residual plots
residuals_pearson = results.resid_pearson
residuals_deviance = results.resid_deviance
fitted_values = results.fittedvalues

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Pearson residuals vs fitted
axes[0].scatter(fitted_values, residuals_pearson, alpha=0.6)
axes[0].axhline(y=0, color='r', linestyle='--')
axes[0].set_xlabel('Fitted Values')
axes[0].set_ylabel('Pearson Residuals')
axes[0].set_title('Pearson Residuals vs Fitted')
axes[0].grid(True, alpha=0.3)

# Deviance residuals vs fitted
axes[1].scatter(fitted_values, residuals_deviance, alpha=0.6)
axes[1].axhline(y=0, color='r', linestyle='--')
axes[1].set_xlabel('Fitted Values')
axes[1].set_ylabel('Deviance Residuals')
axes[1].set_title('Deviance Residuals vs Fitted')
axes[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('glm_diagnostics.png', dpi=300, bbox_inches='tight')

# Deviance test
print(f"Residual Deviance: {results.deviance:.2f}")
print(f"Degrees of Freedom: {results.df_resid}")
print(f"Deviance / DF: {results.deviance / results.df_resid:.3f}")

if results.deviance / results.df_resid > 1.5:
    print("⚠ Poor fit or overdispersion")
else:
    print("✓ Reasonable fit")
```

## Choosing Link Functions

Common alternatives for each family:

```python
import statsmodels.api as sm

# For Binomial family
families = {
    'Logit': sm.families.Binomial(link=sm.families.links.logit()),
    'Probit': sm.families.Binomial(link=sm.families.links.probit()),
    'Log-log': sm.families.Binomial(link=sm.families.links.cloglog())
}

for name, family in families.items():
    model = sm.GLM(y_binary, X_const, family=family)
    results = model.fit()
    print(f"{name} AIC: {results.aic:.2f}")
```

## Comparing GLM to Alternatives

| Use GLM | Use Alternative |
|---------|----------------|
| Need interpretable coefficients | Black-box predictions → Random Forest, XGBoost |
| Know response distribution | Unknown distribution → Use flexible ML |
| Inference and p-values required | Prediction only → Deep learning |
| Small-medium data | Very large data → Scalable ML frameworks |

## Summary

**Family Selection Guide:**
- **Gaussian**: Continuous, symmetric outcomes (standard regression)
- **Binomial**: Binary or proportion outcomes (logistic regression)
- **Poisson**: Count data (non-negative integers)
- **Negative Binomial**: Overdispersed counts (variance > mean)
- **Gamma**: Positive continuous, right-skewed
- **Tweedie**: Zero-inflated continuous positive (insurance claims)
- **Ordinal**: Ordered categories (ratings, severity levels)
- **Multinomial**: Unordered categories (multi-class classification)

**Always check:**
1. Residual plots for model fit
2. Overdispersion (for count models)
3. Link function appropriateness
4. Model comparison with AIC/BIC
