# Matplotlib Plot Types Reference

Comprehensive guide to all major plot types in Matplotlib.

## Line Plots and Scatter Plots

### Multiple line plots with styling
```python
import matplotlib.pyplot as plt
import numpy as np

fig, ax = plt.subplots(figsize=(12, 6))
x = np.linspace(0, 10, 100)

# Different line styles
ax.plot(x, np.sin(x), 'b-', linewidth=2, label='sin(x) - solid')
ax.plot(x, np.cos(x), 'r--', linewidth=2, label='cos(x) - dashed')
ax.plot(x, np.sin(x) * np.cos(x), 'g-.', linewidth=2, label='sin(x)cos(x) - dashdot')
ax.plot(x, np.sin(2*x)/2, 'm:', linewidth=3, label='sin(2x)/2 - dotted')

# Configure plot
ax.set_title('Line Plot Styles', fontweight='bold', fontsize=16)
ax.set_xlabel('X axis', fontweight='bold')
ax.set_ylabel('Y axis', fontweight='bold')
ax.legend(loc='upper right')
ax.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

### Advanced scatter plot
```python
import matplotlib.pyplot as plt
import numpy as np

# Generate data
n = 500
x = np.random.randn(n)
y = np.random.randn(n)
colors = np.random.rand(n)
sizes = 1000 * np.abs(np.random.randn(n))

fig, ax = plt.subplots(figsize=(10, 8))

# Create scatter with color mapping
scatter = ax.scatter(x, y, c=colors, s=sizes, alpha=0.5, 
                     cmap='viridis', edgecolors='black', linewidth=0.5)

ax.set_title('Advanced Scatter Plot', fontweight='bold', fontsize=16)
ax.set_xlabel('X values', fontweight='bold')
ax.set_ylabel('Y values', fontweight='bold')

# Add colorbar
cbar = plt.colorbar(scatter, ax=ax)
cbar.set_label('Color Value', fontweight='bold')

# Add grid
ax.grid(True, alpha=0.3, linestyle='--')

plt.tight_layout()
plt.show()
```

## Bar Charts

### Grouped bar chart
```python
import matplotlib.pyplot as plt
import numpy as np

categories = ['Q1', 'Q2', 'Q3', 'Q4']
product_a = [23, 45, 56, 78]
product_b = [34, 55, 43, 65]
product_c = [20, 38, 50, 60]

x = np.arange(len(categories))
width = 0.25

fig, ax = plt.subplots(figsize=(12, 6))

bars1 = ax.bar(x - width, product_a, width, label='Product A', 
               color='#1f77b4', edgecolor='black')
bars2 = ax.bar(x, product_b, width, label='Product B', 
               color='#ff7f0e', edgecolor='black')
bars3 = ax.bar(x + width, product_c, width, label='Product C', 
               color='#2ca02c', edgecolor='black')

# Add value labels
for bars in [bars1, bars2, bars3]:
    for bar in bars:
        height = bar.get_height()
        ax.text(bar.get_x() + bar.get_width()/2., height,
                f'{int(height)}', ha='center', va='bottom', fontsize=10)

ax.set_xlabel('Quarter', fontweight='bold', fontsize=12)
ax.set_ylabel('Sales', fontweight='bold', fontsize=12)
ax.set_title('Quarterly Sales by Product', fontweight='bold', fontsize=16)
ax.set_xticks(x)
ax.set_xticklabels(categories)
ax.legend()
ax.grid(axis='y', alpha=0.3)

plt.tight_layout()
plt.show()
```

### Stacked bar chart
```python
import matplotlib.pyplot as plt
import numpy as np

categories = ['A', 'B', 'C', 'D', 'E']
values1 = np.array([20, 35, 30, 35, 27])
values2 = np.array([25, 32, 34, 20, 25])
values3 = np.array([15, 18, 20, 25, 20])

fig, ax = plt.subplots(figsize=(10, 6))

ax.bar(categories, values1, label='Category 1', color='#e74c3c')
ax.bar(categories, values2, bottom=values1, label='Category 2', color='#3498db')
ax.bar(categories, values3, bottom=values1+values2, label='Category 3', color='#2ecc71')

ax.set_ylabel('Values', fontweight='bold')
ax.set_title('Stacked Bar Chart', fontweight='bold', fontsize=16)
ax.legend()
ax.grid(axis='y', alpha=0.3)

plt.tight_layout()
plt.show()
```

### Horizontal bar chart
```python
import matplotlib.pyplot as plt
import numpy as np

categories = ['Machine Learning', 'Data Visualization', 'Web Development', 
              'Database Management', 'DevOps']
values = [85, 72, 90, 65, 78]

fig, ax = plt.subplots(figsize=(10, 6))

bars = ax.barh(categories, values, color='steelblue', edgecolor='black')

# Color bars by value
colors = plt.cm.RdYlGn(np.linspace(0.3, 0.9, len(values)))
for bar, color in zip(bars, colors):
    bar.set_color(color)

# Add value labels
for i, (cat, val) in enumerate(zip(categories, values)):
    ax.text(val + 1, i, f'{val}%', va='center', fontweight='bold')

ax.set_xlabel('Proficiency (%)', fontweight='bold')
ax.set_title('Skills Proficiency', fontweight='bold', fontsize=16)
ax.set_xlim([0, 100])
ax.grid(axis='x', alpha=0.3)

plt.tight_layout()
plt.show()
```

## Histograms

### Multi-dataset histogram
```python
import matplotlib.pyplot as plt
import numpy as np

# Generate sample data
data1 = np.random.normal(100, 15, 1000)
data2 = np.random.normal(110, 20, 1000)
data3 = np.random.normal(95, 12, 1000)

fig, ax = plt.subplots(figsize=(12, 6))

# Overlapping histograms
ax.hist(data1, bins=30, alpha=0.6, label='Dataset 1', color='blue', edgecolor='black')
ax.hist(data2, bins=30, alpha=0.6, label='Dataset 2', color='red', edgecolor='black')
ax.hist(data3, bins=30, alpha=0.6, label='Dataset 3', color='green', edgecolor='black')

ax.set_xlabel('Value', fontweight='bold')
ax.set_ylabel('Frequency', fontweight='bold')
ax.set_title('Overlapping Histograms', fontweight='bold', fontsize=16)
ax.legend()
ax.grid(axis='y', alpha=0.3)

plt.tight_layout()
plt.show()
```

### 2D histogram (hexbin)
```python
import matplotlib.pyplot as plt
import numpy as np

# Generate correlated data
x = np.random.randn(10000)
y = x + np.random.randn(10000) * 0.5

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 6))

# Hexbin plot
hexbin = ax1.hexbin(x, y, gridsize=30, cmap='YlOrRd', mincnt=1)
ax1.set_title('Hexbin Plot', fontweight='bold', fontsize=14)
ax1.set_xlabel('X values')
ax1.set_ylabel('Y values')
plt.colorbar(hexbin, ax=ax1, label='Count')

# 2D histogram
h = ax2.hist2d(x, y, bins=30, cmap='Blues')
ax2.set_title('2D Histogram', fontweight='bold', fontsize=14)
ax2.set_xlabel('X values')
ax2.set_ylabel('Y values')
plt.colorbar(h[3], ax=ax2, label='Count')

plt.tight_layout()
plt.show()
```

## Heatmaps and Contour Plots

### Annotated heatmap
```python
import matplotlib.pyplot as plt
import numpy as np

# Create correlation matrix
data = np.random.rand(8, 8)
labels = [f'Var{i+1}' for i in range(8)]

fig, ax = plt.subplots(figsize=(10, 8))

# Create heatmap
im = ax.imshow(data, cmap='RdYlBu_r', aspect='auto', vmin=0, vmax=1)

# Set ticks and labels
ax.set_xticks(np.arange(len(labels)))
ax.set_yticks(np.arange(len(labels)))
ax.set_xticklabels(labels)
ax.set_yticklabels(labels)

# Rotate x labels
plt.setp(ax.get_xticklabels(), rotation=45, ha='right', rotation_mode='anchor')

# Add annotations
for i in range(len(labels)):
    for j in range(len(labels)):
        text = ax.text(j, i, f'{data[i, j]:.2f}',
                      ha='center', va='center', 
                      color='white' if data[i, j] < 0.5 else 'black',
                      fontsize=10)

ax.set_title('Correlation Matrix Heatmap', fontweight='bold', fontsize=16, pad=20)
plt.colorbar(im, ax=ax, label='Correlation')

plt.tight_layout()
plt.show()
```

### Contour plots
```python
import matplotlib.pyplot as plt
import numpy as np

# Create mesh
x = np.linspace(-5, 5, 100)
y = np.linspace(-5, 5, 100)
X, Y = np.meshgrid(x, y)
Z = np.sin(np.sqrt(X**2 + Y**2))

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 6))

# Filled contour
contourf = ax1.contourf(X, Y, Z, levels=20, cmap='coolwarm')
ax1.set_title('Filled Contour Plot', fontweight='bold', fontsize=14)
ax1.set_xlabel('X axis')
ax1.set_ylabel('Y axis')
plt.colorbar(contourf, ax=ax1, label='Z value')

# Line contour with labels
contour = ax2.contour(X, Y, Z, levels=15, colors='black', linewidths=1)
ax2.clabel(contour, inline=True, fontsize=10, fmt='%1.2f')
contourf2 = ax2.contourf(X, Y, Z, levels=15, cmap='viridis', alpha=0.6)
ax2.set_title('Labeled Contour Plot', fontweight='bold', fontsize=14)
ax2.set_xlabel('X axis')
ax2.set_ylabel('Y axis')
plt.colorbar(contourf2, ax=ax2, label='Z value')

plt.tight_layout()
plt.show()
```

## Box Plots and Violin Plots

### Box plot
```python
import matplotlib.pyplot as plt
import numpy as np

# Generate sample data
np.random.seed(10)
data = [np.random.normal(100, 10, 200),
        np.random.normal(90, 20, 200),
        np.random.normal(110, 15, 200),
        np.random.normal(105, 12, 200)]

fig, ax = plt.subplots(figsize=(10, 6))

bp = ax.boxplot(data, labels=['Group A', 'Group B', 'Group C', 'Group D'],
                patch_artist=True, notch=True, showmeans=True)

# Customize colors
colors = ['lightblue', 'lightgreen', 'lightcoral', 'lightyellow']
for patch, color in zip(bp['boxes'], colors):
    patch.set_facecolor(color)

ax.set_ylabel('Values', fontweight='bold')
ax.set_title('Box Plot Comparison', fontweight='bold', fontsize=16)
ax.grid(axis='y', alpha=0.3)

plt.tight_layout()
plt.show()
```

## Pie Charts

### Advanced pie chart
```python
import matplotlib.pyplot as plt

# Data
labels = ['Product A', 'Product B', 'Product C', 'Product D', 'Product E']
sizes = [30, 25, 20, 15, 10]
colors = ['#ff9999', '#66b3ff', '#99ff99', '#ffcc99', '#ff99cc']
explode = (0.1, 0, 0, 0, 0)  # Explode 1st slice

fig, ax = plt.subplots(figsize=(10, 8))

wedges, texts, autotexts = ax.pie(sizes, explode=explode, labels=labels, 
                                    colors=colors, autopct='%1.1f%%',
                                    shadow=True, startangle=90)

# Beautify text
for text in texts:
    text.set_fontsize(12)
    text.set_fontweight('bold')

for autotext in autotexts:
    autotext.set_color('white')
    autotext.set_fontsize(11)
    autotext.set_fontweight('bold')

ax.set_title('Market Share Distribution', fontweight='bold', fontsize=16)

plt.tight_layout()
plt.show()
```

## Polar Plots

### Radar chart
```python
import matplotlib.pyplot as plt
import numpy as np

# Data
categories = ['Speed', 'Reliability', 'Comfort', 'Safety', 'Efficiency']
values = [4, 3, 5, 5, 3]

# Number of variables
N = len(categories)

# Compute angle for each axis
angles = [n / float(N) * 2 * np.pi for n in range(N)]
values += values[:1]  # Complete the circle
angles += angles[:1]

fig, ax = plt.subplots(figsize=(8, 8), subplot_kw=dict(projection='polar'))

# Plot
ax.plot(angles, values, 'o-', linewidth=2, color='b', label='Product Score')
ax.fill(angles, values, alpha=0.25, color='b')

# Fix axis to go in the right order
ax.set_xticks(angles[:-1])
ax.set_xticklabels(categories, fontsize=12)
ax.set_ylim(0, 5)

ax.set_title('Product Performance Radar Chart', 
             fontweight='bold', fontsize=16, pad=20)
ax.legend(loc='upper right', bbox_to_anchor=(1.3, 1.1))

plt.tight_layout()
plt.show()
```

## 3D Plots

### 3D surface plot
```python
import matplotlib.pyplot as plt
import numpy as np
from mpl_toolkits.mplot3d import Axes3D

# Create data
x = np.linspace(-5, 5, 50)
y = np.linspace(-5, 5, 50)
X, Y = np.meshgrid(x, y)
Z = np.sin(np.sqrt(X**2 + Y**2))

# Create 3D plot
fig = plt.figure(figsize=(12, 8))
ax = fig.add_subplot(111, projection='3d')

# Surface plot
surf = ax.plot_surface(X, Y, Z, cmap='viridis', alpha=0.8, 
                       linewidth=0, antialiased=True)

# Customize
ax.set_xlabel('X axis', fontweight='bold')
ax.set_ylabel('Y axis', fontweight='bold')
ax.set_zlabel('Z axis', fontweight='bold')
ax.set_title('3D Surface Plot', fontweight='bold', fontsize=16)

# Add colorbar
fig.colorbar(surf, ax=ax, shrink=0.5, aspect=5)

plt.show()
```

### 3D scatter plot
```python
import matplotlib.pyplot as plt
import numpy as np
from mpl_toolkits.mplot3d import Axes3D

# Generate data
n = 500
x = np.random.randn(n)
y = np.random.randn(n)
z = np.random.randn(n)
colors = np.random.rand(n)
sizes = 50 * np.abs(np.random.randn(n))

# Create 3D scatter
fig = plt.figure(figsize=(12, 8))
ax = fig.add_subplot(111, projection='3d')

scatter = ax.scatter(x, y, z, c=colors, s=sizes, cmap='plasma', 
                     alpha=0.6, edgecolors='black', linewidth=0.5)

ax.set_xlabel('X axis', fontweight='bold')
ax.set_ylabel('Y axis', fontweight='bold')
ax.set_zlabel('Z axis', fontweight='bold')
ax.set_title('3D Scatter Plot', fontweight='bold', fontsize=16)

fig.colorbar(scatter, ax=ax, shrink=0.5, aspect=5)

plt.show()
```

## Stream Plots and Quiver Plots

### Stream plot (flow visualization)
```python
import matplotlib.pyplot as plt
import numpy as np

# Create grid
x = np.linspace(-3, 3, 100)
y = np.linspace(-3, 3, 100)
X, Y = np.meshgrid(x, y)

# Define vector field
U = -Y
V = X

fig, ax = plt.subplots(figsize=(10, 8))

# Create stream plot
strm = ax.streamplot(X, Y, U, V, color=np.sqrt(U**2 + V**2), 
                     cmap='cool', linewidth=2, density=1.5, arrowsize=2)

ax.set_xlabel('X axis', fontweight='bold')
ax.set_ylabel('Y axis', fontweight='bold')
ax.set_title('Vector Field Stream Plot', fontweight='bold', fontsize=16)

plt.colorbar(strm.lines, ax=ax, label='Velocity Magnitude')

plt.tight_layout()
plt.show()
```

## Error Bars and Confidence Intervals

### Error bar plot
```python
import matplotlib.pyplot as plt
import numpy as np

x = np.arange(0, 10, 1)
y = x ** 2
error = x * 2

fig, ax = plt.subplots(figsize=(10, 6))

ax.errorbar(x, y, yerr=error, fmt='o-', linewidth=2, markersize=8,
            capsize=5, capthick=2, ecolor='red', elinewidth=2,
            label='Data with error bars')

ax.set_xlabel('X values', fontweight='bold')
ax.set_ylabel('Y values', fontweight='bold')
ax.set_title('Error Bar Plot', fontweight='bold', fontsize=16)
ax.legend()
ax.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

### Confidence interval bands
```python
import matplotlib.pyplot as plt
import numpy as np

x = np.linspace(0, 10, 100)
y = np.sin(x)
confidence = 0.2

fig, ax = plt.subplots(figsize=(12, 6))

# Plot line with confidence band
ax.plot(x, y, 'b-', linewidth=2, label='Mean')
ax.fill_between(x, y - confidence, y + confidence, 
                alpha=0.3, color='blue', label='95% CI')

ax.set_xlabel('X axis', fontweight='bold')
ax.set_ylabel('Y axis', fontweight='bold')
ax.set_title('Confidence Interval Band', fontweight='bold', fontsize=16)
ax.legend()
ax.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```
