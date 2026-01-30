---
name: regression-analysis
description: Provides expert guidance for building and validating regression models including linear regression, logistic regression, and time series analysis using statsmodels and scikit-learn for predictive modeling
version: 1.0.0
author: Orchestra Research
license: MIT
tags: [Statistical Analysis, Regression, Linear Regression, Logistic Regression, Time Series, Statsmodels, Scikit-Learn, Predictive Modeling]
dependencies: [statsmodels>=0.14.0, scikit-learn>=1.3.0, scipy>=1.11.0, pandas>=2.0.0, numpy>=1.24.0]
---

# Regression Analysis

Expert-level guidance for building, diagnosing, and validating regression models in Python. This skill covers linear regression, logistic regression, regularization, and time series regression using statsmodels and scikit-learn.

## When to Use This Skill

Use regression analysis when you need to:
- **Predict continuous outcomes** (linear regression) from one or more predictors
- **Model binary outcomes** (logistic regression) for classification tasks
- **Quantify relationships** between variables with statistical inference
- **Control for confounders** in observational studies
- **Forecast time series** data with trend and seasonality

**Do NOT use regression for:**
- Non-linear relationships without transformations (use GAMs or tree-based models)
- When predictors are highly collinear (use regularization or PCA first)
- Causal inference without proper experimental/quasi-experimental design
- When sample size < 10-20 observations per predictor (overfitting risk)

---

## Core Concepts

### 1. Linear Regression Fundamentals

**Model**: y = β₀ + β₁x₁ + β₂x₂ + ... + βₚxₚ + ε

**Key Assumptions:**
1. **Linearity**: Relationship between X and y is linear
2. **Independence**: Observations are independent
3. **Homoscedasticity**: Constant variance of residuals
4. **Normality**: Residuals are normally distributed
5. **No multicollinearity**: Predictors are not highly correlated

**Interpretation:**
- β₀ (intercept): Expected y when all x = 0
- βᵢ (slope): Change in y for 1-unit increase in xᵢ, holding others constant
- R²: Proportion of variance in y explained by model (0-1)
- Adjusted R²: R² penalized for number of predictors

### 2. Logistic Regression Fundamentals

**Model**: log(p/(1-p)) = β₀ + β₁x₁ + ... + βₚxₚ

**Key Metrics:**
- **Odds Ratio**: exp(β) = multiplicative change in odds for 1-unit increase in x
- **AUC-ROC**: Area under ROC curve (0.5 = random, 1.0 = perfect)
- **Accuracy**: Correct predictions / total predictions
- **Precision/Recall**: Tradeoff for imbalanced classes

---

## Workflow 1: Simple Linear Regression

**Use Case**: Predict continuous outcome from single predictor

### Checklist

- [ ] Load and explore data
- [ ] Visualize relationship (scatter plot)
- [ ] Fit linear regression model
- [ ] Check model assumptions (residual plots)
- [ ] Interpret coefficients and significance
- [ ] Calculate predictions and confidence intervals
- [ ] Report R² and model diagnostics

### Implementation

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from scipy import stats
import statsmodels.api as sm
from statsmodels.stats.outliers_influence import variance_inflation_factor

# Step 1: Generate/load data
np.random.seed(42)
n = 100
x = np.random.uniform(0, 10, n)
y = 2.5 * x + 10 + np.random.normal(0, 5, n)

df = pd.DataFrame({'x': x, 'y': y})

# Step 2: Visualize relationship
plt.figure(figsize=(10, 6))
plt.scatter(df['x'], df['y'], alpha=0.6)
plt.xlabel('X (Predictor)')
plt.ylabel('Y (Outcome)')
plt.title('Scatter Plot: Y vs X')
plt.grid(True, alpha=0.3)
plt.savefig('scatter_plot.png', dpi=300, bbox_inches='tight')

# Calculate correlation
r, p_value = stats.pearsonr(df['x'], df['y'])
print(f"Pearson correlation: r={r:.4f}, p={p_value:.4f}")

# Step 3: Fit linear regression with statsmodels (for full inference)
X = sm.add_constant(df['x'])  # Add intercept
model = sm.OLS(df['y'], X)
results = model.fit()

# Print detailed summary
print("\n" + "="*70)
print(results.summary())
print("="*70)

# Step 4: Extract key statistics
print(f"\nKey Results:")
print(f"  Intercept (β₀): {results.params[0]:.4f} ± {results.bse[0]:.4f}")
print(f"  Slope (β₁): {results.params[1]:.4f} ± {results.bse[1]:.4f}")
print(f"  R-squared: {results.rsquared:.4f}")
print(f"  Adj. R-squared: {results.rsquared_adj:.4f}")
print(f"  F-statistic: {results.fvalue:.4f}, p={results.f_pvalue:.4f}")

# Confidence intervals
conf_int = results.conf_int(alpha=0.05)
print(f"\n95% Confidence Intervals:")
print(f"  Intercept: [{conf_int.iloc[0, 0]:.4f}, {conf_int.iloc[0, 1]:.4f}]")
print(f"  Slope: [{conf_int.iloc[1, 0]:.4f}, {conf_int.iloc[1, 1]:.4f}]")

# Step 5: Check assumptions with diagnostic plots
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

# Fitted vs Residuals (check homoscedasticity)
residuals = results.resid
fitted = results.fittedvalues

axes[0, 0].scatter(fitted, residuals, alpha=0.6)
axes[0, 0].axhline(y=0, color='r', linestyle='--')
axes[0, 0].set_xlabel('Fitted Values')
axes[0, 0].set_ylabel('Residuals')
axes[0, 0].set_title('Residuals vs Fitted')
axes[0, 0].grid(True, alpha=0.3)

# Q-Q plot (check normality)
sm.qqplot(residuals, line='s', ax=axes[0, 1])
axes[0, 1].set_title('Normal Q-Q Plot')

# Scale-Location plot (check homoscedasticity)
standardized_resid = np.sqrt(np.abs(residuals / np.std(residuals)))
axes[1, 0].scatter(fitted, standardized_resid, alpha=0.6)
axes[1, 0].set_xlabel('Fitted Values')
axes[1, 0].set_ylabel('√|Standardized Residuals|')
axes[1, 0].set_title('Scale-Location Plot')
axes[1, 0].grid(True, alpha=0.3)

# Histogram of residuals
axes[1, 1].hist(residuals, bins=20, edgecolor='black', alpha=0.7)
axes[1, 1].set_xlabel('Residuals')
axes[1, 1].set_ylabel('Frequency')
axes[1, 1].set_title('Histogram of Residuals')
axes[1, 1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('diagnostic_plots.png', dpi=300, bbox_inches='tight')

# Formal tests for assumptions
print("\n" + "="*70)
print("ASSUMPTION TESTS:")
print("="*70)

# Normality test (Shapiro-Wilk)
shapiro_stat, shapiro_p = stats.shapiro(residuals)
print(f"\nNormality (Shapiro-Wilk): W={shapiro_stat:.4f}, p={shapiro_p:.4f}")
if shapiro_p > 0.05:
    print("  ✓ Residuals are approximately normal")
else:
    print("  ✗ Residuals may not be normal")

# Homoscedasticity test (Breusch-Pagan)
from statsmodels.stats.diagnostic import het_breuschpagan
bp_stat, bp_p, _, _ = het_breuschpagan(residuals, X)
print(f"\nHomoscedasticity (Breusch-Pagan): LM={bp_stat:.4f}, p={bp_p:.4f}")
if bp_p > 0.05:
    print("  ✓ Homoscedasticity assumption satisfied")
else:
    print("  ✗ Heteroscedasticity detected")

# Step 6: Make predictions with confidence intervals
new_x = np.array([2, 5, 8])
new_X = sm.add_constant(new_x)

predictions = results.get_prediction(new_X)
pred_summary = predictions.summary_frame(alpha=0.05)

print(f"\nPredictions for x = {new_x}:")
print(pred_summary[['mean', 'mean_ci_lower', 'mean_ci_upper']])
```

---

## Workflow 2: Multiple Linear Regression

**Use Case**: Predict outcome from multiple predictors

### Checklist

- [ ] Load data with multiple predictors
- [ ] Check for multicollinearity (VIF)
- [ ] Fit multiple regression model
- [ ] Perform feature selection (if needed)
- [ ] Validate on holdout set
- [ ] Interpret coefficients in context

### Implementation

```python
import numpy as np
import pandas as pd
import statsmodels.api as sm
from statsmodels.stats.outliers_influence import variance_inflation_factor
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

# Step 1: Generate multivariate data
np.random.seed(42)
n = 200

x1 = np.random.normal(50, 10, n)
x2 = np.random.normal(30, 5, n)
x3 = x1 * 0.3 + np.random.normal(0, 5, n)  # Correlated with x1
y = 2*x1 + 3*x2 - 1.5*x3 + 100 + np.random.normal(0, 15, n)

df = pd.DataFrame({
    'x1': x1,
    'x2': x2,
    'x3': x3,
    'y': y
})

# Step 2: Check multicollinearity with VIF
X = df[['x1', 'x2', 'x3']]
X_with_const = sm.add_constant(X)

print("Variance Inflation Factors (VIF):")
print("="*40)
for i, col in enumerate(X.columns):
    vif = variance_inflation_factor(X_with_const.values, i+1)
    print(f"  {col}: {vif:.2f}")
    if vif > 10:
        print(f"    ✗ High multicollinearity (VIF > 10)")
    elif vif > 5:
        print(f"    ⚠ Moderate multicollinearity (VIF > 5)")
    else:
        print(f"    ✓ Low multicollinearity")

# Step 3: Fit multiple regression
model = sm.OLS(df['y'], X_with_const)
results = model.fit()

print("\n" + results.summary().as_text())

# Step 4: Feature selection with backward elimination
def backward_elimination(X, y, significance_level=0.05):
    """
    Remove features one-by-one based on p-values
    """
    X_with_const = sm.add_constant(X)
    features = list(X.columns)
    
    while True:
        model = sm.OLS(y, X_with_const)
        results = model.fit()
        
        # Find feature with highest p-value
        p_values = results.pvalues[1:]  # Exclude intercept
        max_p = p_values.max()
        
        if max_p > significance_level:
            # Remove feature with highest p-value
            remove_feature = p_values.idxmax()
            features.remove(remove_feature)
            print(f"Removing {remove_feature} (p={max_p:.4f})")
            
            X_with_const = X_with_const.drop(columns=[remove_feature])
        else:
            break
    
    return features, results

selected_features, final_model = backward_elimination(X, df['y'])
print(f"\nSelected features: {selected_features}")
print(f"Final R²: {final_model.rsquared:.4f}")

# Step 5: Train/test split validation
X_train, X_test, y_train, y_test = train_test_split(
    df[['x1', 'x2', 'x3']], df['y'], test_size=0.2, random_state=42
)

# Fit on training set
X_train_const = sm.add_constant(X_train)
train_model = sm.OLS(y_train, X_train_const).fit()

# Predict on test set
X_test_const = sm.add_constant(X_test)
y_pred = train_model.predict(X_test_const)

# Calculate test metrics
from sklearn.metrics import mean_squared_error, r2_score, mean_absolute_error

test_r2 = r2_score(y_test, y_pred)
test_rmse = np.sqrt(mean_squared_error(y_test, y_pred))
test_mae = mean_absolute_error(y_test, y_pred)

print(f"\nTest Set Performance:")
print(f"  R²: {test_r2:.4f}")
print(f"  RMSE: {test_rmse:.4f}")
print(f"  MAE: {test_mae:.4f}")

# Compare to training performance
print(f"\nTrain vs Test R²:")
print(f"  Training R²: {train_model.rsquared:.4f}")
print(f"  Test R²: {test_r2:.4f}")
print(f"  Difference: {abs(train_model.rsquared - test_r2):.4f}")

if abs(train_model.rsquared - test_r2) > 0.1:
    print("  ⚠ Possible overfitting")
else:
    print("  ✓ Model generalizes well")
```

---

## Workflow 3: Logistic Regression (Binary Classification)

**Use Case**: Predict binary outcome (0/1, Yes/No)

### Checklist

- [ ] Load data with binary outcome
- [ ] Handle class imbalance (if present)
- [ ] Fit logistic regression
- [ ] Interpret odds ratios
- [ ] Evaluate with ROC curve and AUC
- [ ] Choose optimal threshold
- [ ] Report classification metrics

### Implementation

```python
import numpy as np
import pandas as pd
import statsmodels.api as sm
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import (classification_report, confusion_matrix, 
                             roc_curve, roc_auc_score, accuracy_score)
import matplotlib.pyplot as plt
import seaborn as sns

# Step 1: Generate binary classification data
np.random.seed(42)
n = 500

x1 = np.random.normal(0, 1, n)
x2 = np.random.normal(0, 1, n)

# Logistic relationship
z = -1 + 2*x1 + 1.5*x2
prob = 1 / (1 + np.exp(-z))
y = (np.random.random(n) < prob).astype(int)

df = pd.DataFrame({
    'x1': x1,
    'x2': x2,
    'y': y
})

print(f"Class distribution:")
print(df['y'].value_counts())
print(f"Class balance: {df['y'].mean():.2%}")

# Step 2: Split data
X_train, X_test, y_train, y_test = train_test_split(
    df[['x1', 'x2']], df['y'], test_size=0.2, random_state=42, stratify=df['y']
)

# Step 3: Fit with statsmodels (for full inference)
X_train_const = sm.add_constant(X_train)
logit_model = sm.Logit(y_train, X_train_const)
results = logit_model.fit()

print("\n" + results.summary().as_text())

# Step 4: Interpret odds ratios
odds_ratios = np.exp(results.params)
conf_int = np.exp(results.conf_int())

print("\nOdds Ratios (95% CI):")
print("="*50)
for i, var in enumerate(X_train_const.columns):
    if var == 'const':
        continue
    or_val = odds_ratios[var]
    ci_low = conf_int.loc[var, 0]
    ci_high = conf_int.loc[var, 1]
    
    print(f"{var}: OR={or_val:.3f}, 95% CI=[{ci_low:.3f}, {ci_high:.3f}]")
    
    if ci_low > 1:
        print(f"  → 1-unit increase in {var} associated with " 
              f"{(or_val-1)*100:.1f}% increase in odds (p<0.05)")
    elif ci_high < 1:
        print(f"  → 1-unit increase in {var} associated with " 
              f"{(1-or_val)*100:.1f}% decrease in odds (p<0.05)")

# Step 5: Predict on test set
X_test_const = sm.add_constant(X_test)
y_pred_prob = results.predict(X_test_const)
y_pred = (y_pred_prob > 0.5).astype(int)

# Step 6: Evaluate with multiple metrics
print("\nClassification Report:")
print("="*50)
print(classification_report(y_test, y_pred, target_names=['Class 0', 'Class 1']))

# Confusion matrix
cm = confusion_matrix(y_test, y_pred)
plt.figure(figsize=(8, 6))
sns.heatmap(cm, annot=True, fmt='d', cmap='Blues')
plt.xlabel('Predicted')
plt.ylabel('Actual')
plt.title('Confusion Matrix')
plt.savefig('confusion_matrix.png', dpi=300, bbox_inches='tight')

# Step 7: ROC curve and AUC
fpr, tpr, thresholds = roc_curve(y_test, y_pred_prob)
auc = roc_auc_score(y_test, y_pred_prob)

plt.figure(figsize=(10, 8))
plt.plot(fpr, tpr, linewidth=2, label=f'ROC Curve (AUC={auc:.3f})')
plt.plot([0, 1], [0, 1], 'k--', label='Random Classifier')
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('ROC Curve')
plt.legend()
plt.grid(True, alpha=0.3)
plt.savefig('roc_curve.png', dpi=300, bbox_inches='tight')

print(f"\nAUC-ROC: {auc:.4f}")

# Step 8: Find optimal threshold (maximize Youden's J)
j_scores = tpr - fpr
optimal_idx = np.argmax(j_scores)
optimal_threshold = thresholds[optimal_idx]

print(f"Optimal threshold: {optimal_threshold:.4f}")
print(f"  TPR at optimal: {tpr[optimal_idx]:.4f}")
print(f"  FPR at optimal: {fpr[optimal_idx]:.4f}")

# Re-classify with optimal threshold
y_pred_optimal = (y_pred_prob > optimal_threshold).astype(int)
print(f"\nAccuracy with default threshold (0.5): {accuracy_score(y_test, y_pred):.4f}")
print(f"Accuracy with optimal threshold ({optimal_threshold:.4f}): "
      f"{accuracy_score(y_test, y_pred_optimal):.4f}")
```

---

## Workflow 4: Regularized Regression (Ridge/Lasso)

**Use Case**: Prevent overfitting with many predictors

### Implementation

```python
from sklearn.linear_model import Ridge, Lasso, ElasticNet
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import cross_val_score, GridSearchCV
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

# Generate high-dimensional data
np.random.seed(42)
n, p = 100, 50  # 50 predictors, only 5 are truly predictive

X = np.random.randn(n, p)
true_coef = np.zeros(p)
true_coef[:5] = [3, -2, 1.5, -1, 2.5]  # Only first 5 matter

y = X @ true_coef + np.random.randn(n) * 0.5

# Standardize features
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# Ridge Regression (L2 penalty)
ridge = Ridge(alpha=1.0)
ridge_scores = cross_val_score(ridge, X_scaled, y, cv=5, 
                               scoring='neg_mean_squared_error')
print(f"Ridge CV RMSE: {np.sqrt(-ridge_scores.mean()):.4f} ± {np.sqrt(ridge_scores.std()):.4f}")

# Lasso Regression (L1 penalty, induces sparsity)
lasso = Lasso(alpha=0.1)
lasso_scores = cross_val_score(lasso, X_scaled, y, cv=5,
                               scoring='neg_mean_squared_error')
print(f"Lasso CV RMSE: {np.sqrt(-lasso_scores.mean()):.4f} ± {np.sqrt(lasso_scores.std()):.4f}")

# Tune alpha with GridSearchCV
param_grid = {'alpha': np.logspace(-4, 1, 20)}

ridge_cv = GridSearchCV(Ridge(), param_grid, cv=5, scoring='neg_mean_squared_error')
ridge_cv.fit(X_scaled, y)

lasso_cv = GridSearchCV(Lasso(max_iter=10000), param_grid, cv=5, 
                        scoring='neg_mean_squared_error')
lasso_cv.fit(X_scaled, y)

print(f"\nOptimal Ridge alpha: {ridge_cv.best_params_['alpha']:.4f}")
print(f"Optimal Lasso alpha: {lasso_cv.best_params_['alpha']:.4f}")

# Compare coefficient sparsity
ridge_best = ridge_cv.best_estimator_
lasso_best = lasso_cv.best_estimator_

print(f"\nNumber of non-zero coefficients:")
print(f"  Ridge: {np.sum(np.abs(ridge_best.coef_) > 0.01)}")
print(f"  Lasso: {np.sum(np.abs(lasso_best.coef_) > 0.01)}")

# Visualize coefficient paths
alphas = np.logspace(-4, 1, 100)
ridge_coefs = []
lasso_coefs = []

for alpha in alphas:
    ridge_coefs.append(Ridge(alpha=alpha).fit(X_scaled, y).coef_)
    lasso_coefs.append(Lasso(alpha=alpha, max_iter=10000).fit(X_scaled, y).coef_)

ridge_coefs = np.array(ridge_coefs)
lasso_coefs = np.array(lasso_coefs)

fig, axes = plt.subplots(1, 2, figsize=(15, 5))

# Ridge path
for i in range(5):  # Plot first 5 coefficients
    axes[0].plot(alphas, ridge_coefs[:, i], label=f'β{i+1}')
axes[0].set_xscale('log')
axes[0].set_xlabel('Alpha (Regularization Strength)')
axes[0].set_ylabel('Coefficient Value')
axes[0].set_title('Ridge Regularization Path')
axes[0].legend()
axes[0].grid(True, alpha=0.3)

# Lasso path
for i in range(5):
    axes[1].plot(alphas, lasso_coefs[:, i], label=f'β{i+1}')
axes[1].set_xscale('log')
axes[1].set_xlabel('Alpha (Regularization Strength)')
axes[1].set_ylabel('Coefficient Value')
axes[1].set_title('Lasso Regularization Path (Sparse)')
axes[1].legend()
axes[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('regularization_paths.png', dpi=300, bbox_inches='tight')
```

---

## Common Issues and Solutions

### Issue 1: Multicollinearity

**Problem**: Correlated predictors inflate coefficient standard errors

**Solution**: Use VIF, remove correlated features, or use regularization

```python
from statsmodels.stats.outliers_influence import variance_inflation_factor
import pandas as pd
import numpy as np

# Check VIF for all features
def calculate_vif(df, features):
    vif_data = pd.DataFrame()
    vif_data["Feature"] = features
    vif_data["VIF"] = [variance_inflation_factor(df[features].values, i) 
                       for i in range(len(features))]
    return vif_data.sort_values('VIF', ascending=False)

# Example
features = ['x1', 'x2', 'x3']
vif_df = calculate_vif(df, features)
print(vif_df)

# Remove features with VIF > 10
high_vif = vif_df[vif_df['VIF'] > 10]['Feature'].tolist()
print(f"\nRemove: {high_vif}")
```

### Issue 2: Heteroscedasticity

**Problem**: Non-constant variance violates OLS assumptions

**Solution**: Use robust standard errors or transform outcome

```python
import statsmodels.api as sm

# Fit with robust standard errors (HC3)
X_const = sm.add_constant(X)
model = sm.OLS(y, X_const)
robust_results = model.fit(cov_type='HC3')

print(robust_results.summary())

# Alternative: Log-transform outcome
import numpy as np
y_log = np.log(y + 1)  # +1 if y can be zero
model_log = sm.OLS(y_log, X_const).fit()
```

### Issue 3: Overfitting

**Problem**: Model fits training data but fails on test data

**Solution**: Use cross-validation, regularization, or reduce features

```python
from sklearn.model_selection import cross_val_score
from sklearn.linear_model import Ridge

# Cross-validation
cv_scores = cross_val_score(Ridge(alpha=1.0), X_scaled, y, cv=10)
print(f"CV R²: {cv_scores.mean():.4f} ± {cv_scores.std():.4f}")

# If train R² >> CV R²: overfitting
```

---

## When to Use vs Alternatives

| Use Regression | Use Alternative |
|----------------|----------------|
| Linear relationships, need inference | Complex non-linear patterns → Use tree-based models (Random Forest, XGBoost) |
| Interpretable coefficients required | Black-box predictions acceptable → Use neural networks |
| Statistical significance testing | Pure prediction focus → Use cross-validated ML |
| Small-medium datasets (n < 10K) | Very large datasets → Use scalable ML frameworks |
| Stable relationships | Time-varying relationships → Use time series models (ARIMA, Prophet) |

---

## Advanced Features

**Polynomial Regression**: See [references/polynomial-regression.md](references/polynomial-regression.md)
**Time Series Regression**: See [references/time-series-regression.md](references/time-series-regression.md)
**Generalized Linear Models**: See [references/glm.md](references/glm.md)

---

## Quick Reference

```python
# Linear regression (statsmodels)
import statsmodels.api as sm
X_const = sm.add_constant(X)
model = sm.OLS(y, X_const).fit()
print(model.summary())

# Linear regression (scikit-learn)
from sklearn.linear_model import LinearRegression
model = LinearRegression().fit(X, y)
print(f"R²: {model.score(X, y):.4f}")

# Logistic regression
from sklearn.linear_model import LogisticRegression
clf = LogisticRegression().fit(X, y)
y_pred_prob = clf.predict_proba(X)[:, 1]

# Ridge/Lasso
from sklearn.linear_model import Ridge, Lasso
ridge = Ridge(alpha=1.0).fit(X, y)
lasso = Lasso(alpha=0.1).fit(X, y)
```

---

## Summary

This skill provides production-ready workflows for:
- **Simple and multiple linear regression** with full diagnostic checks
- **Logistic regression** for binary classification with odds ratios
- **Regularized regression** (Ridge/Lasso) for high-dimensional data
- **Model validation** using train/test splits and cross-validation

**Key Principle**: Always validate assumptions, check for overfitting, and report both statistical significance (p-values) and practical significance (effect sizes, R²).
