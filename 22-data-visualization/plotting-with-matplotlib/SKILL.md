---
name: plotting-with-matplotlib
description: Provides comprehensive static, animated, and interactive plotting using Matplotlib, the foundational plotting library for Python. Use when creating publication-quality figures, scientific plots, statistical charts, or custom visualizations. De facto standard for Python plotting with 40+ years combined development and 15M+ downloads per month.
version: 1.0.0
author: Orchestra Research
license: MIT
tags: [Data Visualization, Matplotlib, Plotting, Charts, Scientific Visualization, Publication Quality, Python]
dependencies: [matplotlib>=3.8.0, numpy>=1.24.0]
---

# Matplotlib - Python Plotting Library

## Quick start

Matplotlib is the foundational plotting library for Python, providing object-oriented APIs for embedding plots into applications and publication-quality figures.

**Installation**:
```bash
pip install matplotlib numpy
```

**Basic usage**:
```python
import matplotlib.pyplot as plt
import numpy as np

# Simple line plot
x = np.linspace(0, 10, 100)
y = np.sin(x)

plt.figure(figsize=(10, 6))
plt.plot(x, y, label='sin(x)')
plt.xlabel('X axis')
plt.ylabel('Y axis')
plt.title('Simple Line Plot')
plt.legend()
plt.grid(True)
plt.show()

# Save figure
plt.savefig('plot.png', dpi=300, bbox_inches='tight')
```

## Common workflows

### Workflow 1: Create publication-quality figure

Copy this checklist:

```
Publication Figure Workflow:
☐ Set up figure with proper size and DPI
☐ Choose appropriate plot type for data
☐ Configure axes, labels, and title
☐ Apply publication style (seaborn-whitegrid, ggplot, etc.)
☐ Add legend and annotations
☐ Adjust spacing and margins
☐ Save in multiple formats (PNG, PDF, SVG)
☐ Validate output meets journal requirements
```

**Implementation**:
```python
import matplotlib.pyplot as plt
import numpy as np

# Set publication style
plt.style.use('seaborn-v0_8-whitegrid')
plt.rcParams.update({
    'font.size': 12,
    'font.family': 'sans-serif',
    'axes.labelsize': 14,
    'axes.titlesize': 16,
    'xtick.labelsize': 12,
    'ytick.labelsize': 12,
    'legend.fontsize': 12,
    'figure.titlesize': 18
})

# Create figure with specific dimensions (in inches)
fig, ax = plt.subplots(figsize=(8, 6), dpi=100)

# Plot data
x = np.linspace(0, 2*np.pi, 100)
ax.plot(x, np.sin(x), 'b-', linewidth=2, label='sin(x)')
ax.plot(x, np.cos(x), 'r--', linewidth=2, label='cos(x)')

# Configure axes
ax.set_xlabel('Angle (radians)', fontweight='bold')
ax.set_ylabel('Amplitude', fontweight='bold')
ax.set_title('Trigonometric Functions', fontweight='bold', pad=20)
ax.legend(loc='upper right', frameon=True, shadow=True)
ax.grid(True, alpha=0.3)

# Set axis limits
ax.set_xlim([0, 2*np.pi])
ax.set_ylim([-1.2, 1.2])

# Add annotation
ax.annotate('Peak', xy=(np.pi/2, 1), xytext=(np.pi/2, 1.3),
            arrowprops=dict(arrowstyle='->', color='black', lw=1.5),
            fontsize=12, ha='center')

# Adjust layout to prevent label cutoff
plt.tight_layout()

# Save in multiple formats for publication
plt.savefig('figure.png', dpi=300, bbox_inches='tight')
plt.savefig('figure.pdf', format='pdf', bbox_inches='tight')
plt.savefig('figure.svg', format='svg', bbox_inches='tight')

plt.show()
```

### Workflow 2: Create multi-panel figure (subplots)

Copy this checklist:

```
Multi-Panel Figure Workflow:
☐ Plan layout (rows, columns, size ratios)
☐ Create subplot grid with constrained_layout
☐ Plot data in each panel
☐ Configure individual axes
☐ Add panel labels (A, B, C, etc.)
☐ Synchronize axes if needed (sharex, sharey)
☐ Add overall title
☐ Save composite figure
```

**Implementation**:
```python
import matplotlib.pyplot as plt
import numpy as np

# Create 2x2 subplot grid
fig, axes = plt.subplots(2, 2, figsize=(12, 10), 
                         constrained_layout=True)

# Flatten axes array for easier iteration
axes = axes.flatten()

# Generate sample data
x = np.linspace(0, 10, 100)
datasets = [
    (np.sin(x), 'Line Plot', 'plot'),
    (np.random.randn(1000), 'Histogram', 'hist'),
    (np.random.randn(50), 'Scatter', 'scatter'),
    (np.random.randn(10, 10), 'Heatmap', 'imshow')
]

# Plot data in each panel
for i, (data, title, plot_type) in enumerate(datasets):
    ax = axes[i]
    
    if plot_type == 'plot':
        ax.plot(x, data, 'b-', linewidth=2)
        ax.set_xlabel('X axis')
        ax.set_ylabel('Y axis')
        ax.grid(True, alpha=0.3)
    
    elif plot_type == 'hist':
        ax.hist(data, bins=30, color='skyblue', edgecolor='black', alpha=0.7)
        ax.set_xlabel('Value')
        ax.set_ylabel('Frequency')
    
    elif plot_type == 'scatter':
        ax.scatter(range(len(data)), data, c=data, cmap='viridis', s=100)
        ax.set_xlabel('Index')
        ax.set_ylabel('Value')
    
    elif plot_type == 'imshow':
        im = ax.imshow(data, cmap='coolwarm', aspect='auto')
        plt.colorbar(im, ax=ax, label='Value')
        ax.set_xlabel('Column')
        ax.set_ylabel('Row')
    
    # Add panel label
    ax.text(-0.1, 1.1, chr(65 + i), transform=ax.transAxes,
            fontsize=20, fontweight='bold', va='top')
    
    ax.set_title(title, fontweight='bold', fontsize=14)

# Add overall title
fig.suptitle('Multi-Panel Scientific Figure', fontsize=18, fontweight='bold')

plt.savefig('multipanel.png', dpi=300, bbox_inches='tight')
plt.show()
```

### Workflow 3: Customize plot with advanced styling

Copy this checklist:

```
Advanced Styling Workflow:
☐ Choose or create custom style sheet
☐ Set color palette and cycle
☐ Configure spine and tick appearance
☐ Add custom markers and line styles
☐ Apply background color/image
☐ Add error bars or confidence intervals
☐ Format tick labels (dates, currency, etc.)
☐ Export with transparent background
```

**Implementation**:
```python
import matplotlib.pyplot as plt
import numpy as np
from matplotlib.dates import DateFormatter
import matplotlib.dates as mdates

# Create custom style
custom_style = {
    'axes.facecolor': '#f5f5f5',
    'axes.edgecolor': '#333333',
    'axes.linewidth': 1.5,
    'grid.color': 'white',
    'grid.linewidth': 1.2,
    'xtick.color': '#333333',
    'ytick.color': '#333333',
    'text.color': '#333333',
    'axes.prop_cycle': plt.cycler('color', 
                                  ['#1f77b4', '#ff7f0e', '#2ca02c', '#d62728'])
}

plt.rcParams.update(custom_style)

# Create figure
fig, ax = plt.subplots(figsize=(12, 6))

# Generate data with error bars
x = np.arange(0, 10, 0.5)
y1 = np.exp(-x/5) * np.cos(2*np.pi*x)
y2 = np.exp(-x/5) * np.sin(2*np.pi*x)
error = 0.1 * np.abs(y1)

# Plot with error bars
ax.errorbar(x, y1, yerr=error, fmt='o-', linewidth=2, 
            markersize=8, capsize=5, capthick=2,
            label='Damped Cosine', alpha=0.8)

ax.fill_between(x, y2 - 0.1, y2 + 0.1, alpha=0.3, label='Confidence Band')
ax.plot(x, y2, 's--', linewidth=2, markersize=6, label='Damped Sine')

# Customize spines
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)
ax.spines['left'].set_linewidth(2)
ax.spines['bottom'].set_linewidth(2)

# Configure labels
ax.set_xlabel('Time (s)', fontsize=14, fontweight='bold')
ax.set_ylabel('Amplitude', fontsize=14, fontweight='bold')
ax.set_title('Custom Styled Plot with Error Bars', 
             fontsize=16, fontweight='bold', pad=20)

# Legend with custom positioning
ax.legend(loc='upper right', frameon=True, 
          fancybox=True, shadow=True, ncol=1)

# Add grid
ax.grid(True, alpha=0.6, linestyle='-', linewidth=1)

# Format tick labels
ax.tick_params(axis='both', which='major', labelsize=12, 
               width=1.5, length=6)
ax.tick_params(axis='both', which='minor', width=1, length=3)

plt.tight_layout()
plt.savefig('custom_style.png', dpi=300, bbox_inches='tight', 
            facecolor='white', edgecolor='none')
plt.show()
```

## When to use vs alternatives

**Use Matplotlib when**:
- Creating publication-quality static figures
- Need fine-grained control over every plot element
- Building custom visualization types
- Creating plots for scientific papers or reports
- Need to export to multiple formats (PNG, PDF, SVG, EPS)
- Working with MATLAB-like plotting interface

**Use Seaborn instead when**:
- Creating statistical visualizations quickly
- Need beautiful default styles out-of-the-box
- Working with pandas DataFrames extensively
- Creating complex statistical plots (violin, box, pair plots)

**Use Plotly instead when**:
- Need interactive web-based visualizations
- Building dashboards or web applications
- Require zoom, pan, hover tooltips
- Creating 3D visualizations

**Use Altair instead when**:
- Need declarative plotting grammar
- Creating interactive plots with minimal code
- Working with Vega/Vega-Lite specifications

## Core plotting types

**Basic plots**: Line plots, scatter plots, bar charts, histograms - See [references/plot-types.md](references/plot-types.md) for comprehensive examples and code

## Common issues

### Issue 1: Figures not showing in Jupyter

**Symptoms**: `plt.show()` doesn't display figures in Jupyter notebooks

**Solution**:
```python
# Add magic command at start of notebook
%matplotlib inline

# Or for interactive plots
%matplotlib widget

# For newer JupyterLab
%matplotlib ipympl
```

### Issue 2: Text/labels cut off when saving

**Symptoms**: Axis labels or titles are clipped in saved images

**Solution**:
```python
# Use bbox_inches='tight' when saving
plt.savefig('plot.png', bbox_inches='tight')

# Or use constrained_layout when creating figure
fig, ax = plt.subplots(constrained_layout=True)

# Or call tight_layout before showing/saving
plt.tight_layout()
plt.savefig('plot.png')
```

### Issue 3: Poor quality images for publication

**Symptoms**: Saved figures look pixelated or blurry

**Solution**:
```python
# Increase DPI (dots per inch) - 300+ for publications
plt.savefig('figure.png', dpi=300)

# Use vector formats for perfect scaling
plt.savefig('figure.pdf', format='pdf')
plt.savefig('figure.svg', format='svg')

# Set figure DPI at creation
fig = plt.figure(figsize=(10, 6), dpi=100)  # Display DPI
```

### Issue 4: Memory leaks with many plots

**Symptoms**: Memory usage grows when creating many plots in loop

**Solution**:
```python
# Close figures explicitly
for i in range(100):
    fig, ax = plt.subplots()
    ax.plot([1, 2, 3], [1, 4, 9])
    plt.savefig(f'plot_{i}.png')
    plt.close(fig)  # Important!

# Or close all figures
plt.close('all')
```

### Issue 5: Inconsistent styles across plots

**Symptoms**: Each plot looks different, need uniform styling

**Solution**:
```python
# Use built-in styles
plt.style.use('seaborn-v0_8-darkgrid')

# View available styles
print(plt.style.available)

# Create custom style
custom_style = {
    'figure.figsize': (10, 6),
    'font.size': 12,
    'axes.labelsize': 14,
    'axes.titlesize': 16,
    'xtick.labelsize': 12,
    'ytick.labelsize': 12,
}
plt.rcParams.update(custom_style)

# Use context manager for temporary style
with plt.style.context('ggplot'):
    plt.plot([1, 2, 3], [1, 4, 9])
    plt.show()
```

### Issue 6: Date formatting on x-axis

**Symptoms**: Date labels overlap or are poorly formatted

**Solution**:
```python
import matplotlib.pyplot as plt
import matplotlib.dates as mdates
import pandas as pd

# Create date data
dates = pd.date_range('2024-01-01', periods=100, freq='D')
values = np.random.randn(100).cumsum()

fig, ax = plt.subplots(figsize=(12, 6))
ax.plot(dates, values)

# Format date labels
ax.xaxis.set_major_formatter(mdates.DateFormatter('%Y-%m-%d'))
ax.xaxis.set_major_locator(mdates.WeekdayLocator(interval=2))

# Rotate labels to prevent overlap
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
plt.show()
```

### Issue 7: Legend blocking data

**Symptoms**: Legend covers important parts of the plot

**Solution**:
```python
# Place legend outside plot area
ax.legend(bbox_to_anchor=(1.05, 1), loc='upper left')

# Or use best location automatically
ax.legend(loc='best')

# Make legend semi-transparent
ax.legend(loc='upper right', framealpha=0.7)

# Place legend below plot
ax.legend(loc='upper center', bbox_to_anchor=(0.5, -0.15), ncol=3)
plt.tight_layout()
```

## Advanced features

**Plot types**: See [references/plot-types.md](references/plot-types.md) for heatmaps, contour plots, polar plots, 3D visualizations

**Animations**: See [references/animations.md](references/animations.md) for creating animated plots and videos

**API reference**: See [references/api.md](references/api.md) for comprehensive pyplot and axes methods

## Resources

- **Official Documentation**: https://matplotlib.org/stable/
- **Gallery**: https://matplotlib.org/stable/gallery/index.html (400+ examples)
- **Cheat Sheets**: https://matplotlib.org/cheatsheets/
- **Tutorial**: https://matplotlib.org/stable/tutorials/index.html
