# Plotly Geographic Maps

Guide to creating interactive maps with Plotly.

## Choropleth Maps

```python
import plotly.express as px

# US state choropleth
df = px.data.election()
fig = px.choropleth(df, locations='district', locationmode='USA-states',
                    color='winner', scope='usa',
                    title='US Election Results')
fig.show()

# World choropleth
df = px.data.gapminder().query('year==2007')
fig = px.choropleth(df, locations='iso_alpha', color='lifeExp',
                    hover_name='country', 
                    color_continuous_scale='Viridis',
                    title='Life Expectancy by Country (2007)')
fig.show()
```

## Scatter Geo Maps

```python
import plotly.express as px

df = px.data.gapminder().query('year==2007')
fig = px.scatter_geo(df, locations='iso_alpha', size='pop',
                     hover_name='country', color='continent',
                     projection='natural earth',
                     title='Population by Country (2007)')
fig.show()
```

## Mapbox Maps

```python
import plotly.express as px

# Requires mapbox token (free at mapbox.com)
px.set_mapbox_access_token('your_token_here')

df = px.data.carshare()
fig = px.scatter_mapbox(df, lat='centroid_lat', lon='centroid_lon',
                        color='peak_hour', size='car_hours',
                        hover_name='centroid_lat',
                        zoom=10, height=600)
fig.show()

# Open Street Map (no token needed)
fig = px.scatter_mapbox(df, lat='centroid_lat', lon='centroid_lon',
                        color='peak_hour', size='car_hours',
                        mapbox_style='open-street-map',
                        zoom=10, height=600)
fig.show()
```

## Line Maps (Routes)

```python
import plotly.graph_objects as go

fig = go.Figure(data=go.Scattergeo(
    lon=[-100, -80, -60],
    lat=[40, 35, 30],
    mode='lines+markers',
    line=dict(width=2, color='red'),
    marker=dict(size=10)
))

fig.update_layout(
    title='Flight Route',
    geo=dict(scope='north america', projection_type='albers usa')
)
fig.show()
```

## Resources

- **Choropleth Maps**: https://plotly.com/python/choropleth-maps/
- **Scatter Geo**: https://plotly.com/python/scatter-plots-on-maps/
- **Mapbox**: https://plotly.com/python/mapbox-layers/
