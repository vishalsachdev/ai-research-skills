# Seaborn Plot Gallery

Comprehensive examples of all Seaborn plot types.

## Distribution Plots

### Histogram with KDE
```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 3, figsize=(15, 5))

# Basic histogram
sns.histplot(data=tips, x='total_bill', ax=axes[0])
axes[0].set_title('Basic Histogram')

# Histogram with KDE overlay
sns.histplot(data=tips, x='total_bill', kde=True, ax=axes[1])
axes[1].set_title('Histogram + KDE')

# Multiple distributions
sns.histplot(data=tips, x='total_bill', hue='time', 
             element='step', ax=axes[2])
axes[2].set_title('Multiple Histograms')

plt.tight_layout()
plt.show()
```

### KDE Plots
```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 3, figsize=(15, 5))

# Univariate KDE
sns.kdeplot(data=tips, x='total_bill', fill=True, ax=axes[0])
axes[0].set_title('Univariate KDE')

# Bivariate KDE
sns.kdeplot(data=tips, x='total_bill', y='tip', 
            fill=True, cmap='Blues', ax=axes[1])
axes[1].set_title('Bivariate KDE')

# Multiple KDEs by category
sns.kdeplot(data=tips, x='total_bill', hue='day', 
            fill=True, alpha=0.5, ax=axes[2])
axes[2].set_title('Multiple KDEs')

plt.tight_layout()
plt.show()
```

### ECDF (Empirical Cumulative Distribution)
```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 2, figsize=(12, 5))

# Basic ECDF
sns.ecdfplot(data=tips, x='total_bill', ax=axes[0])
axes[0].set_title('ECDF Plot')

# ECDF by group
sns.ecdfplot(data=tips, x='total_bill', hue='time', ax=axes[1])
axes[1].set_title('ECDF by Time')

plt.tight_layout()
plt.show()
```

### Rug Plots
```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, ax = plt.subplots(figsize=(12, 6))

# KDE with rug plot
sns.kdeplot(data=tips, x='total_bill', fill=True, ax=ax)
sns.rugplot(data=tips, x='total_bill', height=0.05, ax=ax)

ax.set_title('KDE with Rug Plot')
plt.show()
```

## Categorical Plots

### Strip Plots
```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 3, figsize=(15, 5))

# Basic strip plot
sns.stripplot(data=tips, x='day', y='total_bill', ax=axes[0])
axes[0].set_title('Strip Plot')

# With hue
sns.stripplot(data=tips, x='day', y='total_bill', 
              hue='time', ax=axes[1])
axes[1].set_title('Strip Plot with Hue')

# With dodge
sns.stripplot(data=tips, x='day', y='total_bill', 
              hue='time', dodge=True, ax=axes[2])
axes[2].set_title('Dodged Strip Plot')

plt.tight_layout()
plt.show()
```

### Swarm Plots
```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 2, figsize=(12, 5))

# Basic swarm plot
sns.swarmplot(data=tips, x='day', y='total_bill', ax=axes[0])
axes[0].set_title('Swarm Plot')

# With hue and dodge
sns.swarmplot(data=tips, x='day', y='total_bill', 
              hue='time', dodge=True, ax=axes[1])
axes[1].set_title('Swarm Plot with Hue')

plt.tight_layout()
plt.show()
```

### Box Plots
```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# Basic box plot
sns.boxplot(data=tips, x='day', y='total_bill', ax=axes[0, 0])
axes[0, 0].set_title('Box Plot')

# With hue
sns.boxplot(data=tips, x='day', y='total_bill', 
            hue='time', ax=axes[0, 1])
axes[0, 1].set_title('Box Plot with Hue')

# With swarm overlay
sns.boxplot(data=tips, x='day', y='total_bill', ax=axes[1, 0])
sns.swarmplot(data=tips, x='day', y='total_bill', 
              color='black', alpha=0.5, size=3, ax=axes[1, 0])
axes[1, 0].set_title('Box + Swarm')

# Notched box plot
sns.boxplot(data=tips, x='day', y='total_bill', 
            notch=True, ax=axes[1, 1])
axes[1, 1].set_title('Notched Box Plot')

plt.tight_layout()
plt.show()
```

### Violin Plots
```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# Basic violin plot
sns.violinplot(data=tips, x='day', y='total_bill', ax=axes[0, 0])
axes[0, 0].set_title('Violin Plot')

# With hue and split
sns.violinplot(data=tips, x='day', y='total_bill', 
               hue='time', split=True, ax=axes[0, 1])
axes[0, 1].set_title('Split Violin Plot')

# Inner: box
sns.violinplot(data=tips, x='day', y='total_bill', 
               inner='box', ax=axes[1, 0])
axes[1, 0].set_title('Violin with Inner Box')

# Inner: quartile
sns.violinplot(data=tips, x='day', y='total_bill', 
               inner='quartile', ax=axes[1, 1])
axes[1, 1].set_title('Violin with Quartiles')

plt.tight_layout()
plt.show()
```

### Bar Plots
```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 3, figsize=(15, 5))

# Mean with confidence interval
sns.barplot(data=tips, x='day', y='total_bill', ax=axes[0])
axes[0].set_title('Bar Plot (Mean + CI)')

# With hue
sns.barplot(data=tips, x='day', y='total_bill', 
            hue='time', ax=axes[1])
axes[1].set_title('Bar Plot with Hue')

# Different estimator
sns.barplot(data=tips, x='day', y='total_bill', 
            estimator='sum', ax=axes[2])
axes[2].set_title('Bar Plot (Sum)')

plt.tight_layout()
plt.show()
```

### Count Plots
```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 2, figsize=(12, 5))

# Basic count plot
sns.countplot(data=tips, x='day', ax=axes[0])
axes[0].set_title('Count Plot')

# With hue
sns.countplot(data=tips, x='day', hue='time', ax=axes[1])
axes[1].set_title('Count Plot with Hue')

plt.tight_layout()
plt.show()
```

### Point Plots
```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 2, figsize=(12, 5))

# Basic point plot
sns.pointplot(data=tips, x='day', y='total_bill', ax=axes[0])
axes[0].set_title('Point Plot')

# With hue
sns.pointplot(data=tips, x='day', y='total_bill', 
              hue='time', ax=axes[1])
axes[1].set_title('Point Plot with Hue')

plt.tight_layout()
plt.show()
```

## Regression Plots

### Linear Regression
```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, axes = plt.subplots(1, 3, figsize=(15, 5))

# Basic regression plot
sns.regplot(data=tips, x='total_bill', y='tip', ax=axes[0])
axes[0].set_title('Linear Regression')

# Regression with robust fitting
sns.regplot(data=tips, x='total_bill', y='tip', 
            robust=True, ax=axes[1])
axes[1].set_title('Robust Regression')

# Polynomial regression (order 2)
sns.regplot(data=tips, x='total_bill', y='tip', 
            order=2, ax=axes[2])
axes[2].set_title('Polynomial Regression')

plt.tight_layout()
plt.show()
```

### lmplot (FacetGrid regression)
```python
import seaborn as sns

tips = sns.load_dataset('tips')

# Regression by categorical variable
g = sns.lmplot(data=tips, x='total_bill', y='tip', 
               hue='time', height=5, aspect=1.5)
g.set_axis_labels('Total Bill ($)', 'Tip ($)')
plt.show()

# Faceted by column
g = sns.lmplot(data=tips, x='total_bill', y='tip', 
               col='day', col_wrap=2, height=4)
plt.show()

# With row and column
g = sns.lmplot(data=tips, x='total_bill', y='tip', 
               row='time', col='day', height=3)
plt.show()
```

### Residual Plots
```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, ax = plt.subplots(figsize=(10, 6))

sns.residplot(data=tips, x='total_bill', y='tip', ax=ax)
ax.axhline(0, color='gray', linestyle='--')
ax.set_title('Residual Plot')

plt.show()
```

## Matrix Plots

### Heatmaps
```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

# Create correlation matrix
flights = sns.load_dataset('flights')
flights_pivot = flights.pivot_table(index='month', columns='year', values='passengers')

fig, axes = plt.subplots(1, 2, figsize=(14, 6))

# Basic heatmap
sns.heatmap(flights_pivot, ax=axes[0])
axes[0].set_title('Basic Heatmap')

# Annotated heatmap with custom colormap
sns.heatmap(flights_pivot, annot=True, fmt='d', 
            cmap='YlGnBu', linewidths=0.5, ax=axes[1])
axes[1].set_title('Annotated Heatmap')

plt.tight_layout()
plt.show()
```

### Correlation Heatmap
```python
import seaborn as sns
import matplotlib.pyplot as plt

iris = sns.load_dataset('iris')
corr = iris.select_dtypes(include='number').corr()

fig, axes = plt.subplots(1, 2, figsize=(14, 6))

# Triangle mask for correlation
mask = np.triu(np.ones_like(corr, dtype=bool))

# Correlation heatmap
sns.heatmap(corr, annot=True, cmap='coolwarm', 
            center=0, square=True, ax=axes[0])
axes[0].set_title('Correlation Heatmap')

# With mask
sns.heatmap(corr, mask=mask, annot=True, cmap='coolwarm', 
            center=0, square=True, linewidths=1, ax=axes[1])
axes[1].set_title('Masked Correlation Heatmap')

plt.tight_layout()
plt.show()
```

### Clustermap
```python
import seaborn as sns

iris = sns.load_dataset('iris')
iris_numeric = iris.select_dtypes(include='number')

# Hierarchical clustering heatmap
g = sns.clustermap(iris_numeric, cmap='viridis', 
                   standard_scale=1, figsize=(10, 10))
plt.show()
```

## Multi-plot Grids

### PairPlot
```python
import seaborn as sns

iris = sns.load_dataset('iris')

# Basic pairplot
g = sns.pairplot(iris, hue='species')
plt.show()

# With different diagonal
g = sns.pairplot(iris, hue='species', diag_kind='kde')
plt.show()

# With regression
g = sns.pairplot(iris, hue='species', kind='reg')
plt.show()
```

### FacetGrid
```python
import seaborn as sns

tips = sns.load_dataset('tips')

# Create grid
g = sns.FacetGrid(tips, col='day', row='time', height=4)
g.map(sns.scatterplot, 'total_bill', 'tip')
g.add_legend()
plt.show()

# With histogram
g = sns.FacetGrid(tips, col='day', col_wrap=2, height=4)
g.map(sns.histplot, 'total_bill', kde=True)
plt.show()
```

### JointGrid
```python
import seaborn as sns

tips = sns.load_dataset('tips')

# Scatter with marginal histograms
g = sns.jointplot(data=tips, x='total_bill', y='tip', 
                  kind='scatter', height=8)
plt.show()

# Hexbin with marginal histograms
g = sns.jointplot(data=tips, x='total_bill', y='tip', 
                  kind='hex', height=8)
plt.show()

# KDE
g = sns.jointplot(data=tips, x='total_bill', y='tip', 
                  kind='kde', height=8)
plt.show()

# Regression
g = sns.jointplot(data=tips, x='total_bill', y='tip', 
                  kind='reg', height=8)
plt.show()
```

### PairGrid (custom pairplot)
```python
import seaborn as sns

iris = sns.load_dataset('iris')

# Custom pairplot
g = sns.PairGrid(iris, hue='species')
g.map_upper(sns.scatterplot)
g.map_lower(sns.kdeplot)
g.map_diag(sns.histplot)
g.add_legend()

plt.show()
```

## Time Series Plots

### Line plots
```python
import seaborn as sns
import matplotlib.pyplot as plt

flights = sns.load_dataset('flights')
flights_subset = flights[flights['year'] == 1960]

fig, ax = plt.subplots(figsize=(12, 6))

sns.lineplot(data=flights, x='year', y='passengers', 
             hue='month', ax=ax)
ax.set_title('Time Series Plot')

plt.show()
```

### Relational plots with time
```python
import seaborn as sns

flights = sns.load_dataset('flights')

# Line plot with error bands
g = sns.relplot(data=flights, x='year', y='passengers', 
                kind='line', height=6, aspect=2)
plt.show()

# Faceted time series
g = sns.relplot(data=flights, x='year', y='passengers', 
                col='month', col_wrap=4, kind='line', height=3)
plt.show()
```
