# Matplotlib API Reference

Comprehensive reference for pyplot and axes methods.

## Figure and Axes Creation

### Creating figures
```python
import matplotlib.pyplot as plt

# Simple figure
fig = plt.figure()

# Figure with size and DPI
fig = plt.figure(figsize=(10, 6), dpi=100)

# Figure with constrained layout
fig = plt.figure(constrained_layout=True)

# Figure with custom facecolor
fig = plt.figure(facecolor='#f5f5f5')
```

### Creating subplots
```python
# Single subplot
fig, ax = plt.subplots()

# Grid of subplots
fig, axes = plt.subplots(2, 3, figsize=(15, 10))

# Subplots with shared axes
fig, axes = plt.subplots(2, 2, sharex=True, sharey=True)

# Complex subplot layout
fig = plt.figure(figsize=(12, 8))
ax1 = plt.subplot(2, 2, 1)
ax2 = plt.subplot(2, 2, 2)
ax3 = plt.subplot(2, 1, 2)  # Spans bottom row

# GridSpec for complex layouts
from matplotlib.gridspec import GridSpec

fig = plt.figure(figsize=(12, 8))
gs = GridSpec(3, 3, figure=fig)

ax1 = fig.add_subplot(gs[0, :])    # Top row, all columns
ax2 = fig.add_subplot(gs[1, :-1])  # Middle row, first 2 columns
ax3 = fig.add_subplot(gs[1:, -1])  # Middle+bottom rows, last column
ax4 = fig.add_subplot(gs[-1, 0])   # Bottom row, first column
ax5 = fig.add_subplot(gs[-1, 1])   # Bottom row, second column
```

## Plot Types

### Line plots
```python
# Basic line
ax.plot(x, y)

# Styled line
ax.plot(x, y, 'r--', linewidth=2, label='Data', alpha=0.7)

# Multiple lines
ax.plot(x, y1, x, y2, x, y3)

# Line with markers
ax.plot(x, y, 'o-', markersize=8, markerfacecolor='red', 
        markeredgecolor='black', markeredgewidth=1.5)
```

### Scatter plots
```python
# Basic scatter
ax.scatter(x, y)

# Styled scatter
ax.scatter(x, y, s=sizes, c=colors, cmap='viridis', 
           alpha=0.6, edgecolors='black', linewidth=0.5)

# Scatter with custom markers
ax.scatter(x, y, marker='^', s=100)
```

### Bar plots
```python
# Vertical bars
ax.bar(x, height, width=0.8, color='blue', edgecolor='black')

# Horizontal bars
ax.barh(y, width, height=0.8, color='green')

# Stacked bars
ax.bar(x, y1, label='Series 1')
ax.bar(x, y2, bottom=y1, label='Series 2')

# Grouped bars
width = 0.35
ax.bar(x - width/2, y1, width, label='Group 1')
ax.bar(x + width/2, y2, width, label='Group 2')
```

### Histograms
```python
# Basic histogram
ax.hist(data, bins=30)

# Styled histogram
ax.hist(data, bins=50, color='skyblue', edgecolor='black', 
        alpha=0.7, density=True)

# Multiple histograms
ax.hist([data1, data2, data3], bins=30, alpha=0.6, 
        label=['Dataset 1', 'Dataset 2', 'Dataset 3'])

# 2D histogram
ax.hist2d(x, y, bins=30, cmap='Blues')
```

### Box plots
```python
# Basic box plot
ax.boxplot(data)

# Styled box plot
ax.boxplot([data1, data2, data3], labels=['A', 'B', 'C'],
           patch_artist=True, notch=True, showmeans=True)

# Horizontal box plot
ax.boxplot(data, vert=False)
```

### Pie charts
```python
# Basic pie
ax.pie(sizes, labels=labels)

# Styled pie
ax.pie(sizes, labels=labels, autopct='%1.1f%%', startangle=90,
       explode=(0.1, 0, 0, 0), shadow=True, colors=colors)
```

### Heatmaps
```python
# Imshow for heatmaps
im = ax.imshow(data, cmap='hot', aspect='auto')
plt.colorbar(im, ax=ax)

# With custom vmin/vmax
im = ax.imshow(data, cmap='RdYlBu_r', vmin=-1, vmax=1)

# Interpolation
im = ax.imshow(data, interpolation='bilinear', cmap='viridis')
```

### Contour plots
```python
# Filled contour
cs = ax.contourf(X, Y, Z, levels=20, cmap='coolwarm')
plt.colorbar(cs, ax=ax)

# Line contour
cs = ax.contour(X, Y, Z, levels=10, colors='black')
ax.clabel(cs, inline=True, fontsize=10)

# Combined
ax.contourf(X, Y, Z, levels=20, cmap='viridis', alpha=0.8)
ax.contour(X, Y, Z, levels=10, colors='black', linewidths=0.5)
```

### Error bars
```python
# Vertical error bars
ax.errorbar(x, y, yerr=error, fmt='o-', capsize=5, capthick=2)

# Horizontal error bars
ax.errorbar(x, y, xerr=error_x, fmt='s-')

# Both directions
ax.errorbar(x, y, xerr=error_x, yerr=error_y, fmt='d-')

# Asymmetric errors
ax.errorbar(x, y, yerr=[lower_error, upper_error], fmt='o-')
```

### Fill between
```python
# Simple fill
ax.fill_between(x, y1, y2, alpha=0.3)

# Conditional fill
ax.fill_between(x, y1, y2, where=(y2 > y1), alpha=0.3, 
                color='green', label='Positive')
ax.fill_between(x, y1, y2, where=(y2 <= y1), alpha=0.3, 
                color='red', label='Negative')

# Fill to axis
ax.fill_between(x, 0, y, alpha=0.3)
```

## Axes Configuration

### Limits and scaling
```python
# Set limits
ax.set_xlim([0, 10])
ax.set_ylim([-1, 1])

# Get current limits
xlim = ax.get_xlim()
ylim = ax.get_ylim()

# Auto-scale
ax.autoscale()
ax.autoscale(axis='x')  # Only x-axis

# Log scale
ax.set_xscale('log')
ax.set_yscale('log')

# Symmetric log scale
ax.set_yscale('symlog')
```

### Labels and titles
```python
# Basic labels
ax.set_xlabel('X axis')
ax.set_ylabel('Y axis')
ax.set_title('Plot Title')

# Styled labels
ax.set_xlabel('X axis', fontsize=14, fontweight='bold', color='blue')
ax.set_ylabel('Y axis', fontsize=14, fontweight='bold')
ax.set_title('Plot Title', fontsize=16, fontweight='bold', pad=20)

# LaTeX in labels
ax.set_xlabel(r'$\alpha$ (radians)')
ax.set_ylabel(r'$\sin(\alpha)$')
ax.set_title(r'$y = \sin(x)$')
```

### Ticks and tick labels
```python
# Set tick locations
ax.set_xticks([0, 1, 2, 3, 4, 5])
ax.set_yticks(np.linspace(0, 1, 11))

# Set tick labels
ax.set_xticklabels(['A', 'B', 'C', 'D', 'E', 'F'])

# Tick parameters
ax.tick_params(axis='both', which='major', labelsize=12, 
               width=2, length=6, color='blue')
ax.tick_params(axis='both', which='minor', width=1, length=3)

# Minor ticks
ax.minorticks_on()

# Rotate tick labels
ax.tick_params(axis='x', rotation=45)
plt.setp(ax.get_xticklabels(), rotation=45, ha='right')

# Format tick labels
from matplotlib.ticker import FuncFormatter

def currency(x, pos):
    return f'${x:.2f}'

ax.yaxis.set_major_formatter(FuncFormatter(currency))
```

### Grid
```python
# Basic grid
ax.grid(True)

# Styled grid
ax.grid(True, which='major', color='gray', linestyle='-', 
        linewidth=0.5, alpha=0.5)
ax.grid(True, which='minor', color='gray', linestyle=':', 
        linewidth=0.5, alpha=0.3)

# Grid on specific axis
ax.grid(True, axis='x')
ax.grid(True, axis='y')
```

### Spines
```python
# Hide spines
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)

# Style spines
ax.spines['left'].set_linewidth(2)
ax.spines['bottom'].set_linewidth(2)
ax.spines['left'].set_color('blue')

# Move spines
ax.spines['left'].set_position(('data', 0))
ax.spines['bottom'].set_position(('data', 0))
```

### Legend
```python
# Basic legend
ax.legend()

# Positioned legend
ax.legend(loc='upper right')
ax.legend(loc='upper left', bbox_to_anchor=(1, 1))

# Styled legend
ax.legend(frameon=True, fancybox=True, shadow=True, 
          ncol=2, fontsize=12, title='Legend Title')

# Custom legend entries
from matplotlib.patches import Patch

legend_elements = [
    Patch(facecolor='blue', label='Category A'),
    Patch(facecolor='red', label='Category B')
]
ax.legend(handles=legend_elements)
```

## Annotations and Text

### Text
```python
# Simple text
ax.text(0.5, 0.5, 'Text', transform=ax.transAxes)

# Styled text
ax.text(x, y, 'Label', fontsize=12, fontweight='bold', 
        color='red', ha='center', va='center',
        bbox=dict(boxstyle='round', facecolor='wheat', alpha=0.5))
```

### Annotations
```python
# Simple annotation
ax.annotate('Point', xy=(x, y))

# Annotation with arrow
ax.annotate('Important', xy=(x, y), xytext=(x+1, y+1),
            arrowprops=dict(arrowstyle='->', color='black', lw=2))

# Styled annotation
ax.annotate('Peak', xy=(x, y), xytext=(x+1, y+1),
            fontsize=12, fontweight='bold',
            arrowprops=dict(arrowstyle='->', connectionstyle='arc3,rad=0.3',
                          color='red', lw=2),
            bbox=dict(boxstyle='round', facecolor='yellow', alpha=0.7))
```

## Colormaps and Colors

### Using colormaps
```python
# Built-in colormaps
cmap = plt.get_cmap('viridis')
cmap = plt.get_cmap('plasma')
cmap = plt.get_cmap('coolwarm')

# Reversed colormap
cmap = plt.get_cmap('viridis_r')

# Custom colormap
from matplotlib.colors import LinearSegmentedColormap

colors = ['blue', 'white', 'red']
n_bins = 256
cmap = LinearSegmentedColormap.from_list('custom', colors, N=n_bins)
```

### Color specification
```python
# Named colors
color = 'red'
color = 'blue'

# Hex colors
color = '#FF5733'

# RGB tuples
color = (0.5, 0.2, 0.8)

# RGBA with alpha
color = (0.5, 0.2, 0.8, 0.5)
```

### Colorbars
```python
# Basic colorbar
cbar = plt.colorbar(im, ax=ax)

# Styled colorbar
cbar = plt.colorbar(im, ax=ax, orientation='horizontal', 
                   label='Value', shrink=0.8, aspect=20)

# Colorbar ticks
cbar.set_ticks([0, 0.5, 1])
cbar.set_ticklabels(['Low', 'Medium', 'High'])
```

## Styling

### rcParams
```python
# Set parameters
plt.rcParams['figure.figsize'] = (10, 6)
plt.rcParams['font.size'] = 12
plt.rcParams['axes.labelsize'] = 14
plt.rcParams['axes.titlesize'] = 16
plt.rcParams['lines.linewidth'] = 2

# Update multiple parameters
plt.rcParams.update({
    'font.family': 'sans-serif',
    'font.sans-serif': ['Arial'],
    'axes.grid': True,
    'grid.alpha': 0.3
})

# Reset to defaults
plt.rcdefaults()
```

### Styles
```python
# Use built-in style
plt.style.use('seaborn-v0_8-darkgrid')
plt.style.use('ggplot')
plt.style.use('dark_background')

# List available styles
print(plt.style.available)

# Temporary style with context
with plt.style.context('seaborn-v0_8-whitegrid'):
    # Plots here use this style
    pass

# Custom style file
plt.style.use('path/to/custom_style.mplstyle')
```

## Saving Figures

### Basic save
```python
plt.savefig('figure.png')
```

### Save with options
```python
# High DPI for publication
plt.savefig('figure.png', dpi=300)

# Tight bounding box
plt.savefig('figure.png', bbox_inches='tight')

# Transparent background
plt.savefig('figure.png', transparent=True)

# Custom facecolor
plt.savefig('figure.png', facecolor='white', edgecolor='none')

# Multiple formats
plt.savefig('figure.png', dpi=300, bbox_inches='tight')
plt.savefig('figure.pdf', bbox_inches='tight')
plt.savefig('figure.svg', bbox_inches='tight')
```

## Event Handling

### Mouse events
```python
def on_click(event):
    print(f'Clicked at x={event.xdata}, y={event.ydata}')

fig.canvas.mpl_connect('button_press_event', on_click)
```

### Key events
```python
def on_key(event):
    print(f'Pressed: {event.key}')

fig.canvas.mpl_connect('key_press_event', on_key)
```

## 3D Plotting

### Setup 3D axes
```python
from mpl_toolkits.mplot3d import Axes3D

fig = plt.figure()
ax = fig.add_subplot(111, projection='3d')

# Or
fig, ax = plt.subplots(subplot_kw={'projection': '3d'})
```

### 3D plot types
```python
# 3D line
ax.plot(x, y, z, 'b-', linewidth=2)

# 3D scatter
ax.scatter(x, y, z, c=colors, cmap='viridis', s=50)

# 3D surface
ax.plot_surface(X, Y, Z, cmap='coolwarm', alpha=0.8)

# 3D wireframe
ax.plot_wireframe(X, Y, Z, color='black', linewidth=0.5)

# 3D contour
ax.contour(X, Y, Z, levels=10, cmap='viridis')
```

### 3D view control
```python
# Set viewing angle
ax.view_init(elev=20, azim=45)

# Set labels
ax.set_xlabel('X axis')
ax.set_ylabel('Y axis')
ax.set_zlabel('Z axis')

# Set limits
ax.set_xlim([-10, 10])
ax.set_ylim([-10, 10])
ax.set_zlim([0, 20])
```

## Common Patterns

### Multiple y-axes
```python
fig, ax1 = plt.subplots()

# First y-axis
ax1.plot(x, y1, 'b-')
ax1.set_ylabel('Y1', color='b')
ax1.tick_params(axis='y', labelcolor='b')

# Second y-axis
ax2 = ax1.twinx()
ax2.plot(x, y2, 'r-')
ax2.set_ylabel('Y2', color='r')
ax2.tick_params(axis='y', labelcolor='r')
```

### Inset axes
```python
from mpl_toolkits.axes_grid1.inset_locator import inset_axes

# Create inset
axins = inset_axes(ax, width='40%', height='40%', loc='upper right')

# Plot in inset
axins.plot(x, y)
```

### Custom legends
```python
# Legend outside plot
ax.legend(bbox_to_anchor=(1.05, 1), loc='upper left')

# Legend below plot
ax.legend(loc='upper center', bbox_to_anchor=(0.5, -0.15), ncol=3)

# Two-column legend
ax.legend(ncol=2)
```
