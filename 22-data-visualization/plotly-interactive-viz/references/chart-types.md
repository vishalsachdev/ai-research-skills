# Plotly Chart Types Reference

Comprehensive guide to Plotly's 40+ chart types.

## Statistical Charts

### Scatter Plots
```python
import plotly.express as px

df = px.data.iris()

# Basic scatter
fig = px.scatter(df, x='sepal_width', y='sepal_length')

# With color, size, and hover
fig = px.scatter(df, x='sepal_width', y='sepal_length',
                 color='species', size='petal_length',
                 hover_data=['petal_width'])

# With trend line
fig = px.scatter(df, x='sepal_width', y='sepal_length',
                 trendline='ols', trendline_color_override='red')

fig.show()
```

### Line Charts
```python
import plotly.express as px

df = px.data.stocks()

# Multi-line plot
fig = px.line(df, x='date', y=['GOOG', 'AAPL', 'AMZN'])

# With markers
fig = px.line(df, x='date', y='GOOG', markers=True)

fig.show()
```

### Bar Charts
```python
import plotly.express as px

df = px.data.tips()

# Vertical bars
fig = px.bar(df, x='day', y='total_bill', color='sex')

# Horizontal bars
fig = px.bar(df, x='total_bill', y='day', orientation='h')

# Stacked bars
fig = px.bar(df, x='day', y='total_bill', color='sex',
             barmode='stack')

fig.show()
```

### Histograms
```python
import plotly.express as px

df = px.data.tips()

# Basic histogram
fig = px.histogram(df, x='total_bill', nbins=30)

# With color grouping
fig = px.histogram(df, x='total_bill', color='sex',
                   marginal='box')  # box, violin, or rug

fig.show()
```

### Box and Violin Plots
```python
import plotly.express as px

df = px.data.tips()

# Box plot
fig = px.box(df, x='day', y='total_bill', color='time',
             notched=True, points='all')

# Violin plot
fig = px.violin(df, x='day', y='total_bill', color='time',
                box=True, points='all')

fig.show()
```

## Financial Charts

### Candlestick Charts
```python
import plotly.graph_objects as go
import pandas as pd

# Sample OHLC data
df = pd.DataFrame({
    'date': pd.date_range('2024-01-01', periods=30),
    'open': [100 + i + np.random.randn() for i in range(30)],
    'high': [105 + i + np.random.randn() for i in range(30)],
    'low': [95 + i + np.random.randn() for i in range(30)],
    'close': [102 + i + np.random.randn() for i in range(30)]
})

fig = go.Figure(data=[go.Candlestick(
    x=df['date'],
    open=df['open'],
    high=df['high'],
    low=df['low'],
    close=df['close']
)])

fig.update_layout(title='Candlestick Chart')
fig.show()
```

### OHLC Charts
```python
import plotly.graph_objects as go

fig = go.Figure(data=[go.Ohlc(
    x=df['date'],
    open=df['open'],
    high=df['high'],
    low=df['low'],
    close=df['close']
)])

fig.show()
```

## Matrix and Heatmaps

### Heatmap
```python
import plotly.express as px

# Correlation matrix
df = px.data.iris()
corr = df.select_dtypes(include='number').corr()

fig = px.imshow(corr, text_auto=True, color_continuous_scale='RdBu_r',
                zmin=-1, zmax=1)
fig.update_layout(title='Correlation Heatmap')
fig.show()
```

### Animated Heatmap
```python
import plotly.express as px

df = px.data.gapminder()
pivot = df.pivot_table(values='lifeExp', index='continent', 
                       columns='year')

fig = px.imshow(pivot, color_continuous_scale='Viridis',
                labels=dict(x="Year", y="Continent", color="Life Expectancy"))
fig.show()
```

## Multidimensional Charts

### Parallel Coordinates
```python
import plotly.express as px

df = px.data.iris()
fig = px.parallel_coordinates(df, color='species_id',
                              dimensions=['sepal_width', 'sepal_length',
                                        'petal_width', 'petal_length'])
fig.show()
```

### Parallel Categories
```python
import plotly.express as px

df = px.data.tips()
fig = px.parallel_categories(df, dimensions=['sex', 'smoker', 'day'],
                             color='size', color_continuous_scale='Viridis')
fig.show()
```

### Radar Chart
```python
import plotly.graph_objects as go

categories = ['Speed', 'Reliability', 'Comfort', 'Safety', 'Efficiency']
values = [4, 3, 5, 5, 3]

fig = go.Figure()
fig.add_trace(go.Scatterpolar(
    r=values + [values[0]],  # Close the polygon
    theta=categories + [categories[0]],
    fill='toself',
    name='Product A'
))

fig.update_layout(
    polar=dict(radialaxis=dict(visible=True, range=[0, 5])),
    title='Radar Chart'
)
fig.show()
```

## Distribution Charts

### Density Heatmap
```python
import plotly.express as px

df = px.data.tips()
fig = px.density_heatmap(df, x='total_bill', y='tip')
fig.show()
```

### Density Contour
```python
import plotly.express as px

df = px.data.tips()
fig = px.density_contour(df, x='total_bill', y='tip',
                         marginal_x='histogram', marginal_y='histogram')
fig.show()
```

## Specialized Charts

### Sunburst Chart
```python
import plotly.express as px

df = px.data.tips()
fig = px.sunburst(df, path=['day', 'time', 'sex'], values='total_bill')
fig.show()
```

### Treemap
```python
import plotly.express as px

df = px.data.tips()
fig = px.treemap(df, path=['day', 'time', 'sex'], values='total_bill',
                 color='total_bill', color_continuous_scale='Viridis')
fig.show()
```

### Funnel Chart
```python
import plotly.express as px

data = {
    'Stage': ['Visit', 'Sign-up', 'Purchase', 'Return'],
    'Count': [1000, 500, 200, 50]
}

fig = px.funnel(data, x='Count', y='Stage')
fig.show()
```

### Sankey Diagram
```python
import plotly.graph_objects as go

fig = go.Figure(data=[go.Sankey(
    node=dict(
        pad=15,
        thickness=20,
        label=['A1', 'A2', 'B1', 'B2', 'C1', 'C2'],
        color='blue'
    ),
    link=dict(
        source=[0, 1, 0, 2, 3, 3],
        target=[2, 3, 3, 4, 4, 5],
        value=[8, 4, 2, 8, 4, 2]
    )
)])

fig.update_layout(title='Sankey Diagram')
fig.show()
```

## Color Configuration

### Discrete Colors
```python
# Use predefined color sequences
px.colors.qualitative.Plotly
px.colors.qualitative.Set1
px.colors.qualitative.Pastel

# Apply to plot
fig = px.scatter(df, x='x', y='y', color='category',
                color_discrete_sequence=px.colors.qualitative.Set1)
```

### Continuous Colors
```python
# Use continuous color scales
px.colors.sequential.Viridis
px.colors.sequential.Plasma
px.colors.diverging.RdBu

# Apply to plot
fig = px.scatter(df, x='x', y='y', color='value',
                color_continuous_scale='Viridis')
```

## Resources

- **Chart Types**: https://plotly.com/python/
- **Color Scales**: https://plotly.com/python/builtin-colorscales/
