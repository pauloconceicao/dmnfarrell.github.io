---
layout: post
title:  "Javascript callbacks for linking bokeh plots to panel widgets"
date:   2019-09-15 15:46:00
categories: plotting
tags: [bokeh,pyviz,python]
thumbnail: /img/bokeh_js_links.gif
---

## Background

Both Bokeh and Panel use widgets to control plot elements. (These tools have been covered in a [previous post](pyviz-panel)). Widgets can be linked to other widgets and plots in a few different ways. One way is to define functions in JavaScript that are called when the widget or plot element is changed. This is used to provide specialized behaviours in response to changes in a plot for example. In this example I wanted to have a slider that pans a plot left and right. The plot is also draggable, so the slider should be updated to reflect the plot movements also. This is a two way link between plot and widget that can be achieved with the `jslink` function in Panel.

## Code

Note: if you want to run this inside a jupyter lab you need to install the extensions:

```
jupyter labextension install @bokeh/jupyter_bokeh
jupyter labextension install @pyviz/jupyterlab_pyviz
```

This example uses a plot with colored boxes though the exact content is not important. You can install Bokeh and Panel with pip or conda.

```python
import random
from bokeh.models import ColumnDataSource
from bokeh.plotting import figure
from bokeh.models.glyphs import Text, Rect

def random_colors(size, seed=30):
    """random list of html colors of length sizes"""

    random.seed = seed    
    colors = ["#"+''.join([random.choice('0123456789ABCDEF') for j in range(6)])
                for i in range(size)]
    return colors

def dummy_plot(start=0,end=500,plot_width=1000,callback=None):

    m=3
    length=end
    x = np.arange(0,end,2)   
    y = list(range(1,m)) * len(x)
    y = y[:len(x)]
    colors = random_colors(len(y))
    w = [random.random()*3 for i in x]    
    source = ColumnDataSource({'x':x,'y':y,'w':w,'color':colors})
        x_range = (0,end)
    viewlen = end-start
    tools="xpan,xwheel_zoom,box_select"
    p = figure(title=None, plot_width=plot_width, plot_height=200, x_range=x_range,
                y_range=(0,m), tools=tools, min_border=0, toolbar_location='right')
    rects = Rect(x="x", y="y", width="w", height=.6, fill_color='color',
                fill_alpha=0.8, line_width=2, name='rects')
    p.add_glyph(source, rects)
    p.x_range.end=100
    return p
```

We then create the slider and plot and link them with jslink. Note that the code that does the updating is a JS code snippet, not a Python function. `source` and `target` are special keywords used in the JavaScript namespace to allow the widget values to be addressed.

```python
#create a slider
s = pnw.IntSlider(name='value',start=0,end=100,value=20)
p = dummy_plot(plot_width=800)
#wrap the figure in a pane so we can use jslink
pp = pn.panel(p)
#link the plot range to the value of the slider
pp.jslink(s, **{'x_range.start': 'value'})
#code for updating the plot range when the slider is moved
jsupdateplot = '''
    r = target.x_range.end - target.x_range.start
    target.x_range.start = source.value
    target.x_range.end = source.value+r
'''
#link to the plot object
s.jslink(p, code={'value': jsupdateplot})
#display the elements
pn.Column(s, pp)
```

The final behaviour is as shown below with both dragging slider and plot updating the other widget.

  <div style="width: 650px;">
  <a href="/img/bokeh_js_links.gif"> <img src="/img/bokeh_js_links.gif" width="600px"></a>
  </div>

## Links

* [Linking objects in Panel](https://panel.pyviz.org/user_guide/Links.html)
* [JavaScript Callbacks in Bokeh](https://bokeh.pydata.org/en/latest/docs/user_guide/interaction/callbacks.html)
