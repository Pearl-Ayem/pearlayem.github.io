---
layout: page
title: Contrast stretching on satellite data
description: Skills - python, sklearn, rasterio, TIFF 
img: contrast_stretch_3 - Copy.png
importance: 1
category: fun
---

# Introduction 
This project was carried out as an experiment to understand contrast stretching on Landsat satellite imagery. Constrast stretching is a normalization technique to enhance the image quality by stretching the range of intesity values. The method allows us to create better images from satellite data captured at high temporal resolutions, but in partially cloud conditions. 


### 1. Read bands 3,4 and 5 (green, red and near-ir) and convert the image to double-precision (64-bit) floating point format:
{% raw %}
```python
tiff_file=list(data_directory.glob("*B1.TIF"))[0]
meta_file=list(data_directory.glob("*MTL.txt"))[0]
jpeg_file=list(data_directory.glob("*T1.jpg"))[0]

out=landsat_metadata(meta_file)
meta_dict = out.__dict__

with pil_image.open(tiff_file) as img:
    tiff_meta_dict = {TAGS[key] : img.tag[key] for key in img.tag.keys()}
    ch1 = img_as_float(img)
    
img_eq = exposure.equalize_hist(ch1)  
plt.hist(ch1.ravel());
```
{% endraw %}

### 2. Draw histogram of intensity from the float obtained above


<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets\img\contrast_stretch_1 - Copy.png" title="float histogram" class="img-fluid z-depth-1"%}
    </div>
</div>

### 3. Creating plot of the low contrast image and high intensity normalized image
{% raw %}
```python
plt.imshow(ch1)
plt.imshow(img_eq[1000:4000,1500:5000],interpolation="nearest");
```
{% endraw %}

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="\assets\img\contrast_stretch_2 - Copy.png" title="low contrast image" class="img-fluid z-depth-1" %}
    </div>
</div>
<div class="caption">
    Low contrast original image
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets\img\contrast_stretch_3 - Copy.png" title="high intensity image" class="img-fluid z-depth-1" %}
    </div>
</div>
<div class="caption">
    High intensity normalized image
</div>

### 4. Convert high contrast image to map, with shapefile for borders. Create histogram of the new image floats.

{% raw %}
```python
from matplotlib import pyplot as plt
import cartopy
zone=meta_dict['UTM_ZONE']
zone_code = 32633
#https://epsg.io/32610
crs = cartopy.crs.epsg(zone_code)
fig, ax = plt.subplots(1, 1,figsize=[5,5],subplot_kw={'projection': crs})
ax.imshow(img_eq, origin='lower', extent=[xmin, xmax, ymin, ymax], transform=crs, 
          interpolation='nearest')
ax.coastlines(resolution='10m',color='red',lw=1)
ax.set_extent([xmin,xmax,ymin,ymax],crs)
```
{% endraw %}

<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets\img\contrast_stretch_4 - Copy.png" title="high intensity map" class="img-fluid z-depth-1"%}
    </div>
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets\img\contrast_stretch_5 - Copy.png" title="high intensity image" class="img-fluid z-depth-1" %}
    </div>
</div>
<div class="caption">
    High intensity normalized image histogram
</div>

### 5. Plot comparision of image quality with its histograms
{% raw %}
```python

%matplotlib inline
import matplotlib
import matplotlib.pyplot as plt
import numpy as np

from skimage import data, img_as_float
from skimage import exposure


matplotlib.rcParams['font.size'] = 8


def plot_img_and_hist(image, axes, bins=256):
    """Plot an image along with its histogram and cumulative histogram.

    """
    image = img_as_float(image)
    ax_img, ax_hist = axes
    ax_cdf = ax_hist.twinx()

    # Display image
    ax_img.imshow(image, cmap=plt.cm.gray)
    ax_img.set_axis_off()

    # Display histogram
    ax_hist.hist(image.ravel(), bins=bins, histtype='step', color='black')
    ax_hist.ticklabel_format(axis='y', style='scientific', scilimits=(0, 0))
    ax_hist.set_xlabel('Pixel intensity')
    ax_hist.set_xlim(0, 1)
    ax_hist.set_yticks([])

    # Display cumulative distribution
    img_cdf, bins = exposure.cumulative_distribution(image, bins)
    ax_cdf.plot(bins, img_cdf, 'r')
    ax_cdf.set_yticks([])

    return ax_img, ax_hist, ax_cdf


# Load an example image
img = np.array(pil_image.open(tiff_file))

# Contrast stretching
p2, p98 = np.percentile(img, (2, 98))
img_rescale = exposure.rescale_intensity(img, in_range=(p2, p98))

# Equalization
img_eq = exposure.equalize_hist(img)

# Adaptive Equalization
img_adapteq = exposure.equalize_adapthist(img, clip_limit=0.01)

# Display results
fig = plt.figure(figsize=(8, 5))
axes = np.zeros((2, 4), dtype=np.object)
axes[0, 0] = fig.add_subplot(2, 4, 1)
for i in range(1, 4):
    axes[0, i] = fig.add_subplot(2, 4, 1+i, sharex=axes[0,0], sharey=axes[0,0])
for i in range(0, 4):
    axes[1, i] = fig.add_subplot(2, 4, 5+i)

ax_img, ax_hist, ax_cdf = plot_img_and_hist(img, axes[:, 0])
ax_img.set_title('Low contrast image')

y_min, y_max = ax_hist.get_ylim()
ax_hist.set_ylabel('Number of pixels')
ax_hist.set_yticks(np.linspace(0, y_max, 5))

ax_img, ax_hist, ax_cdf = plot_img_and_hist(img_rescale, axes[:, 1])
ax_img.set_title('Contrast stretching')

ax_img, ax_hist, ax_cdf = plot_img_and_hist(img_eq, axes[:, 2])
ax_img.set_title('Histogram equalization')

ax_img, ax_hist, ax_cdf = plot_img_and_hist(img_adapteq, axes[:, 3])
ax_img.set_title('Adaptive equalization')

ax_cdf.set_ylabel('Fraction of total intensity')
ax_cdf.set_yticks(np.linspace(0, 1, 5))

# prevent overlap of y-axis labels
fig.tight_layout()
plt.show()
```
{% endraw %}

<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/contrast_stretch_6.png" title="contrast stretching vs hist" class="img-fluid z-depth-1"%}
    </div>
</div>







