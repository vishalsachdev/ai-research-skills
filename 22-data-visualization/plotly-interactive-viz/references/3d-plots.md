# Plotly 3D Visualizations

Complete guide to 3D plotting in Plotly.

## 3D Scatter Plots

```python
import plotly.express as px
import plotly.graph_objects as go
import numpy as np

# Using Plotly Express
df = px.data.iris()
fig = px.scatter_3d(df, x='sepal_length', y='sepal_width', z='petal_width',
                    color='species', size='petal_length')
fig.show()

# Using Graph Objects for more control
n = 500
fig = go.Figure(data=[go.Scatter3d(
    x=np.random.randn(n),
    y=np.random.randn(n),
    z=np.random.randn(n),
    mode='markers',
    marker=dict(
        size=5,
        color=np.random.randn(n),
        colorscale='Viridis',
        showscale=True
    )
)])

fig.update_layout(
    scene=dict(
        xaxis_title='X Axis',
        yaxis_title='Y Axis',
        zaxis_title='Z Axis'
    ),
    title='3D Scatter Plot'
)
fig.show()
```

## 3D Surface Plots

```python
import plotly.graph_objects as go
import numpy as np

# Create mesh data
x = np.linspace(-5, 5, 50)
y = np.linspace(-5, 5, 50)
X, Y = np.meshgrid(x, y)
Z = np.sin(np.sqrt(X**2 + Y**2))

# Create surface plot
fig = go.Figure(data=[go.Surface(
    x=X, y=Y, z=Z,
    colorscale='Viridis',
    contours=dict(
        z=dict(show=True, usecolormap=True, project=dict(z=True))
    )
)])

fig.update_layout(
    title='3D Surface Plot',
    scene=dict(
        camera=dict(eye=dict(x=1.5, y=1.5, z=1.3))
    )
)
fig.show()
```

## 3D Line Plots

```python
import plotly.graph_objects as go
import numpy as np

t = np.linspace(0, 10, 100)
x = np.sin(t)
y = np.cos(t)
z = t

fig = go.Figure(data=[go.Scatter3d(
    x=x, y=y, z=z,
    mode='lines',
    line=dict(color=z, colorscale='Plasma', width=5)
)])

fig.update_layout(title='3D Parametric Curve')
fig.show()
```

## 3D Mesh Plots

```python
import plotly.graph_objects as go
import numpy as np

# Create mesh (sphere)
u = np.linspace(0, 2*np.pi, 50)
v = np.linspace(0, np.pi, 50)
x = 10 * np.outer(np.cos(u), np.sin(v))
y = 10 * np.outer(np.sin(u), np.sin(v))
z = 10 * np.outer(np.ones(np.size(u)), np.cos(v))

fig = go.Figure(data=[go.Mesh3d(
    x=x.flatten(),
    y=y.flatten(),
    z=z.flatten(),
    opacity=0.8,
    color='lightblue'
)])

fig.update_layout(title='3D Mesh (Sphere)')
fig.show()
```

## Camera and View Control

```python
# Set camera position
fig.update_layout(
    scene_camera=dict(
        eye=dict(x=2, y=2, z=1.5),  # Camera position
        center=dict(x=0, y=0, z=0),  # Look at point
        up=dict(x=0, y=0, z=1)       # Up direction
    )
)

# Orthographic projection
fig.update_layout(
    scene_camera_projection_type='orthographic'
)
```

## Resources

- **3D Charts**: https://plotly.com/python/3d-charts/
- **3D Surface Plots**: https://plotly.com/python/3d-surface-plots/
