# Seaborn Colors and Styling

Complete guide to color palettes, themes, and styling in Seaborn.

## Color Palettes

### Categorical Palettes
Use for distinct categories (no inherent order).

```python
import seaborn as sns
import matplotlib.pyplot as plt

# View palette
sns.palplot(sns.color_palette('Set2'))
plt.show()

# Use palette
sns.set_palette('Set2')

# Available categorical palettes
palettes = ['Set1', 'Set2', 'Set3', 'Paired', 'Accent', 
            'Pastel1', 'Pastel2', 'Dark2', 'tab10', 'tab20']
```

### Sequential Palettes
Use for data ranging from low to high values.

```python
# Light to dark
sns.set_palette('Blues')
sns.set_palette('Greens')
sns.set_palette('Reds')

# Perceptually uniform
sns.set_palette('viridis')
sns.set_palette('plasma')
sns.set_palette('inferno')
sns.set_palette('magma')
sns.set_palette('cividis')
```

### Diverging Palettes
Use for data with critical midpoint (e.g., zero, neutral).

```python
# Cool to warm
sns.set_palette('coolwarm')
sns.set_palette('RdBu_r')
sns.set_palette('RdYlGn')
sns.set_palette('Spectral')
sns.set_palette('seismic')
```

### Custom Palettes
```python
# Custom colors
custom = ['#FF6B6B', '#4ECDC4', '#45B7D1', '#F7DC6F']
sns.set_palette(custom)

# Blend colors
sns.blend_palette(['white', 'red'], n_colors=10)

# Light palette from single color
sns.light_palette('navy', n_colors=8)

# Dark palette from single color
sns.dark_palette('green', n_colors=8, reverse=True)

# Diverging palette
sns.diverging_palette(250, 10, n=10)  # From hue 250 to 10
```

## Themes and Styles

### Built-in Styles
```python
# Available styles
styles = ['darkgrid', 'whitegrid', 'dark', 'white', 'ticks']

# Set style
sns.set_theme(style='whitegrid')
sns.set_theme(style='ticks')

# With custom parameters
sns.set_theme(style='whitegrid', 
              rc={'axes.facecolor': '#f5f5f5'})
```

### Context Scaling
Scale all plot elements together for different use cases.

```python
# Contexts (smallest to largest)
sns.set_context('paper')      # Journal papers
sns.set_context('notebook')   # Default, Jupyter notebooks
sns.set_context('talk')       # Presentations
sns.set_context('poster')     # Conference posters

# Custom scaling
sns.set_context('notebook', 
                font_scale=1.5,  # Make fonts 50% larger
                rc={'lines.linewidth': 2.5})
```

### Complete Theme Configuration
```python
sns.set_theme(
    style='whitegrid',
    palette='muted',
    context='notebook',
    font='sans-serif',
    font_scale=1.2,
    rc={
        'figure.figsize': (10, 6),
        'axes.labelsize': 14,
        'axes.titlesize': 16,
        'xtick.labelsize': 12,
        'ytick.labelsize': 12,
        'legend.fontsize': 12,
        'grid.linewidth': 0.8,
        'lines.linewidth': 2,
    }
)
```

## Color Theory for Visualizations

### Choosing the Right Palette

**Categorical data** (distinct groups):
- Set1, Set2, tab10, husl
- Maximum ~12 categories
- Example: Species, countries, product types

**Sequential data** (low → high):
- Blues, Greens, viridis, plasma
- Single hue (Blues) or multi-hue (viridis)
- Example: Temperature, price, age

**Diverging data** (negative ← 0 → positive):
- coolwarm, RdBu_r, seismic
- Critical midpoint emphasized
- Example: Correlation, change from baseline

### Accessibility Considerations
```python
# Colorblind-friendly palettes
sns.color_palette('colorblind')

# High contrast
sns.set_palette('bright')

# Test your palette
from colorspacious import cspace_convert

def check_colorblind(palette):
    """Simulate deuteranopia (green-blind)"""
    rgb = sns.color_palette(palette, as_cmap=False)
    cvd_rgb = [cspace_convert(c, "sRGB1", "sRGB1+CVD", 
                               cvd_type='deuteranomaly') 
               for c in rgb]
    return cvd_rgb
```

## Style Examples

### Publication-ready style
```python
import seaborn as sns
import matplotlib.pyplot as plt

sns.set_theme(
    style='ticks',
    context='paper',
    font_scale=1.1,
    rc={
        'figure.figsize': (7, 5),
        'figure.dpi': 300,
        'savefig.dpi': 300,
        'savefig.bbox': 'tight',
        'font.family': 'serif',
        'font.serif': ['Times New Roman'],
    }
)

# Remove top and right spines
sns.despine()
```

### Presentation style
```python
sns.set_theme(
    style='white',
    context='talk',
    font_scale=1.3,
    palette='bright',
    rc={
        'figure.figsize': (12, 8),
        'axes.linewidth': 2,
        'grid.linewidth': 1.5,
    }
)
```

### Dark mode
```python
plt.style.use('dark_background')
sns.set_palette('bright')
```

## rcParams Reference

Common rcParams for customization:

```python
custom_params = {
    # Figure
    'figure.figsize': (10, 6),
    'figure.dpi': 100,
    'savefig.dpi': 300,
    'savefig.bbox': 'tight',
    'savefig.pad_inches': 0.1,
    
    # Fonts
    'font.size': 12,
    'font.family': 'sans-serif',
    'axes.labelsize': 14,
    'axes.titlesize': 16,
    'xtick.labelsize': 12,
    'ytick.labelsize': 12,
    'legend.fontsize': 12,
    
    # Lines and markers
    'lines.linewidth': 2,
    'lines.markersize': 8,
    
    # Grid
    'axes.grid': True,
    'grid.alpha': 0.3,
    'grid.linewidth': 0.8,
    
    # Spines
    'axes.linewidth': 1.5,
    'axes.edgecolor': '#333333',
    
    # Colors
    'axes.facecolor': 'white',
    'figure.facecolor': 'white',
    
    # Legend
    'legend.frameon': True,
    'legend.framealpha': 0.8,
    'legend.fancybox': True,
}

sns.set_theme(rc=custom_params)
```

## Palette Utilities

### Generate custom palettes
```python
# Evenly spaced hues
sns.color_palette('husl', n_colors=10)

# Cubehelix (varying brightness and hue)
sns.cubehelix_palette(n_colors=10, start=0, rot=0.4)

# Light/dark gradients
sns.light_palette('green', n_colors=8)
sns.dark_palette('purple', n_colors=8)

# Blend two colors
sns.blend_palette(['white', 'red', 'black'], n_colors=10)
```

### View palettes
```python
# Display palette
sns.palplot(sns.color_palette('Set2'))

# Compare palettes
fig, axes = plt.subplots(5, 1, figsize=(10, 8))
palettes = ['Set1', 'Set2', 'Pastel1', 'Dark2', 'Paired']

for ax, pal in zip(axes, palettes):
    colors = sns.color_palette(pal)
    ax.barh([0]*len(colors), [1]*len(colors), 
            color=colors, height=1)
    ax.set_xlim(0, len(colors))
    ax.axis('off')
    ax.text(-0.5, 0, pal, va='center', fontweight='bold')

plt.tight_layout()
plt.show()
```

## Resources

- **Color Palette Tool**: https://seaborn.pydata.org/tutorial/color_palettes.html
- **ColorBrewer**: https://colorbrewer2.org/
- **Matplotlib Colormaps**: https://matplotlib.org/stable/tutorials/colors/colormaps.html
