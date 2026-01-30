# Statistical Annotations in Seaborn

Guide to adding statistical annotations, p-values, and significance markers to Seaborn plots.

## Using statannotations Package

```bash
pip install statannotations
```

```python
from statannotations.Annotator import Annotator
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

# Create plot
ax = sns.boxplot(data=tips, x='day', y='total_bill')

# Define pairs to compare
pairs = [('Thur', 'Fri'), ('Fri', 'Sat'), ('Sat', 'Sun')]

# Create annotator
annotator = Annotator(ax, pairs, data=tips, x='day', y='total_bill')

# Configure and apply
annotator.configure(test='t-test_ind', text_format='star', loc='inside')
annotator.apply_and_annotate()

plt.show()
```

## Manual Statistical Annotations

### Add significance bars
```python
import seaborn as sns
import matplotlib.pyplot as plt
from scipy import stats

tips = sns.load_dataset('tips')

# Create plot
fig, ax = plt.subplots(figsize=(10, 6))
sns.boxplot(data=tips, x='day', y='total_bill', ax=ax)

# Perform t-test
group1 = tips[tips['day'] == 'Thur']['total_bill']
group2 = tips[tips['day'] == 'Fri']['total_bill']
t_stat, p_value = stats.ttest_ind(group1, group2)

# Add significance bar
x1, x2 = 0, 1  # positions of Thur and Fri
y = tips['total_bill'].max() * 1.1
h = y * 0.02

ax.plot([x1, x1, x2, x2], [y, y+h, y+h, y], lw=1.5, c='black')

# Add p-value text
if p_value < 0.001:
    sig_text = '***'
elif p_value < 0.01:
    sig_text = '**'
elif p_value < 0.05:
    sig_text = '*'
else:
    sig_text = 'ns'

ax.text((x1+x2)/2, y+h, sig_text, ha='center', va='bottom', fontsize=14)

plt.show()
```

### Add mean values
```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

tips = sns.load_dataset('tips')

fig, ax = plt.subplots(figsize=(10, 6))
bp = sns.boxplot(data=tips, x='day', y='total_bill', ax=ax)

# Calculate means
means = tips.groupby('day')['total_bill'].mean()

# Add mean markers
positions = range(len(means))
ax.scatter(positions, means, color='red', s=100, zorder=3, 
           marker='D', label='Mean')

# Add mean value text
for i, (day, mean) in enumerate(means.items()):
    ax.text(i, mean + 2, f'{mean:.2f}', ha='center', 
            fontweight='bold', color='red')

ax.legend()
plt.show()
```

## Correlation Annotations

### Correlation heatmap with p-values
```python
import seaborn as sns
import matplotlib.pyplot as plt
from scipy import stats
import numpy as np

iris = sns.load_dataset('iris')
numeric_cols = iris.select_dtypes(include='number')

# Calculate correlations and p-values
n = len(numeric_cols.columns)
corr_matrix = np.zeros((n, n))
p_matrix = np.zeros((n, n))

for i, col1 in enumerate(numeric_cols.columns):
    for j, col2 in enumerate(numeric_cols.columns):
        corr, p = stats.pearsonr(numeric_cols[col1], numeric_cols[col2])
        corr_matrix[i, j] = corr
        p_matrix[i, j] = p

# Create heatmap
fig, ax = plt.subplots(figsize=(10, 8))
sns.heatmap(corr_matrix, annot=True, cmap='coolwarm', center=0,
            xticklabels=numeric_cols.columns,
            yticklabels=numeric_cols.columns, ax=ax)

# Add significance markers
for i in range(n):
    for j in range(n):
        if p_matrix[i, j] < 0.05 and i != j:
            ax.text(j+0.5, i+0.7, '*', ha='center', va='center',
                   color='black', fontsize=16, fontweight='bold')

ax.set_title('Correlation Matrix with Significance (* p<0.05)', 
             fontweight='bold', fontsize=14)

plt.tight_layout()
plt.show()
```

## Regression Statistics

### Add R² to regression plot
```python
import seaborn as sns
import matplotlib.pyplot as plt
from scipy import stats

tips = sns.load_dataset('tips')

fig, ax = plt.subplots(figsize=(10, 6))

# Create regression plot
sns.regplot(data=tips, x='total_bill', y='tip', ax=ax)

# Calculate R²
slope, intercept, r_value, p_value, std_err = stats.linregress(
    tips['total_bill'], tips['tip'])

# Add statistics to plot
stats_text = f'R² = {r_value**2:.3f}\np < 0.001'
ax.text(0.05, 0.95, stats_text, transform=ax.transAxes,
        fontsize=14, verticalalignment='top',
        bbox=dict(boxstyle='round', facecolor='wheat', alpha=0.8))

ax.set_title('Regression with Statistics', fontweight='bold')
plt.show()
```

## Sample Size Annotations

### Add n to categorical plots
```python
import seaborn as sns
import matplotlib.pyplot as plt

tips = sns.load_dataset('tips')

fig, ax = plt.subplots(figsize=(10, 6))
sns.boxplot(data=tips, x='day', y='total_bill', ax=ax)

# Count samples per category
counts = tips['day'].value_counts()

# Add n to each box
for i, day in enumerate(['Thur', 'Fri', 'Sat', 'Sun']):
    n = counts[day]
    ax.text(i, ax.get_ylim()[0] - 2, f'n={n}', 
            ha='center', fontsize=10, fontweight='bold')

ax.set_title('Box Plot with Sample Sizes', fontweight='bold')
plt.show()
```

## Effect Size Annotations

### Cohen's d for group comparisons
```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np

tips = sns.load_dataset('tips')

def cohen_d(group1, group2):
    """Calculate Cohen's d effect size"""
    n1, n2 = len(group1), len(group2)
    var1, var2 = np.var(group1, ddof=1), np.var(group2, ddof=1)
    pooled_std = np.sqrt(((n1-1)*var1 + (n2-1)*var2) / (n1+n2-2))
    return (np.mean(group1) - np.mean(group2)) / pooled_std

# Compare groups
lunch = tips[tips['time'] == 'Lunch']['total_bill']
dinner = tips[tips['time'] == 'Dinner']['total_bill']

d = cohen_d(lunch, dinner)

# Create plot
fig, ax = plt.subplots(figsize=(10, 6))
sns.violinplot(data=tips, x='time', y='total_bill', ax=ax)

# Add Cohen's d
effect_text = f"Cohen's d = {d:.2f}\n"
if abs(d) < 0.2:
    effect_text += "(small)"
elif abs(d) < 0.5:
    effect_text += "(medium)"
else:
    effect_text += "(large)"

ax.text(0.5, 0.95, effect_text, transform=ax.transAxes,
        ha='center', va='top', fontsize=12,
        bbox=dict(boxstyle='round', facecolor='lightblue', alpha=0.8))

plt.show()
```

## Resources

- **statannotations**: https://github.com/trevismd/statannotations
- **scipy.stats**: https://docs.scipy.org/doc/scipy/reference/stats.html
- **Statistical tests guide**: https://www.statsmodels.org/stable/index.html
