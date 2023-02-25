---
layout: page
title: Principal Component Analysis on sample datasets
description: Skills: sklearn, scipy, seaborn
img:
importance: 2
category: fun
---

# Introduction 
The goal of this toy project was to use new statistical and visualization libraries to conduct PCA/CCA on sample data. The libraries and functions used in this project include: scikit-learn and scipy for linear decomposition and conducting Principal Componenet Analysis and Canonical Componenet Analysis and seaborn for new visualization tecniques. 

The first sample dataset contains time series of four variables: x1, x2, x3 and x4.
<br>
The following analysis was done:

## a) Plot the time series for each variable.
{% raw %}
```python
plt.figure(figsize=(10,5))
sns.set_style("darkgrid", {"axes.facecolor": ".9"})
ax = df.plot()
ax.set_title('Timeseries for each variable')
ax.legend(frameon=False, loc='lower center', ncol=len(col_names))
ax.axis([0,40,-4.5,4.5])
plt.savefig('1a')
```
{% endraw %}

<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/pca_1.png" title="PCA 1" class="img-fluid z-depth-1"%}
    </div>
</div>


## b) Perform PCA on the data.

To perform PCA I normalized the data, then used the [PCA](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.PCA.html) function from sklearn library. 


{% raw %}
```python
#normalize data and check it out
data_norm = (df - df.mean())/df.std()
## Run PCA
n_modes = np.min(np.shape(data_norm))
pca = PCA(n_components = n_modes)
PCs = pca.fit_transform(data_norm)
eigvecs = pca.components_
fracVar = pca.explained_variance_ratio_
n=2
print(np.sum(fracVar[:n])*100)  #sum of the first n modes = total percent variance explained by the first n eigen vectors
```
{% endraw %}

## c) Plot fraction of variance explained by each mode, and keep significant modes to reconstruct the data

{% raw %}
```python
plt.figure(figsize=(10,5))

plt.subplot(1,2,1)
plt.scatter(range(len(fracVar)),fracVar)
plt.xlabel('Mode Number')
plt.ylabel('Fraction Variance Explained')
plt.title('Variance Explained by All Modes')

plt.subplot(1,2,2)
n_modes_show = n
plt.scatter(range(n_modes_show),fracVar[:n_modes_show])
plt.xlabel('Mode Number')
plt.ylabel('Fraction Variance Explained')
plt.title('Variance Explained by First ' + str(n_modes_show) + ' Modes')
plt.tight_layout()
plt.show()
```
{% endraw %}

Using the plots below I decide which modes I want to keep to reconstruct the data. I keep the first 2 modes because they explain 97.6% of all the variance. Eventhough there is a drop off after the first mode, the reason I kept the first two was because just the first one explained only 59.8% of the variance.

<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/pca_2.png" title="PCA 2" class="img-fluid z-depth-1"%}
    </div>
</div>


## d) Plot the PCs of the significant modes (i.e. those that you kept) in time. Briefly discuss the 
results (what are these plots telling you?) 
e) Plot PC1 vs PC2. Discuss any feature that you find interesting.

To give your project a background in the portfolio page, just add the img tag to the front matter like so:

    ---
    layout: page
    title: project
    description: a project with a background image
    img: /assets/img/12.jpg
    ---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/1.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/3.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/5.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Caption photos easily. On the left, a road goes through a tunnel. Middle, leaves artistically fall in a hipster photoshoot. Right, in another hipster photoshoot, a lumberjack grasps a handful of pine needles.
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/5.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    This image can also have a caption. It's like magic.
</div>

You can also put regular text between your rows of images.
Say you wanted to write a little bit about your project before you posted the rest of the images.
You describe how you toiled, sweated, *bled* for your project, and then... you reveal its glory in the next row of images.


<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.html path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.html path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    You can also have artistically styled 2/3 + 1/3 images, like these.
</div>


The code is simple.
Just wrap your images with `<div class="col-sm">` and place them inside `<div class="row">` (read more about the <a href="https://getbootstrap.com/docs/4.4/layout/grid/">Bootstrap Grid</a> system).
To make images responsive, add `img-fluid` class to each; for rounded corners and shadows use `rounded` and `z-depth-1` classes.
Here's the code for the last row of images above:

{% raw %}
```html
<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.html path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.html path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
```
{% endraw %}
