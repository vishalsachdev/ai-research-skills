---
name: seaborn-statistical-viz
description: Provides high-level statistical data visualization built on Matplotlib. Use when creating statistical graphics, distribution plots, regression visualizations, or categorical comparisons with pandas DataFrames. Optimized for statistical analysis with beautiful defaults and minimal code.
version: 1.0.0
author: Orchestra Research
license: MIT
tags: [Data Visualization, Seaborn, Statistical Plots, Distribution Analysis, Regression, Python, pandas]
dependencies: [seaborn>=0.13.0, matplotlib>=3.8.0, pandas>=2.0.0, numpy>=1.24.0]
---

# Seaborn - Statistical Data Visualization

## Quick start

Seaborn is a high-level visualization library built on Matplotlib, specialized for statistical graphics with pandas integration and beautiful defaults.

**Installation**:
```bash
pip install seaborn pandas matplotlib numpy
```

**Basic usage**:
```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd

# Load sample dataset
tips = sns.load_dataset('tips')

# Create statistical plot with one line
sns.scatterplot(data=tips, x='total_bill', y='tip', hue='time')
plt.show()

# Distribution plot
sns.histplot(data=tips, x='total_bill', kde=True)
plt.show()

# Categorical plot
sns.boxplot(data=tips, x='day', y='total_bill')
plt.show()
```

## Common workflows

### Workflow 1: Exploratory data analysis (EDA)

Copy this checklist:

```
EDA Workflow:
☐ Load data into pandas DataFrame
☐ Set seaborn style and color palette
☐ Create distribution plots for numeric variables
☐ Create categorical plots for groups
☐ Generate correlation heatmap
☐ Create pairplot for relationships
☐ Add regression lines where appropriate
☐ Export visualizations
```

**Implementation**:
```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd

# Set style
sns.set_theme(style='whitegrid', palette='muted')

# Load data
df = pd.read_csv('data.csv')

# 1. Distribution analysis
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

# Histogram with KDE
sns.histplot(data=df, x='value', kde=True, ax=axes[0, 0])
axes[0, 0].set_title('Distribution with KDE', fontweight='bold')

# Box plot by category
sns.boxplot(data=df, x='category', y='value', ax=axes[0, 1])
axes[0, 1].set_title('Box Plot by Category', fontweight='bold')

# Violin plot
sns.violinplot(data=df, x='category', y='value', ax=axes[1, 0])
axes[1, 0].set_title('Violin Plot', fontweight='bold')

# Count plot for categorical
sns.countplot(data=df, x='category', ax=axes[1, 1])
axes[1, 1].set_title('Category Counts', fontweight='bold')

plt.tight_layout()
plt.savefig('distributions.png', dpi=300, bbox_inches='tight')
plt.show()

# 2. Correlation heatmap
plt.figure(figsize=(10, 8))
correlation = df.select_dtypes(include='number').corr()
sns.heatmap(correlation, annot=True, cmap='coolwarm', 
            center=0, square=True, linewidths=1,
            cbar_kws={'label': 'Correlation'})
plt.title('Correlation Heatmap', fontweight='bold', fontsize=16)
plt.tight_layout()
plt.savefig('correlation.png', dpi=300, bbox_inches='tight')
plt.show()

# 3. Pairplot for relationships
sns.pairplot(df, hue='category', diag_kind='kde', 
             plot_kws={'alpha': 0.6, 's': 80, 'edgecolor': 'k'},
             height=2.5)
plt.savefig('pairplot.png', dpi=300, bbox_inches='tight')
plt.show()
```

### Workflow 2: Statistical comparison and regression

Copy this checklist:

```
Statistical Comparison Workflow:
☐ Define comparison groups
☐ Create distribution plots by group
☐ Add regression lines with confidence intervals
☐ Generate strip plots or swarm plots
☐ Create faceted plots for multiple comparisons
☐ Add statistical annotations
☐ Export publication-ready figures
```

**Implementation**:
```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd

# Set publication style
sns.set_theme(style='ticks', font_scale=1.2)

# Load data
tips = sns.load_dataset('tips')

# 1. Regression plot with confidence interval
fig, axes = plt.subplots(1, 2, figsize=(14, 6))

sns.regplot(data=tips, x='total_bill', y='tip', 
            scatter_kws={'alpha': 0.5, 's': 50},
            line_kws={'color': 'red', 'linewidth': 2},
            ax=axes[0])
axes[0].set_title('Linear Regression with 95% CI', fontweight='bold')
axes[0].set_xlabel('Total Bill ($)', fontweight='bold')
axes[0].set_ylabel('Tip ($)', fontweight='bold')

# 2. Residual plot
sns.residplot(data=tips, x='total_bill', y='tip',
              scatter_kws={'alpha': 0.5},
              line_kws={'color': 'red', 'linewidth': 2},
              ax=axes[1])
axes[1].set_title('Residual Plot', fontweight='bold')
axes[1].axhline(0, color='gray', linestyle='--', linewidth=1)

plt.tight_layout()
plt.savefig('regression_analysis.png', dpi=300, bbox_inches='tight')
plt.show()

# 3. Categorical comparison with statistical layers
fig, axes = plt.subplots(1, 3, figsize=(16, 5))

# Box plot with swarm overlay
sns.boxplot(data=tips, x='day', y='total_bill', ax=axes[0])
sns.swarmplot(data=tips, x='day', y='total_bill', 
              color='black', alpha=0.5, size=3, ax=axes[0])
axes[0].set_title('Box Plot + Swarm', fontweight='bold')

# Violin plot with inner quartiles
sns.violinplot(data=tips, x='day', y='total_bill', 
               inner='quartile', ax=axes[1])
axes[1].set_title('Violin Plot', fontweight='bold')

# Point plot with error bars
sns.pointplot(data=tips, x='day', y='total_bill', 
              errorbar='ci', capsize=0.1, ax=axes[2])
axes[2].set_title('Point Plot with CI', fontweight='bold')

plt.tight_layout()
plt.savefig('categorical_comparison.png', dpi=300, bbox_inches='tight')
plt.show()

# 4. Faceted regression plots
g = sns.lmplot(data=tips, x='total_bill', y='tip', 
               hue='time', col='day', height=4, aspect=0.8,
               scatter_kws={'alpha': 0.6, 's': 50})
g.set_axis_labels('Total Bill ($)', 'Tip ($)', fontweight='bold')
g.fig.suptitle('Regression by Day and Time', 
               fontweight='bold', fontsize=16, y=1.02)
plt.savefig('faceted_regression.png', dpi=300, bbox_inches='tight')
plt.show()
```

### Workflow 3: Multi-panel publication figure

Copy this checklist:

```
Publication Figure Workflow:
☐ Plan layout and panel arrangement
☐ Create figure with proper dimensions
☐ Add distribution plots (histograms, KDE)
☐ Add relationship plots (scatter, regression)
☐ Add categorical comparisons (box, violin)
☐ Apply consistent styling across panels
☐ Add panel labels (A, B, C, etc.)
☐ Save in publication formats (PDF, PNG, SVG)
```

**Implementation**:
```python
import seaborn as sns
import matplotlib.pyplot as plt
import pandas as pd
import string

# Set publication style
sns.set_theme(style='white', font_scale=1.1)

# Load data
iris = sns.load_dataset('iris')

# Create multi-panel figure
fig = plt.figure(figsize=(14, 10), constrained_layout=True)
gs = fig.add_gridspec(2, 2)

# Panel A: Pairwise scatter with regression
ax1 = fig.add_subplot(gs[0, 0])
sns.scatterplot(data=iris, x='sepal_length', y='sepal_width', 
                hue='species', style='species', s=80, ax=ax1)
sns.regplot(data=iris, x='sepal_length', y='sepal_width', 
            scatter=False, color='gray', ax=ax1)
ax1.set_title('Sepal Dimensions', fontweight='bold')
ax1.legend(title='Species', loc='best')

# Panel B: Distribution comparison
ax2 = fig.add_subplot(gs[0, 1])
sns.violinplot(data=iris, x='species', y='petal_length', 
               inner='box', ax=ax2)
ax2.set_title('Petal Length by Species', fontweight='bold')

# Panel C: Joint distribution
ax3 = fig.add_subplot(gs[1, :])
for species in iris['species'].unique():
    subset = iris[iris['species'] == species]
    sns.kdeplot(data=subset, x='petal_length', y='petal_width',
                label=species, fill=True, alpha=0.3, ax=ax3)
ax3.set_title('Joint Distribution of Petal Dimensions', fontweight='bold')
ax3.legend(title='Species')

# Add panel labels
for i, ax in enumerate([ax1, ax2, ax3]):
    ax.text(-0.1, 1.05, string.ascii_uppercase[i], 
            transform=ax.transAxes, fontsize=16, 
            fontweight='bold', va='top')

# Remove top and right spines
sns.despine()

plt.savefig('publication_figure.pdf', dpi=300, bbox_inches='tight')
plt.savefig('publication_figure.png', dpi=300, bbox_inches='tight')
plt.show()
```

## When to use vs alternatives

**Use Seaborn when**:
- Creating statistical visualizations quickly
- Working with pandas DataFrames
- Need beautiful defaults without extensive customization
- Creating distribution plots, regression plots, categorical comparisons
- Performing exploratory data analysis
- Publishing statistical results

**Use Matplotlib instead when**:
- Need fine-grained control over every element
- Creating non-statistical plots (technical diagrams, schematics)
- Building custom visualization types
- Need animation capabilities
- Working without pandas DataFrames

**Use Plotly instead when**:
- Need interactive web-based visualizations
- Building dashboards
- Require zoom, pan, hover tooltips
- Creating 3D visualizations

**Use Altair instead when**:
- Prefer declarative plotting grammar
- Creating complex interactive visualizations
- Need Vega/Vega-Lite specifications

## Core plot types

**Distribution plots**: histplot, kdeplot, ecdfplot, rugplot - See [references/plot-gallery.md](references/plot-gallery.md)

**Categorical plots**: stripplot, swarmplot, boxplot, violinplot, barplot, pointplot - See [references/plot-gallery.md](references/plot-gallery.md)

**Regression plots**: regplot, lmplot, residplot - See [references/plot-gallery.md](references/plot-gallery.md)

## Styling and themes

**Built-in themes**: See [references/colors.md](references/colors.md) for styles (whitegrid, darkgrid, white, dark, ticks), color palettes (categorical, sequential, diverging), and context scaling (paper, notebook, talk, poster)

## Common issues

### Issue 1: Overlapping labels in categorical plots

**Symptoms**: X-axis labels overlap when categories have long names

**Solution**:
```python
# Rotate labels
plt.xticks(rotation=45, ha='right')

# Or use seaborn's rotation
sns.boxplot(data=df, x='category', y='value')
plt.xticks(rotation=45)

# Or make horizontal
sns.boxplot(data=df, x='value', y='category')  # Swap x and y
```

### Issue 2: Legend blocking data

**Symptoms**: Legend covers important parts of visualization

**Solution**:
```python
# Move legend outside
sns.scatterplot(data=df, x='x', y='y', hue='category')
plt.legend(bbox_to_anchor=(1.05, 1), loc='upper left')

# Or remove legend
sns.scatterplot(data=df, x='x', y='y', hue='category', legend=False)

# Reposition within plot
plt.legend(loc='lower right')
```

### Issue 3: Figure too small for multi-panel plots

**Symptoms**: Cramped subplots or cut-off labels

**Solution**:
```python
# Increase figure size
fig, axes = plt.subplots(2, 2, figsize=(14, 12))

# Use constrained_layout
fig, axes = plt.subplots(2, 2, constrained_layout=True)

# Or tight_layout
plt.tight_layout()
```

### Issue 4: Incorrect data aggregation in barplot

**Symptoms**: Bar heights don't match expected values

**Solution**:
```python
# By default, barplot shows mean with confidence interval
# Specify estimator explicitly
sns.barplot(data=df, x='category', y='value', estimator='mean')

# Use sum instead of mean
sns.barplot(data=df, x='category', y='value', estimator='sum')

# Or use countplot for counts
sns.countplot(data=df, x='category')
```

### Issue 5: Heatmap annotations too small

**Symptoms**: Can't read numbers in annotated heatmap

**Solution**:
```python
# Increase annotation font size
sns.heatmap(data, annot=True, fmt='.2f', 
            annot_kws={'size': 12})

# Larger figure
plt.figure(figsize=(12, 10))
sns.heatmap(data, annot=True)

# Format numbers
sns.heatmap(data, annot=True, fmt='.1f')  # 1 decimal place
```

### Issue 6: Color palette not suitable for data

**Symptoms**: Colors don't represent data appropriately

**Solution**:
```python
# Use appropriate palette type

# Categorical data (distinct groups)
sns.set_palette('Set2')
sns.set_palette('tab10')

# Sequential data (low to high)
sns.set_palette('Blues')
sns.set_palette('viridis')

# Diverging data (negative to positive)
sns.set_palette('coolwarm')
sns.set_palette('RdBu_r')

# Custom palette
sns.color_palette(['#FF0000', '#00FF00', '#0000FF'])
```

### Issue 7: FacetGrid plots too small

**Symptoms**: Individual facets are too small to read

**Solution**:
```python
# Increase height and aspect
g = sns.FacetGrid(df, col='category', height=5, aspect=1.2)
g.map(sns.histplot, 'value')

# Or use larger col_wrap
g = sns.FacetGrid(df, col='category', col_wrap=3, height=4)
```

## Advanced features

**Plot gallery**: See [references/plot-gallery.md](references/plot-gallery.md) for comprehensive examples

**Color palettes**: See [references/colors.md](references/colors.md) for palette creation and color theory

**Statistical annotations**: See [references/statistics.md](references/statistics.md) for adding p-values and statistical tests

## Resources

- **Official Documentation**: https://seaborn.pydata.org/
- **Gallery**: https://seaborn.pydata.org/examples/index.html (100+ examples)
- **Tutorial**: https://seaborn.pydata.org/tutorial.html
- **API Reference**: https://seaborn.pydata.org/api.html
