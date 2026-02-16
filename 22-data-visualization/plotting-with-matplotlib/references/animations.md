# Matplotlib Animations

Complete guide to creating animations and saving them as videos or GIFs.

## Basic Animation Framework

Matplotlib provides the `animation` module for creating animated visualizations.

### Simple line animation
```python
import matplotlib.pyplot as plt
import matplotlib.animation as animation
import numpy as np

fig, ax = plt.subplots(figsize=(10, 6))

x = np.linspace(0, 2*np.pi, 100)
line, = ax.plot([], [], 'b-', linewidth=2)

ax.set_xlim(0, 2*np.pi)
ax.set_ylim(-1.5, 1.5)
ax.set_xlabel('X axis', fontweight='bold')
ax.set_ylabel('Y axis', fontweight='bold')
ax.set_title('Animated Sine Wave', fontweight='bold')
ax.grid(True, alpha=0.3)

def init():
    """Initialize animation"""
    line.set_data([], [])
    return line,

def animate(frame):
    """Animation function called sequentially"""
    y = np.sin(x + frame/10)
    line.set_data(x, y)
    return line,

# Create animation
anim = animation.FuncAnimation(fig, animate, init_func=init,
                               frames=100, interval=50, blit=True)

plt.show()
```

### Save animation as MP4
```python
import matplotlib.pyplot as plt
import matplotlib.animation as animation
import numpy as np

fig, ax = plt.subplots(figsize=(10, 6))

x = np.linspace(0, 2*np.pi, 100)
line, = ax.plot([], [], 'b-', linewidth=2)

ax.set_xlim(0, 2*np.pi)
ax.set_ylim(-1.5, 1.5)
ax.grid(True, alpha=0.3)

def init():
    line.set_data([], [])
    return line,

def animate(frame):
    y = np.sin(x + frame/10)
    line.set_data(x, y)
    return line,

anim = animation.FuncAnimation(fig, animate, init_func=init,
                               frames=100, interval=50, blit=True)

# Save as MP4 (requires ffmpeg)
anim.save('sine_wave.mp4', writer='ffmpeg', fps=30, dpi=150)

# Or save as GIF (requires pillow or imagemagick)
anim.save('sine_wave.gif', writer='pillow', fps=30)
```

## Advanced Animations

### Animated scatter plot
```python
import matplotlib.pyplot as plt
import matplotlib.animation as animation
import numpy as np

fig, ax = plt.subplots(figsize=(10, 8))

# Initialize empty scatter
scat = ax.scatter([], [], s=100, c=[], cmap='viridis', 
                 alpha=0.6, edgecolors='black', linewidth=0.5)

ax.set_xlim(-3, 3)
ax.set_ylim(-3, 3)
ax.set_xlabel('X axis', fontweight='bold')
ax.set_ylabel('Y axis', fontweight='bold')
ax.set_title('Animated Scatter Plot', fontweight='bold')
ax.grid(True, alpha=0.3)

# Add colorbar
cbar = plt.colorbar(scat, ax=ax)
cbar.set_label('Color Value', fontweight='bold')

def init():
    scat.set_offsets(np.empty((0, 2)))
    scat.set_array(np.array([]))
    return scat,

def animate(frame):
    # Generate random points that evolve over time
    n_points = 100
    theta = np.linspace(0, 2*np.pi, n_points) + frame/10
    r = 1 + 0.5 * np.sin(5*theta + frame/5)
    
    x = r * np.cos(theta)
    y = r * np.sin(theta)
    
    positions = np.column_stack((x, y))
    colors = theta
    
    scat.set_offsets(positions)
    scat.set_array(colors)
    
    return scat,

anim = animation.FuncAnimation(fig, animate, init_func=init,
                               frames=200, interval=50, blit=True)

plt.show()
```

### Animated bar chart (race chart)
```python
import matplotlib.pyplot as plt
import matplotlib.animation as animation
import numpy as np

fig, ax = plt.subplots(figsize=(12, 8))

categories = ['A', 'B', 'C', 'D', 'E', 'F', 'G', 'H']
n_categories = len(categories)

# Initialize bars
bars = ax.barh(categories, [0] * n_categories, color='steelblue', edgecolor='black')

ax.set_xlim(0, 100)
ax.set_xlabel('Value', fontweight='bold', fontsize=14)
ax.set_title('Animated Bar Chart Race', fontweight='bold', fontsize=16)
ax.grid(axis='x', alpha=0.3)

# Text objects for values
texts = []
for i, bar in enumerate(bars):
    text = ax.text(0, i, '', ha='left', va='center', fontweight='bold')
    texts.append(text)

def init():
    for bar in bars:
        bar.set_width(0)
    for text in texts:
        text.set_text('')
    return bars + texts

def animate(frame):
    # Simulate evolving values
    values = np.abs(50 + 30 * np.sin(frame/10 + np.arange(n_categories)))
    
    # Sort by values
    sorted_indices = np.argsort(values)[::-1]
    sorted_values = values[sorted_indices]
    sorted_categories = [categories[i] for i in sorted_indices]
    
    # Update bars
    for i, (bar, val) in enumerate(zip(bars, sorted_values)):
        bar.set_width(val)
        # Color by rank
        color = plt.cm.viridis(i / n_categories)
        bar.set_color(color)
        
        # Update value text
        texts[i].set_text(f'{val:.1f}')
        texts[i].set_position((val + 2, i))
    
    # Update y-axis labels
    ax.set_yticklabels(sorted_categories)
    
    return bars + texts

anim = animation.FuncAnimation(fig, animate, init_func=init,
                               frames=100, interval=100, blit=True)

plt.show()
```

### Animated heatmap
```python
import matplotlib.pyplot as plt
import matplotlib.animation as animation
import numpy as np

fig, ax = plt.subplots(figsize=(10, 8))

# Initialize heatmap
data = np.random.rand(10, 10)
im = ax.imshow(data, cmap='hot', aspect='auto', vmin=0, vmax=1)

ax.set_title('Animated Heatmap', fontweight='bold', fontsize=16)
plt.colorbar(im, ax=ax, label='Value')

def init():
    im.set_data(np.zeros((10, 10)))
    return [im]

def animate(frame):
    # Create evolving pattern
    x = np.linspace(-3, 3, 10)
    y = np.linspace(-3, 3, 10)
    X, Y = np.meshgrid(x, y)
    data = np.sin(np.sqrt(X**2 + Y**2) - frame/5)
    
    im.set_data(data)
    im.set_clim(vmin=data.min(), vmax=data.max())
    
    return [im]

anim = animation.FuncAnimation(fig, animate, init_func=init,
                               frames=100, interval=100, blit=True)

plt.show()
```

## 3D Animations

### Rotating 3D surface
```python
import matplotlib.pyplot as plt
import matplotlib.animation as animation
import numpy as np
from mpl_toolkits.mplot3d import Axes3D

fig = plt.figure(figsize=(12, 8))
ax = fig.add_subplot(111, projection='3d')

# Create surface data
u = np.linspace(0, 2*np.pi, 100)
v = np.linspace(0, np.pi, 100)
x = 10 * np.outer(np.cos(u), np.sin(v))
y = 10 * np.outer(np.sin(u), np.sin(v))
z = 10 * np.outer(np.ones(np.size(u)), np.cos(v))

# Plot surface
surf = ax.plot_surface(x, y, z, cmap='viridis', alpha=0.8,
                       linewidth=0, antialiased=True)

ax.set_xlabel('X axis', fontweight='bold')
ax.set_ylabel('Y axis', fontweight='bold')
ax.set_zlabel('Z axis', fontweight='bold')
ax.set_title('Rotating 3D Surface', fontweight='bold', fontsize=16)

def init():
    ax.view_init(elev=20, azim=0)
    return [surf]

def animate(frame):
    # Rotate view
    ax.view_init(elev=20, azim=frame)
    return [surf]

anim = animation.FuncAnimation(fig, animate, init_func=init,
                               frames=360, interval=50, blit=True)

plt.show()
```

### Animated 3D scatter
```python
import matplotlib.pyplot as plt
import matplotlib.animation as animation
import numpy as np
from mpl_toolkits.mplot3d import Axes3D

fig = plt.figure(figsize=(12, 8))
ax = fig.add_subplot(111, projection='3d')

# Initialize scatter
scat = ax.scatter([], [], [], c=[], cmap='plasma', s=50, alpha=0.6)

ax.set_xlim(-10, 10)
ax.set_ylim(-10, 10)
ax.set_zlim(-10, 10)
ax.set_xlabel('X axis', fontweight='bold')
ax.set_ylabel('Y axis', fontweight='bold')
ax.set_zlabel('Z axis', fontweight='bold')
ax.set_title('Animated 3D Scatter', fontweight='bold', fontsize=16)

def init():
    scat._offsets3d = ([], [], [])
    scat.set_array(np.array([]))
    return scat,

def animate(frame):
    # Generate spiraling points
    t = np.linspace(0, 4*np.pi, 200) + frame/10
    x = 10 * np.cos(t) * (1 - t/(4*np.pi))
    y = 10 * np.sin(t) * (1 - t/(4*np.pi))
    z = t
    
    colors = t
    
    scat._offsets3d = (x, y, z)
    scat.set_array(colors)
    
    # Rotate view
    ax.view_init(elev=20, azim=frame)
    
    return scat,

anim = animation.FuncAnimation(fig, animate, init_func=init,
                               frames=200, interval=50, blit=False)

plt.show()
```

## Interactive Updates (not animations)

For interactive plots that update based on user input or real-time data:

### Real-time data plotting
```python
import matplotlib.pyplot as plt
import numpy as np

plt.ion()  # Turn on interactive mode

fig, ax = plt.subplots(figsize=(12, 6))
line, = ax.plot([], [], 'b-', linewidth=2)

ax.set_xlim(0, 100)
ax.set_ylim(-1, 1)
ax.set_xlabel('Time', fontweight='bold')
ax.set_ylabel('Value', fontweight='bold')
ax.set_title('Real-time Data Stream', fontweight='bold')
ax.grid(True, alpha=0.3)

x_data = []
y_data = []

# Simulate real-time data
for i in range(100):
    x_data.append(i)
    y_data.append(np.sin(i/10) + np.random.randn()*0.1)
    
    line.set_data(x_data, y_data)
    
    # Auto-scale if needed
    ax.relim()
    ax.autoscale_view()
    
    fig.canvas.draw()
    fig.canvas.flush_events()
    
    plt.pause(0.05)

plt.ioff()  # Turn off interactive mode
plt.show()
```

## Saving Animations

### Save as MP4 video
```python
# Requires ffmpeg to be installed
# Install: conda install ffmpeg  or  apt-get install ffmpeg

anim.save('animation.mp4', writer='ffmpeg', fps=30, dpi=150,
          bitrate=1800, codec='libx264')
```

### Save as GIF
```python
# Requires pillow
# Install: pip install pillow

anim.save('animation.gif', writer='pillow', fps=30)

# Or with imagemagick for better quality
# Install imagemagick first
anim.save('animation.gif', writer='imagemagick', fps=30, dpi=100)
```

### Save as HTML5 video
```python
from IPython.display import HTML

# For Jupyter notebooks
HTML(anim.to_html5_video())

# Or save to file
with open('animation.html', 'w') as f:
    f.write(anim.to_html5_video())
```

### Custom writer configuration
```python
from matplotlib.animation import FFMpegWriter

# Configure custom writer
writer = FFMpegWriter(fps=30, bitrate=1800, codec='libx264',
                      extra_args=['-pix_fmt', 'yuv420p'])

anim.save('animation.mp4', writer=writer, dpi=150)
```

## Performance Tips

### Optimize animation performance
```python
# 1. Use blitting (only redraw changed artists)
anim = animation.FuncAnimation(fig, animate, blit=True)

# 2. Reduce number of objects
# Instead of recreating, update existing objects

# 3. Lower DPI for preview
fig = plt.figure(dpi=72)  # Lower DPI for speed

# 4. Cache expensive computations
# Precompute data if possible

# 5. Use smaller figure size
fig = plt.figure(figsize=(8, 6))  # Smaller = faster
```

### Profiling animations
```python
import time

def animate(frame):
    start = time.time()
    
    # Your animation code here
    
    elapsed = time.time() - start
    print(f"Frame {frame}: {elapsed*1000:.2f}ms")
    
    return artists
```

## Troubleshooting

### Animation not displaying
```python
# In Jupyter notebooks, use:
%matplotlib notebook

# Or:
from IPython.display import HTML
HTML(anim.to_html5_video())
```

### Video encoding errors
```bash
# Check if ffmpeg is installed
ffmpeg -version

# Install ffmpeg if missing:
# macOS: brew install ffmpeg
# Ubuntu: sudo apt-get install ffmpeg
# conda: conda install ffmpeg
```

### Memory issues with long animations
```python
# Save frames incrementally instead of keeping all in memory
from matplotlib.animation import FFMpegWriter

fig, ax = plt.subplots()
writer = FFMpegWriter(fps=30)

with writer.saving(fig, 'animation.mp4', dpi=150):
    for frame in range(1000):
        # Update plot
        ax.clear()
        # ... plot code ...
        
        writer.grab_frame()
```
