# Polynomial and Non-Linear Regression

## Overview

Polynomial regression fits curved relationships by transforming predictors into polynomial terms. This guide covers polynomial features, interaction terms, and alternatives for non-linear patterns.

## Polynomial Regression Basics

Transform linear model y = β₀ + β₁x into polynomial:
y = β₀ + β₁x + β₂x² + β₃x³ + ... + βₙxⁿ

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression
from sklearn.pipeline import Pipeline
from sklearn.model_selection import cross_val_score

# Generate non-linear data
np.random.seed(42)
X = np.random.uniform(-3, 3, 100).reshape(-1, 1)
y = 0.5 * X.ravel()**3 - 2 * X.ravel()**2 + X.ravel() + np.random.normal(0, 5, 100)

# Try different polynomial degrees
degrees = [1, 2, 3, 5, 10]
fig, axes = plt.subplots(2, 3, figsize=(15, 10))
axes = axes.ravel()

for idx, degree in enumerate(degrees):
    # Create polynomial pipeline
    model = Pipeline([
        ('poly', PolynomialFeatures(degree=degree)),
        ('linear', LinearRegression())
    ])
    
    model.fit(X, y)
    
    # Predict on smooth range
    X_plot = np.linspace(-3, 3, 300).reshape(-1, 1)
    y_plot = model.predict(X_plot)
    
    # Cross-validation score
    cv_score = cross_val_score(model, X, y, cv=5, 
                               scoring='neg_mean_squared_error').mean()
    rmse = np.sqrt(-cv_score)
    
    # Plot
    axes[idx].scatter(X, y, alpha=0.6, s=20)
    axes[idx].plot(X_plot, y_plot, 'r-', linewidth=2)
    axes[idx].set_title(f'Degree {degree} (CV RMSE: {rmse:.2f})')
    axes[idx].set_xlabel('X')
    axes[idx].set_ylabel('y')
    axes[idx].grid(True, alpha=0.3)

# Remove empty subplot
fig.delaxes(axes[-1])
plt.tight_layout()
plt.savefig('polynomial_degrees.png', dpi=300, bbox_inches='tight')

print("Cross-Validation RMSE by Degree:")
for degree in degrees:
    model = Pipeline([
        ('poly', PolynomialFeatures(degree=degree)),
        ('linear', LinearRegression())
    ])
    cv_scores = cross_val_score(model, X, y, cv=5, 
                                scoring='neg_mean_squared_error')
    rmse = np.sqrt(-cv_scores.mean())
    print(f"  Degree {degree}: {rmse:.4f}")
```

## Choosing Polynomial Degree

Use cross-validation to avoid overfitting:

```python
from sklearn.model_selection import learning_curve
import numpy as np
import matplotlib.pyplot as plt

def plot_learning_curves(X, y, degree):
    """Plot training and validation curves"""
    model = Pipeline([
        ('poly', PolynomialFeatures(degree=degree)),
        ('linear', LinearRegression())
    ])
    
    train_sizes, train_scores, val_scores = learning_curve(
        model, X, y, 
        train_sizes=np.linspace(0.1, 1.0, 10),
        cv=5, 
        scoring='neg_mean_squared_error'
    )
    
    train_rmse = np.sqrt(-train_scores.mean(axis=1))
    val_rmse = np.sqrt(-val_scores.mean(axis=1))
    
    plt.figure(figsize=(10, 6))
    plt.plot(train_sizes, train_rmse, 'o-', label='Training RMSE')
    plt.plot(train_sizes, val_rmse, 'o-', label='Validation RMSE')
    plt.xlabel('Training Set Size')
    plt.ylabel('RMSE')
    plt.title(f'Learning Curves (Degree {degree})')
    plt.legend()
    plt.grid(True, alpha=0.3)
    plt.savefig(f'learning_curve_deg{degree}.png', dpi=300, bbox_inches='tight')

plot_learning_curves(X, y, degree=3)
```

## Regularized Polynomial Regression

Prevent overfitting with Ridge/Lasso:

```python
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import Ridge, Lasso
from sklearn.pipeline import Pipeline
from sklearn.model_selection import GridSearchCV

# High-degree polynomial with regularization
model = Pipeline([
    ('poly', PolynomialFeatures(degree=10)),
    ('ridge', Ridge())
])

# Tune alpha
param_grid = {'ridge__alpha': np.logspace(-3, 3, 20)}
grid_search = GridSearchCV(model, param_grid, cv=5, 
                           scoring='neg_mean_squared_error')
grid_search.fit(X, y)

print(f"Best alpha: {grid_search.best_params_['ridge__alpha']:.4f}")
print(f"Best CV RMSE: {np.sqrt(-grid_search.best_score_):.4f}")

# Compare to unregularized
unreg_model = Pipeline([
    ('poly', PolynomialFeatures(degree=10)),
    ('linear', LinearRegression())
])
unreg_cv = cross_val_score(unreg_model, X, y, cv=5, 
                           scoring='neg_mean_squared_error')
print(f"Unregularized CV RMSE: {np.sqrt(-unreg_cv.mean()):.4f}")
```

## Interaction Terms

Model multiplicative effects between predictors:

```python
import numpy as np
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression

# Generate data with interaction
np.random.seed(42)
n = 200
X1 = np.random.uniform(0, 10, n)
X2 = np.random.uniform(0, 10, n)

# y depends on X1*X2 interaction
y = 2*X1 + 3*X2 + 0.5*X1*X2 + np.random.normal(0, 5, n)

X = np.column_stack([X1, X2])

# Model without interaction
model_no_int = LinearRegression()
model_no_int.fit(X, y)
r2_no_int = model_no_int.score(X, y)

# Model with interaction (degree=2, interaction_only=True)
poly = PolynomialFeatures(degree=2, interaction_only=True, include_bias=False)
X_int = poly.fit_transform(X)

model_int = LinearRegression()
model_int.fit(X_int, y)
r2_int = model_int.score(X_int, y)

print("Feature names:", poly.get_feature_names_out(['X1', 'X2']))
print(f"\nR² without interaction: {r2_no_int:.4f}")
print(f"R² with interaction: {r2_int:.4f}")
print(f"\nCoefficients with interaction:")
for name, coef in zip(poly.get_feature_names_out(['X1', 'X2']), model_int.coef_):
    print(f"  {name}: {coef:.4f}")
```

## Piecewise Regression (Splines)

Fit different functions in different regions:

```python
import numpy as np
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import PolynomialFeatures
import matplotlib.pyplot as plt

# Generate data with change point
np.random.seed(42)
X = np.linspace(0, 10, 100).reshape(-1, 1)
y = np.where(X.ravel() < 5, 
             2*X.ravel() + np.random.normal(0, 1, 100),
             -1*X.ravel() + 15 + np.random.normal(0, 1, 100))

# Fit piecewise linear regression
knot = 5.0  # Change point

# Create indicator variables
X_left = np.where(X < knot, X - knot, 0)
X_right = np.where(X >= knot, X - knot, 0)

X_piecewise = np.column_stack([X, X_left, X_right])

model = LinearRegression()
model.fit(X_piecewise, y)

# Predict
y_pred = model.predict(X_piecewise)

plt.figure(figsize=(10, 6))
plt.scatter(X, y, alpha=0.6, label='Data')
plt.plot(X, y_pred, 'r-', linewidth=2, label='Piecewise Linear')
plt.axvline(knot, color='k', linestyle='--', alpha=0.5, label=f'Knot at {knot}')
plt.xlabel('X')
plt.ylabel('y')
plt.title('Piecewise Linear Regression')
plt.legend()
plt.grid(True, alpha=0.3)
plt.savefig('piecewise_regression.png', dpi=300, bbox_inches='tight')
```

## B-Splines with Patsy

More flexible spline fitting:

```python
import numpy as np
import pandas as pd
import statsmodels.api as sm
from patsy import dmatrix
import matplotlib.pyplot as plt

# Generate non-linear data
np.random.seed(42)
X = np.linspace(0, 10, 100)
y = np.sin(X) + 0.2*X + np.random.normal(0, 0.3, 100)

df = pd.DataFrame({'X': X, 'y': y})

# Fit B-spline with different degrees of freedom
for df_spline in [4, 6, 10]:
    # Create B-spline basis
    X_spline = dmatrix(f"bs(X, df={df_spline})", df, return_type='dataframe')
    
    # Fit model
    model = sm.OLS(df['y'], X_spline)
    results = model.fit()
    
    # Predict
    y_pred = results.predict(X_spline)
    
    plt.figure(figsize=(10, 6))
    plt.scatter(df['X'], df['y'], alpha=0.6, label='Data')
    plt.plot(df['X'], y_pred, 'r-', linewidth=2, label=f'B-spline (df={df_spline})')
    plt.xlabel('X')
    plt.ylabel('y')
    plt.title(f'B-Spline Regression (df={df_spline})')
    plt.legend()
    plt.grid(True, alpha=0.3)
    plt.savefig(f'bspline_df{df_spline}.png', dpi=300, bbox_inches='tight')
    
    print(f"B-spline (df={df_spline}) R²: {results.rsquared:.4f}")
```

## Generalized Additive Models (GAMs)

Flexible non-linear models with interpretability:

```python
from pygam import LinearGAM, s, f
import numpy as np
import matplotlib.pyplot as plt

# Generate data
np.random.seed(42)
X = np.random.uniform(0, 10, 200).reshape(-1, 1)
y = np.sin(X.ravel()) + 0.1*X.ravel() + np.random.normal(0, 0.3, 200)

# Fit GAM
gam = LinearGAM(s(0, n_splines=10))
gam.gridsearch(X, y)

# Predict
X_plot = np.linspace(0, 10, 300).reshape(-1, 1)
y_pred = gam.predict(X_plot)
y_ci = gam.prediction_intervals(X_plot, width=0.95)

# Plot
plt.figure(figsize=(12, 6))
plt.scatter(X, y, alpha=0.6, label='Data')
plt.plot(X_plot, y_pred, 'r-', linewidth=2, label='GAM')
plt.fill_between(X_plot.ravel(), y_ci[:, 0], y_ci[:, 1], 
                 alpha=0.3, color='red', label='95% CI')
plt.xlabel('X')
plt.ylabel('y')
plt.title('Generalized Additive Model')
plt.legend()
plt.grid(True, alpha=0.3)
plt.savefig('gam_fit.png', dpi=300, bbox_inches='tight')

print(f"GAM R²: {gam.statistics_['pseudo_r2']['explained_deviance']:.4f}")

# Partial dependence plot
fig, ax = plt.subplots(figsize=(10, 6))
XX = gam.generate_X_grid(term=0, n=300)
ax.plot(XX[:, 0], gam.partial_dependence(term=0, X=XX))
ax.plot(XX[:, 0], gam.partial_dependence(term=0, X=XX, width=0.95)[1], 
        c='r', ls='--')
ax.set_xlabel('X')
ax.set_ylabel('Partial Dependence')
ax.set_title('GAM Partial Dependence')
plt.grid(True, alpha=0.3)
plt.savefig('gam_partial_dependence.png', dpi=300, bbox_inches='tight')
```

## Polynomial Logistic Regression

Non-linear decision boundaries for classification:

```python
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LogisticRegression
from sklearn.datasets import make_moons
import matplotlib.pyplot as plt
import numpy as np

# Generate non-linear classification data
X, y = make_moons(n_samples=200, noise=0.2, random_state=42)

# Fit with polynomial features
poly = PolynomialFeatures(degree=3)
X_poly = poly.fit_transform(X)

clf = LogisticRegression(max_iter=1000)
clf.fit(X_poly, y)

# Plot decision boundary
def plot_decision_boundary(X, y, model, poly):
    h = 0.02
    x_min, x_max = X[:, 0].min() - 0.5, X[:, 0].max() + 0.5
    y_min, y_max = X[:, 1].min() - 0.5, X[:, 1].max() + 0.5
    
    xx, yy = np.meshgrid(np.arange(x_min, x_max, h),
                         np.arange(y_min, y_max, h))
    
    Z = model.predict(poly.transform(np.c_[xx.ravel(), yy.ravel()]))
    Z = Z.reshape(xx.shape)
    
    plt.figure(figsize=(10, 6))
    plt.contourf(xx, yy, Z, alpha=0.3, cmap='RdYlBu')
    plt.scatter(X[:, 0], X[:, 1], c=y, cmap='RdYlBu', edgecolors='black')
    plt.xlabel('Feature 1')
    plt.ylabel('Feature 2')
    plt.title('Polynomial Logistic Regression Decision Boundary')
    plt.savefig('poly_logistic_boundary.png', dpi=300, bbox_inches='tight')

plot_decision_boundary(X, y, clf, poly)
print(f"Accuracy: {clf.score(X_poly, y):.4f}")
```

## When to Use Polynomial Regression

**Use polynomial regression when:**
- Relationship is curved but still relatively smooth
- Need interpretable coefficients
- Feature space is low-dimensional (1-3 features)
- Have sufficient data relative to degree chosen

**Use alternatives when:**
- Many predictors → Use tree-based models (Random Forest, XGBoost)
- Complex non-linearities → Use GAMs or neural networks
- Interactions unknown → Use automated feature engineering
- High-dimensional → Use kernel methods (SVM, GP)

## Common Pitfalls

### 1. Overfitting with High Degrees

```python
# Always use cross-validation
from sklearn.model_selection import cross_val_score

degrees_to_try = range(1, 15)
cv_scores = []

for degree in degrees_to_try:
    model = Pipeline([
        ('poly', PolynomialFeatures(degree=degree)),
        ('ridge', Ridge(alpha=1.0))  # Use regularization!
    ])
    cv_score = cross_val_score(model, X, y, cv=5, 
                               scoring='neg_mean_squared_error')
    cv_scores.append(-cv_score.mean())

# Plot CV error vs degree
plt.figure(figsize=(10, 6))
plt.plot(degrees_to_try, cv_scores, 'o-')
plt.xlabel('Polynomial Degree')
plt.ylabel('CV MSE')
plt.title('Cross-Validation Error vs Polynomial Degree')
plt.grid(True, alpha=0.3)
plt.savefig('cv_error_vs_degree.png', dpi=300, bbox_inches='tight')

best_degree = degrees_to_try[np.argmin(cv_scores)]
print(f"Best degree: {best_degree}")
```

### 2. Extrapolation Dangers

Polynomials behave poorly outside training range:

```python
# Train on limited range
X_train = np.random.uniform(0, 5, 50).reshape(-1, 1)
y_train = 2*X_train.ravel() + np.random.normal(0, 1, 50)

# Fit polynomial
model = Pipeline([
    ('poly', PolynomialFeatures(degree=5)),
    ('linear', LinearRegression())
])
model.fit(X_train, y_train)

# Predict on extended range (extrapolation)
X_plot = np.linspace(-2, 10, 300).reshape(-1, 1)
y_plot = model.predict(X_plot)

plt.figure(figsize=(10, 6))
plt.scatter(X_train, y_train, label='Training Data')
plt.plot(X_plot, y_plot, 'r-', label='Polynomial Fit')
plt.axvline(0, color='k', linestyle='--', alpha=0.5, label='Training Range')
plt.axvline(5, color='k', linestyle='--', alpha=0.5)
plt.xlabel('X')
plt.ylabel('y')
plt.title('Polynomial Extrapolation (Dangerous!)')
plt.legend()
plt.grid(True, alpha=0.3)
plt.savefig('polynomial_extrapolation.png', dpi=300, bbox_inches='tight')
```

## Summary

**Model Selection:**
- **Degree 2-3**: Most common, captures mild curvature
- **Degree 4-5**: More complex curves, risk of overfitting
- **Splines/GAMs**: Very flexible, better for complex patterns
- **Always regularize**: Use Ridge/Lasso for high degrees
- **Never extrapolate**: Predictions outside training range are unreliable
