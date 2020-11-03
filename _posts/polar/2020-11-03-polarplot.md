---
layout:     post
title:      Polar plot
subtitle:   special
date:       2020-11-03
author:     Kongyue
header-img: post/11.jpg
catalog: true
tags:
    - ATMOS
    - python
    - pie
---

```python
import numpy as np
import matplotlib.pyplot as plt

class Radar(object):

  def __init__(self, fig, titles, label, rect=None):
    if rect is None:
        rect = [0.05, 0.15, 0.95, 0.75]

    self.n = len(titles)
    self.angles = [a if a <=360. else a - 360. for a in np.arange(90, 90+360, 360.0/self.n)]
    self.axes = [fig.add_axes(rect, projection="polar", label="axes%d" % i) 
                    for i in range(self.n)]

    self.ax = self.axes[0]

    # Show the labels
    self.ax.set_thetagrids(self.angles, labels=titles, fontsize=14, weight="bold", color="black")

    for ax in self.axes[1:]:
        ax.patch.set_visible(False)
        ax.grid(False)
        ax.xaxis.set_visible(False)
        self.ax.yaxis.grid(False)

    for ax, angle in zip(self.axes, self.angles):
        ax.set_rgrids(range(1, 6), labels=label, angle=angle, fontsize=12)
        # hide outer spine (circle)
        ax.spines["polar"].set_visible(False)
        ax.set_ylim(0, 6)
        ax.xaxis.grid(True, color='black', linestyle='-', zorder=1)

        # draw a line on the y axis at each label
        ax.tick_params(axis='y', pad=0, left=True, length=6, width=1, direction='inout')

  def decorate_ticks(self, axes):
    for idx, tick in enumerate(axes.xaxis.majorTicks):
        # get the gridline
        gl = tick.gridline
        gl.set_marker('o')
        gl.set_markersize(15)
        if idx == 0:
            gl.set_markerfacecolor('#003399')
        elif idx == 1:
            gl.set_markerfacecolor('#336666')
        elif idx == 2:
            gl.set_markerfacecolor('#336699')
        elif idx == 3:
            gl.set_markerfacecolor('#CC3333')
        elif idx == 4:
            gl.set_markerfacecolor('#CC9933')
        # this doesn't get used. The center doesn't seem to be different than 5
        else:
            gl.set_markerfacecolor('#000000')

        if idx == 0 or idx == 3:
            tick.set_pad(10)
        else:
            tick.set_pad(30)
    axes.plot(0, 0, 'o', markersize=15, markerfacecolor='m', markeredgecolor='k')

  def plot(self, values, *args, **kw):
    angle = np.deg2rad(np.r_[self.angles, self.angles[0]])
    values = np.r_[values, values[0]]
    self.ax.plot(angle, values, *args, **kw)

fig = plt.figure(1)

titles = ['TG01', 'TG02', 'TG03', 'TG04', 'TG05', 'TG06']
label = list("ABCDE")

radar = Radar(fig, titles, label)
radar.plot([3.75, 3.25, 3.0, 2.75, 4.25, 3.5], "-", linewidth=2, color="b",   alpha=.7, label="Data01")
radar.plot([3.25, 2.25, 2.25, 2.25, 1.5, 1.75],"-", linewidth=2, color="r", alpha=.7, label="Data02")

radar.decorate_ticks(radar.ax)

# this avoids clipping the markers below the thetagrid labels
radar.ax.xaxis.grid(clip_on = False)

radar.ax.legend(loc='upper center', bbox_to_anchor=(0.5, -0.10),
  fancybox=True, shadow=True, ncol=4)

plt.show()
```

![png](/fig/201103/polar1.jpg){:width="500px"}



```python
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.ticker import (MultipleLocator, FormatStrFormatter,
                               AutoMinorLocator)
#Define Angle Range
measured_thetas = np.linspace(np.pi/3, 5*np.pi/6, 10)
calculated_thetas = np.linspace(4*np.pi/3, 6*np.pi/3, 10)

#Genarate radial data 
measured_rs = np.random.uniform(3, 5, len(measured_thetas))
calculated_rs = np.random.uniform(2, 4, len(calculated_thetas))

ax = plt.subplot(111, projection='polar')
# offset Radial Axis works with Matplotlib > 2.2.3
ax.set_rorigin(0)
ax.set_ylim(2, 6)

# Plot series data and Legend
ax.plot(measured_thetas, measured_rs, c='b', label="Calculated")
ax.plot(calculated_thetas, calculated_rs, c='r', label="Measured")
ax.legend(loc="center",frameon=False,fontsize =  'x-small')

#Set Radial Axes labels
ax.set_rlabel_position(np.rad2deg((min(measured_thetas))))

# Set Radial Axis Titles
label_position=ax.get_rlabel_position()
ax.text(np.math.radians(label_position-10),(ax.get_rmax()+2)/2.,'Measured',
        rotation= 60,ha='center',va='center')
ax.text(np.math.radians(np.rad2deg((min(calculated_thetas)))-10),(ax.get_rmax()+2)/2.,"Calculated",
        rotation= 60,ha='center',va='center')

# Set Gridlines
ax.set_rticks([*np.arange(2,7,1)], minor=False)  # Less radial ticks

# Adjust ticks to data, taking different step sizes into account
ax.set_xticks([
    *np.arange(min(measured_thetas), max(measured_thetas) + np.deg2rad(1), np.deg2rad(30)),
    *np.arange(min(calculated_thetas), max(calculated_thetas) + np.deg2rad(1), np.deg2rad(15)),
], minor = False)

# Turn on the minor TICKS, which are required for the minor GRID
ax.minorticks_on()
# For the minor ticks, use no labels; default NullFormatter.
ax.xaxis.set_minor_locator(AutoMinorLocator(2))
ax.yaxis.set_minor_locator(AutoMinorLocator(2))
# Customize the major grid
ax.grid(which='major', linestyle='-', linewidth='0.25', color='black')
# Customize the minor grid
ax.grid(which='minor', linestyle='--', linewidth='0.15', color='black')

 # to control how far the scale is from the plot (axes coordinates)
def add_scale(ax, X_OFF, Y_OFF):
    # add extra axes for the scale
    X_OFFSET = X_OFF
    Y_OFFSET = Y_OFF
    rect = ax.get_position()
    rect = (rect.xmin-X_OFFSET, rect.ymin+rect.height/2-Y_OFFSET, # x, y
            rect.width, rect.height/2) # width, height
    scale_ax = ax.figure.add_axes(rect)
   # if (X_OFFSET >= 0):
        # hide most elements of the new axes
    for loc in ['right', 'top', 'bottom']:
        scale_ax.spines[loc].set_visible(False)
#    else:
#        for loc in ['right', 'top', 'bottom']:
#            scale_ax.spines[loc].set_visible(False)
    scale_ax.tick_params(bottom=False, labelbottom=False)
    scale_ax.patch.set_visible(False) # hide white background
    # adjust the scale
    scale_ax.spines['left'].set_bounds(*ax.get_ylim())
    # scale_ax.spines['left'].set_bounds(0, ax.get_rmax()) # mpl < 2.2.3
    scale_ax.set_yticks(ax.get_yticks())
    scale_ax.set_ylim(ax.get_rorigin(), ax.get_rmax())
    # scale_ax.set_ylim(ax.get_ylim()) # Matplotlib < 2.2.3


#Dummy Chart to hide unused gridlines 
padding_degree = 5    
dummy_thetas1 = np.linspace(0 + np.deg2rad(padding_degree), min(measured_thetas) - np.deg2rad(padding_degree), 100)
dummy_thetas2 = np.linspace(max(measured_thetas)+ np.deg2rad(padding_degree), min(calculated_thetas)- np.deg2rad(padding_degree), 100)

#Genrate Values 
dummy_r =  np.ones(len(dummy_thetas1))*float(max(ax.get_ylim())+0.1)

ax.plot(dummy_thetas1, dummy_r, c='y',  alpha = 1 ,linewidth = 30, ls = 'solid')
ax.plot(dummy_thetas2, dummy_r, c='y',alpha = 1,linewidth = 30, ls = 'solid') 





add_scale(ax,0.1,0.5)
add_scale(ax,-0.6,0)

plt.show()
```
![png](/fig/201103/polar2.jpg){:width="500px"}



