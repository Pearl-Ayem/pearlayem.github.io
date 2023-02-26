---
layout: page
title: Cannonical Component Analysis on sample datasets
description: Skills- sklearn, scipy, seaborn, CCA, statistics
img: assets/img/cca_2.png
importance: 3
category: fun
---


# Introduction 
Canonical correlation analysis is used to identify and measure the associations among two sets of variables. Canonical correlation is appropriate in the same situations where multiple regression would be, but where are there are multiple intercorrelated outcome variables

The goal of this toy project was to use new statistical and visualization libraries to conduct CCA on sample datasets. The libraries and functions used in this project include: scikit-learn and scipy for linear decomposition and conducting Canonical Componenet Analysis and seaborn for new visualization tecniques. 

I will try to perform canonical correlation analysis (CCA) between the two datasets X and Y. Dataset X contains time series x1, x2, x3, and dataset Y contains time series y1, y2, y3.
<br>
The following analysis was done:
<br>

## a) Load CCA dataset, and split into the X and Y data to carry out analysis

Original Data


| x1 | x2 | x3 | y1 | y2 | y3 |
| :------------: | :------------: | :------------: | :------------: | :------------: | :------------: |
| 1.530845      | 0.316551      |-1.823282 | 1.215005 |1.273343 |0.026177 |
| 1.285960      | 0.823623      |-1.389774 | 1.579516 |0.621029 |0.804732 |
| 0.106213      | 0.578572      | 0.564672 | -0.018080 |-1.103449 |1.082170 |
| -0.937513     | 0.040444     | 1.064561 | -1.748572 |-1.387991 |-0.676495 |
| -1.516770     | -1.462519    |	1.001928 | -0.673814 | 0.430090 |-1.128205 |

<br>
Data is split into X and Y

{% raw %}
```python
n1,n2 = np.shape(data)
xdata = data.iloc[:,:3] # xdata has three variables, 200 observations
ydata = data.iloc[:,3:]  # ydata has three variables, 200 observations
t = range(n1)
```
{% endraw %}

<br>

## b) Plot the time series in 2-D space

{% raw %}
```python
plt.figure(figsize = (10,10))

plt.subplot(3,2,1)
plt.plot(xdata.iloc[:,0])
plt.title('x1')

plt.subplot(3,2,3)
plt.plot(xdata.iloc[:,1])
plt.title('x2')

plt.subplot(3,2,5)
plt.plot(xdata.iloc[:,2])
plt.title('x3')

plt.subplot(3,2,2)
plt.plot(ydata.iloc[:,0])
plt.title('y1')

plt.subplot(3,2,4)
plt.plot(ydata.iloc[:,1])
plt.title('y2')

plt.subplot(3,2,6)
plt.plot(ydata.iloc[:,2])
plt.title('y3')

plt.tight_layout()
plt.show()
```
{% endraw %}
<br>
<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/cca_1.png" title="CCA 1" class="img-fluid z-depth-1"%}
    </div>
</div>
<br>

## c) Plot the time series in 3-D space

{% raw %}
```python
fig = plt.figure(figsize=(16,6))

ax = fig.add_subplot(121,projection='3d')
ax.scatter(xdata.iloc[:,0],xdata.iloc[:,1],xdata.iloc[:,2],c=t)
plt.title('x space')

ax = fig.add_subplot(122,projection='3d')
p = ax.scatter(ydata.iloc[:,0],ydata.iloc[:,1],ydata.iloc[:,2],c=t)
plt.title('y space')
cbar = fig.colorbar(p)
cbar.set_label('time')
```
{% endraw %}

<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/cca_2.png" title="CCA 2" class="img-fluid z-depth-1"%}
    </div>
</div>
<br>

## d) Perform CCA on the dataset and plot output.

To perform PCA I normalized the data, then used the [PCA](https://scikit-learn.org/stable/modules/generated/sklearn.decomposition.PCA.html) function from sklearn library. 


{% raw %}
```python
n_modes = 3 #modes to keep
cca = CCA(n_components=n_modes,max_iter = 10000)
U,V = cca.fit_transform(xdata,ydata)
A = cca.x_weights_
B = cca.y_weights_
F = np.cov(xdata.T)@A
G = np.cov(ydata.T)@B
r = [np.corrcoef(U[:,ii],V[:,ii]) for ii in range(n_modes)]


plt.figure(figsize=(10,6))
for kk in range(n_modes):
    
    plt.subplot(n_modes,3,kk*3+1)
    plt.plot(F[:,kk])
    plt.title('F: Mode ' + str(kk+1))
    
    plt.subplot(n_modes,3,kk*3+2)
    plt.plot(G[:,kk])
    plt.title('G: Mode ' + str(kk+1))
    
    plt.subplot(n_modes,3,kk*3+3)
    plt.plot(U[:,kk])
    plt.plot(V[:,kk])
    plt.title('U and V: r = ' + str(r[kk][0][1])[:4], fontsize = 18)
    plt.legend(['U','V'])
    
plt.tight_layout()
plt.show()

FM1=F[:,0]  #Mode 1
FM2=F[:,1]  #Mode 2
FM3=F[:,2]  #Mode 3

GM1=G[:,0]  #Mode 1
GM2=G[:,1]  #Mode 2
GM3=G[:,2]  #Mode 3
```
{% endraw %}
<br>
F: <br>
[[ 0.00446775 | 1.38448709  | 0.74085775]<br>
 [ 0.36774811 | 1.18278841  | 0.65839886]<br>
 [ 0.35360286 | -1.20036722 | -0.64154713]]
<br>
<br>
G: 
[[-0.60781613 |  1.04238248  | -0.11125062]<br>
 [-1.21694479 | -0.0053953   |  0.14002947]<br>
 [ 0.62447444 |  1.02810331  | -0.23102376]]
<br>
<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/cca_3.png" title="CCA 3" class="img-fluid z-depth-1"%}
    </div>
</div>
<br>



## e) Plot vectors that correspond to the modes of high correlation in the 3-D x-space and in the 3-D y-space
Plot vector F1 and vector F2 in x-space, and G1 and G2 in y-space since the first 2 modes have high correlation

{% raw %}
```python

```
{% endraw %}
<br>
