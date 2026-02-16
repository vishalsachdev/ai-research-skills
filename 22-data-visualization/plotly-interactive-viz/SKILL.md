---
name: plotly-interactive-viz
description: Provides interactive web-based data visualizations with zoom, pan, and hover tooltips. Use when building dashboards, creating 3D plots, or needing browser-based interactivity with export to HTML. Supports Python, R, and JavaScript with 40+ chart types and Dash integration for web apps.
version: 1.0.0
author: Orchestra Research
license: MIT
tags: [Data Visualization, Plotly, Interactive Plots, Dashboards, 3D Visualization, Web Visualization, Python]
dependencies: [plotly>=5.18.0, pandas>=2.0.0, numpy>=1.24.0]
---

# Plotly - Interactive Web Visualizations

## Quick start

Plotly creates interactive, publication-quality graphs that can be viewed in web browsers, Jupyter notebooks, or standalone HTML files.

**Installation**:
```bash
pip install plotly pandas numpy kaleido  # kaleido for static image export
```

**Basic usage**:
```python
import plotly.express as px
import plotly.graph_objects as go
import pandas as pd

# Plotly Express (high-level, simple)
df = px.data.iris()
fig = px.scatter(df, x='sepal_width', y='sepal_length', 
                 color='species', size='petal_length',
                 hover_data=['petal_width'])
fig.show()

# Graph Objects (low-level, customizable)
fig = go.Figure(data=go.Scatter(x=[1, 2, 3], y=[2, 4, 3],
                                mode='lines+markers'))
fig.update_layout(title='Interactive Line Plot')
fig.show()

# Save as HTML
fig.write_html('plot.html')
```

## Common workflows

### Workflow 1: Create interactive dashboard visualization

Copy this checklist:

```
Interactive Dashboard Workflow:
☐ Load data into pandas DataFrame
☐ Choose appropriate plot types (scatter, line, bar, etc.)
☐ Create plots with Plotly Express
☐ Add interactivity (hover tooltips, dropdowns)
☐ Configure layout and styling
☐ Add annotations and shapes
☐ Test interactivity (zoom, pan, hover)
☐ Export as HTML or integrate with Dash
```

**Implementation**:
```python
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import pandas as pd

# Load data
df = px.data.stocks()

# Create subplots
fig = make_subplots(
    rows=2, cols=2,
    subplot_titles=('Stock Prices', 'Price Distribution', 
                    'Daily Returns', 'Correlation Heatmap'),
    specs=[[{'type': 'scatter'}, {'type': 'histogram'}],
           [{'type': 'scatter'}, {'type': 'heatmap'}]]
)

# 1. Line plot with multiple traces
for column in ['GOOG', 'AAPL', 'AMZN']:
    fig.add_trace(
        go.Scatter(x=df['date'], y=df[column], name=column,
                  mode='lines', line=dict(width=2)),
        row=1, col=1
    )

# 2. Histogram
fig.add_trace(
    go.Histogram(x=df['GOOG'], nbinsx=30, name='GOOG',
                marker_color='blue'),
    row=1, col=2
)

# 3. Scatter plot with trend line
fig.add_trace(
    go.Scatter(x=df['date'], y=df['GOOG'] - df['GOOG'].shift(1),
              mode='markers', name='Daily Returns',
              marker=dict(size=5, color='red')),
    row=2, col=1
)

# 4. Correlation heatmap
corr = df[['GOOG', 'AAPL', 'AMZN', 'MSFT']].corr()
fig.add_trace(
    go.Heatmap(z=corr.values, x=corr.columns, y=corr.index,
              colorscale='RdBu', zmid=0, text=corr.values,
              texttemplate='%{text:.2f}', textfont={"size": 12}),
    row=2, col=2
)

# Update layout
fig.update_layout(
    height=800,
    showlegend=True,
    title_text='Stock Analysis Dashboard',
    title_font_size=20,
    hovermode='x unified'
)

# Save as interactive HTML
fig.write_html('dashboard.html')
fig.show()
```

### Workflow 2: Build 3D visualization

Copy this checklist:

```
3D Visualization Workflow:
☐ Prepare 3D data (X, Y, Z coordinates)
☐ Choose 3D plot type (surface, scatter, mesh)
☐ Create 3D plot with appropriate parameters
☐ Configure camera and viewing angle
☐ Add color mapping and scales
☐ Customize axes labels and title
☐ Test 3D rotation and interaction
☐ Export for web or presentation
```

**Implementation**:
```python
import plotly.graph_objects as go
import numpy as np

# Generate 3D data
x = np.linspace(-5, 5, 50)
y = np.linspace(-5, 5, 50)
X, Y = np.meshgrid(x, y)
Z = np.sin(np.sqrt(X**2 + Y**2))

# Create 3D surface plot
fig = go.Figure(data=[
    go.Surface(
        x=X, y=Y, z=Z,
        colorscale='Viridis',
        colorbar=dict(title='Z Value'),
        contours=dict(
            z=dict(show=True, usecolormap=True, 
                   highlightcolor="limegreen", project=dict(z=True))
        )
    )
])

# Update layout
fig.update_layout(
    title='Interactive 3D Surface Plot',
    scene=dict(
        xaxis_title='X Axis',
        yaxis_title='Y Axis',
        zaxis_title='Z Axis',
        camera=dict(
            eye=dict(x=1.5, y=1.5, z=1.3)
        )
    ),
    width=900,
    height=700
)

fig.write_html('3d_surface.html')
fig.show()

# 3D Scatter plot
np.random.seed(42)
n = 500
fig = go.Figure(data=[
    go.Scatter3d(
        x=np.random.randn(n),
        y=np.random.randn(n),
        z=np.random.randn(n),
        mode='markers',
        marker=dict(
            size=5,
            color=np.random.randn(n),
            colorscale='Plasma',
            showscale=True,
            colorbar=dict(title='Value')
        )
    )
])

fig.update_layout(
    title='3D Scatter Plot',
    scene=dict(
        xaxis_title='X',
        yaxis_title='Y',
        zaxis_title='Z'
    )
)

fig.show()
```

### Workflow 3: Animated time series visualization

Copy this checklist:

```
Animated Visualization Workflow:
☐ Prepare time-series data with time column
☐ Choose animation parameter (time, category)
☐ Create plot with animation_frame argument
☐ Configure animation settings (duration, transition)
☐ Add play/pause controls
☐ Test animation smoothness
☐ Add slider for manual control
☐ Export as HTML with embedded animation
```

**Implementation**:
```python
import plotly.express as px

# Load data with time component
df = px.data.gapminder()

# Create animated scatter plot
fig = px.scatter(
    df,
    x='gdpPercap',
    y='lifeExp',
    animation_frame='year',
    animation_group='country',
    size='pop',
    color='continent',
    hover_name='country',
    log_x=True,
    size_max=60,
    range_x=[100, 100000],
    range_y=[25, 90],
    title='Life Expectancy vs GDP Per Capita Over Time'
)

# Customize animation
fig.layout.updatemenus[0].buttons[0].args[1]['frame']['duration'] = 300
fig.layout.updatemenus[0].buttons[0].args[1]['transition']['duration'] = 200

# Update layout
fig.update_layout(
    width=1000,
    height=600,
    xaxis_title='GDP per Capita (log scale)',
    yaxis_title='Life Expectancy (years)',
    font=dict(size=14)
)

fig.write_html('animated_gapminder.html')
fig.show()
```

## When to use vs alternatives

**Use Plotly when**:
- Need interactive web-based visualizations
- Building dashboards or web applications
- Require zoom, pan, hover tooltips, click events
- Creating 3D visualizations
- Exporting plots to HTML for web embedding
- Need cross-platform compatibility (works in any browser)

**Use Matplotlib instead when**:
- Creating static publication figures
- Need fine-grained control over every element
- Working in environments without web browser
- Generating plots for print media
- Building animations for video export

**Use Seaborn instead when**:
- Creating statistical visualizations quickly
- Need beautiful defaults for statistical plots
- Working extensively with pandas DataFrames
- Don't need interactivity

**Use Dash when**:
- Building full web applications
- Need complex user interactions
- Creating production dashboards
- Require backend data processing with UI updates

## Core plot types

**Basic charts**: Scatter, line, bar, histogram, box, violin - See [references/chart-types.md](references/chart-types.md)

**3D plots**: Surface, scatter3d, mesh3d, volume - See [references/3d-plots.md](references/3d-plots.md)

**Maps and geospatial**: Choropleth, scatter_geo, mapbox - See [references/maps.md](references/maps.md)

## Plotly Express vs Graph Objects

### Plotly Express (recommended for most use cases)
```python
import plotly.express as px

# High-level, concise, automatic styling
df = px.data.iris()
fig = px.scatter(df, x='sepal_width', y='sepal_length', 
                 color='species', title='Iris Dataset')
fig.show()
```

**Use when**: Quick visualizations, standard chart types, working with DataFrames

### Graph Objects (for advanced customization)
```python
import plotly.graph_objects as go

# Low-level, full control
fig = go.Figure()
fig.add_trace(go.Scatter(x=[1, 2, 3], y=[2, 4, 3],
                        mode='lines+markers',
                        name='Series 1',
                        line=dict(color='blue', width=3)))
fig.update_layout(title='Custom Plot')
fig.show()
```

**Use when**: Complex customization, multiple traces, custom layouts

## Common issues

### Issue 1: Plot not showing in Jupyter

**Symptoms**: `fig.show()` doesn't display plot in notebook

**Solution**:
```python
# Install notebook renderer
pip install nbformat

# Set default renderer for Jupyter
import plotly.io as pio
pio.renderers.default = 'notebook'

# Or use inline renderer
fig.show(renderer='notebook')
```

### Issue 2: HTML file too large

**Symptoms**: Exported HTML is many megabytes

**Solution**:
```python
# Use CDN instead of embedding full library
fig.write_html('plot.html', include_plotlyjs='cdn')

# Or reduce data points
df_sample = df.sample(n=1000)  # Sample data
fig = px.scatter(df_sample, x='x', y='y')

# Or use WebGL for large datasets
fig = px.scatter(df, x='x', y='y', render_mode='webgl')
```

### Issue 3: Legend blocking data

**Symptoms**: Legend covers important parts of visualization

**Solution**:
```python
# Move legend outside plot area
fig.update_layout(
    legend=dict(
        yanchor="top",
        y=0.99,
        xanchor="left",
        x=1.01
    )
)

# Or position at bottom
fig.update_layout(
    legend=dict(
        orientation="h",
        yanchor="bottom",
        y=-0.2,
        xanchor="center",
        x=0.5
    )
)
```

### Issue 4: Date axis formatting issues

**Symptoms**: Dates displayed incorrectly or with strange formatting

**Solution**:
```python
# Ensure dates are datetime objects
df['date'] = pd.to_datetime(df['date'])

# Custom date format
fig.update_xaxes(
    tickformat='%Y-%m-%d',
    tickangle=-45
)

# Or use rangeslider
fig.update_xaxes(rangeslider_visible=True)
```

### Issue 5: Poor performance with large datasets

**Solution**: Use WebGL rendering (`render_mode='webgl'`), sample data, or aggregate time series. See [references/performance.md](references/performance.md)

### Issue 6: Colors not matching

**Solution**: Use `color_discrete_sequence`, `color_continuous_scale`, or `color_discrete_map`. See [references/chart-types.md](references/chart-types.md)

### Issue 7: Static image export not working

**Solution**: Install kaleido engine with `pip install kaleido`, then use `fig.write_image('plot.png')`

## Advanced features

**Chart types**: See [references/chart-types.md](references/chart-types.md) for all 40+ chart types

**3D visualizations**: See [references/3d-plots.md](references/3d-plots.md) for surface, mesh, and volume plots

**Geographic maps**: See [references/maps.md](references/maps.md) for choropleth and scatter maps

## Resources

- **Official Documentation**: https://plotly.com/python/
- **Gallery**: https://plotly.com/python/plotly-express/ (100+ examples)
- **API Reference**: https://plotly.com/python-api-reference/
- **Dash Framework**: https://dash.plotly.com/ (for web apps)
- **Community Forum**: https://community.plotly.com/
