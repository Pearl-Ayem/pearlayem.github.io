---
layout: page
title: Application of multiple linear regression and stepwise regression
description: Skills- sklearn, scipy, seaborn, CCA, statistics
img: assets/img/mlr_8.png
importance: 4
category: fun
---



# Introduction

There are four examples of applying multiple linear regression and stepwise
regression





## Example 1: Testing MLR on artificial data

Y = a0 + a1\*X1 + a2\*X2 + a3\*X3 + a4\*X4
<br>

### Load X data and look at the first few rows
``` python
X = pd.read_csv('Xdata.csv',names=['X1','X2','X3','X4'])
X1 = X['X1']
X2 = X['X2']
X3 = X['X3']
X4 = X['X4']
```
<br>

### Define regression coefficients, and calculate Y
``` python

a0 = 0
a1 = 1
a2 = -2
a3 = 3
a4 = -4

Y = a0 + a1*X1 + a2*X2 + a3*X3 + a4*X4
```
<br>

### Plot X and Y
``` python
plt.subplot(111)
plt.plot(Y)
plt.plot(X)
plt.legend(['Y','X1','X2','X3','X4'])
plt.show()
```

<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/mlr_1.png" title="MLR 1" class="img-fluid z-depth-1"%}
    </div>
</div>
<br>

### Apply MLR on X and Y to see if this method can recover the known coefficients
``` python

lm_MLR = linear_model.LinearRegression()
model = lm_MLR.fit(X,Y)
ypred_MLR = lm_MLR.predict(X) #y predicted by MLR
intercept_MLR = lm_MLR.intercept_ #intercept predicted by MLR
coef_MLR = lm_MLR.coef_ #regression coefficients in MLR model
R2_MLR = lm_MLR.score(X,Y) #R-squared value from MLR model
```
> MLR results:
<br> a0 = 1.1368683772161603e-13
<br> a1 = 1.0000000000000002
<br> a2 = -2.0000000000000027
<br> a3 = 3.0
<br> a4 = -4.0
<br>

### Use stepwise regression to find which predictors to use

``` python
result = stepwise_selection(X, Y)
print(result)
```
    Add  X4                             with p-value 1.96607e-11
    Add  X3                             with p-value 4.6339e-09
    Add  X2                             with p-value 1.94203e-20
    Add  X1                             with p-value 0.0
    resulting features:
    ['X4', 'X3', 'X2', 'X1']
<br>

### Do MLR using predictors chosen from stepwise regression

``` python

lm_step = linear_model.LinearRegression()
model_step = lm_step.fit(X[result],Y)
ypred_step = lm_step.predict(X[result]) #y predicted by MLR
intercept_step = lm_step.intercept_ #intercept predicted by MLR
coef_step = lm_step.coef_ #regression coefficients in MLR model
R2_step = lm_step.score(X[result],Y) #R-squared value from MLR model
```

### Visualize MLR and stepwise model performance

{.cell .code execution_count="9"}
``` python

ax1 = plt.subplot(121)
ax1.scatter(Y,ypred_MLR)
l1 = np.min(ax1.get_xlim())
l2 = np.max(ax1.get_xlim())
ax1.plot([l1,l2], [l1,l2], ls="--", c=".3")
plt.xlabel('Y')
plt.ylabel('Y_MLR')
plt.title('MLR')

ax2 = plt.subplot(122)
ax2.scatter(Y,ypred_step)
l1 = np.min(ax2.get_xlim())
l2 = np.max(ax2.get_xlim())
ax2.plot([l1,l2], [l1,l2], ls="--", c=".3")
plt.xlabel('Y')
plt.ylabel('Y_step')
plt.title('Stepwise')

plt.tight_layout()
plt.show()
```

<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/mlr_2.png" title="MLR 1" class="img-fluid z-depth-1"%}
    </div>
</div>
<br>


# Example 2

Y = a0 + a1*X1 + a2*X2 + a3*X3 + a4*X4 + Yrand

Add some random noise to Y and rerun MLR/stepwise
<br>

### Load Yrand and check it out
``` python

Yrand = pd.read_csv('Yrand.csv',header=None)[0]
Ynew = Y + 5*Yrand
plt.plot(Yrand)
plt.show()
```

<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/mlr_3.png" title="MLR 3" class="img-fluid z-depth-1"%}
    </div>
</div>
<br>

### plot Y, Ynew, and X
``` python


plt.subplot()
plt.plot(Y)
plt.plot(Ynew)
plt.plot(X)
plt.legend(['Y','YRand','X1','X2','X3','X4'])
plt.show()
```
<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/mlr_4.png" title="MLR 4" class="img-fluid z-depth-1"%}
    </div>
</div>
<br>


### Now: MLR
``` python


lm_MLR = linear_model.LinearRegression()
model = lm_MLR.fit(X,Ynew)
ypred_MLR = lm_MLR.predict(X) #y predicted by MLR
intercept_MLR = lm_MLR.intercept_ #intercept predicted by MLR
coef_MLR = lm_MLR.coef_ #regression coefficients in MLR model
R2_MLR = lm_MLR.score(X,Ynew) #R-squared value from MLR model

print('MLR results:')
print('a0 = ' + str(intercept_MLR))
print('a1 = ' + str(coef_MLR[0]))
print('a2 = ' + str(coef_MLR[1]))
print('a3 = ' + str(coef_MLR[2]))
print('a4 = ' + str(coef_MLR[3]))
```

> MLR results:
<br> a0 = 311.33892044632
<br> a1 = 1.0297744038862915
<br> a2 = -2.0570869420780595
<br> a3 = 2.4549509246271173
<br> a4 = -4.341338235550949
<br>

### Now: stepwise, and then MLR with kept predictors

``` python
result = stepwise_selection(X, Ynew,threshold_in=0.05,threshold_out=0.1)
print(result)

lm_step = linear_model.LinearRegression()
model_step = lm_step.fit(X[result],Ynew)
ypred_step = lm_step.predict(X[result]) #y predicted by MLR
intercept_step = lm_step.intercept_ #intercept predicted by MLR
coef_step = lm_step.coef_ #regression coefficients in MLR model
R2_step = lm_step.score(X[result],Ynew) #R-squared value from MLR model
```

    Add  X4                             with p-value 8.3401e-06
    Add  X2                             with p-value 0.0088294
    Add  X3                             with p-value 0.0195409
    resulting features:
    ['X4', 'X2', 'X3']




``` python
ax1 = plt.subplot(121)
ax1.scatter(Ynew,ypred_MLR)
l1 = np.min(ax1.get_xlim())
l2 = np.max(ax1.get_xlim())
ax1.plot([l1,l2], [l1,l2], ls="--", c=".3")
plt.xlabel('Ynew')
plt.ylabel('Y_MLR')
plt.title('MLR: R^2 = ' + str(R2_MLR)[:4])

ax2 = plt.subplot(122)
ax2.scatter(Ynew,ypred_step)
l1 = np.min(ax2.get_xlim())
l2 = np.max(ax2.get_xlim())
ax2.plot([l1,l2], [l1,l2], ls="--", c=".3")
plt.xlabel('Ynew')
plt.ylabel('Y_step')
plt.title('Stepwise: R^2 = ' + str(R2_step)[:4])

plt.tight_layout()
plt.show()
```

<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/mlr_5.png" title="MLR 5" class="img-fluid z-depth-1"%}
    </div>
</div>
<br>




option 2: perfroming MLR with all combinations of predictors on one part
of the data (calibration sample); use another part of the data
(validation sample) for finding the best model according to R2 (or
p-values). The best model = the one with smallest p-value over the
validation sample

calibration sample: first 25 observations validation sample: the rest

set arrey from 1 to the total number of predictors (i.e. 4)

### Goal: loop through every combination of normalized predictors, make linear model, and find one with best performance
``` python

R2_best = []
combo_best = []

for kk in range(1,5): #for each total number of predictors to use in model (from 1 predictor to 9 predictors)
    
    v0 = range(np.shape(X)[1])
    combinations = list(itertools.combinations(range(np.shape(X)[1]),kk)) #all possible combinations of kk total predictors 
    R2_test = []
    
    for ind in range(len(combinations)): #for each combination of predictors, make MLR model and compute R^2

        test_vars = np.array(combinations[ind])
        X_test = X.iloc[:25,test_vars]
        Y_test = Ynew.iloc[:25]
        
        X_valid = X.iloc[25:,test_vars]
        Y_valid = Ynew.iloc[25:]

        lm_test = linear_model.LinearRegression()
        model_test = lm_test.fit(X_test,Y_test)
        ypred_test = lm_test.predict(X_test) #y predicted by MLR
        R2_test.append(lm_test.score(X_valid,Y_valid)) #R-squared value from MLR model

    R2_best.append(np.max(R2_test))
    combo_best.append(combinations[np.argmax(R2_test)])
    
R2_best_final = np.max(R2_best)
combo_best_final = combo_best[np.argmax(R2_best)]

```


    The best combination of predictors is: 
    ['X1', 'X2', 'X4']

<br>

### Make linear model out of best predictors
``` python
X_calib_valid = X.iloc[:,np.asarray(combo_best_final)]
lm_calib_valid = linear_model.LinearRegression()
model_calib_valid = lm_calib_valid.fit(X_calib_valid,Ynew)
ypred_calib_valid = lm_calib_valid.predict(X_calib_valid) #y predicted by MLR
intercept_calib_valid = lm_calib_valid.intercept_ #intercept predicted by MLR
coef_calib_valid = lm_calib_valid.coef_ #regression coefficients in MLR model
R2_calib_valid = lm_calib_valid.score(X_calib_valid,Ynew) #R-squared value from MLR model
```
<br>

### Visualize calibration-validation model performance
``` python
ax = plt.subplot(111)
ax.scatter(Ynew,ypred_calib_valid)
l1 = np.min(ax.get_xlim())
l2 = np.max(ax.get_xlim())
ax.plot([l1,l2], [l1,l2], ls="--", c=".3")
plt.xlabel('Measured g_norm')
plt.ylabel('Modelled g_norm')
plt.title('Calibration-Validation Model Results: R^2 = ' + str(R2_calib_valid)[:4])
plt.show()
```
<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/mlr_6.png" title="MLR 6" class="img-fluid z-depth-1"%}
    </div>
</div>
<br>


## Example 3 - Again, do MLR and stepwise, but with different coefficients

### New coefficients, new Y
``` python
a0 = 0
a1 = 4
a2 = -3
a3 = 2
a4 = -1

Y = a0 + a1*X1 + a2*X2 + a3*X3 + a4*X4
Ynew = Y + 5*Yrand
```


### Now: MLR, stepwise, and then MLR with kept predictors
``` python
#MLR
lm_MLR = linear_model.LinearRegression()
model = lm_MLR.fit(X,Ynew)
ypred_MLR = lm_MLR.predict(X) #y predicted by MLR
intercept_MLR = lm_MLR.intercept_ #intercept predicted by MLR
coef_MLR = lm_MLR.coef_ #regression coefficients in MLR model
R2_MLR = lm_MLR.score(X,Ynew) #R-squared value from MLR model

lm_step = linear_model.LinearRegression()
model_step = lm_step.fit(X[result],Ynew)
ypred_step = lm_step.predict(X[result]) #y predicted by MLR
intercept_step = lm_step.intercept_ #intercept predicted by MLR
coef_step = lm_step.coef_ #regression coefficients in MLR model
R2_step = lm_step.score(X[result],Ynew) #R-squared value from MLR model
```


    MLR results:
    a0 = 311.33892044632
    a1 = 4.029774403886292
    a2 = -3.0570869420780578
    a3 = 1.4549509246271157
    a4 = -1.3413382355509498

    Stepwise results:
    Add  X1                             with p-value 0.000431949
    Add  X2                             with p-value 0.000336855
    Add  X3                             with p-value 0.0455512
    Resulting features:
    ['X1', 'X2', 'X3']


### Visualize
``` python


ax1 = plt.subplot(121)
ax1.scatter(Ynew,ypred_MLR)
l1 = np.min(ax1.get_xlim())
l2 = np.max(ax1.get_xlim())
ax1.plot([l1,l2], [l1,l2], ls="--", c=".3")
plt.xlabel('Ynew')
plt.ylabel('Y_MLR')
plt.title('MLR: R^2 = ' + str(R2_MLR)[:4])

ax2 = plt.subplot(122)
ax2.scatter(Ynew,ypred_step)
l1 = np.min(ax2.get_xlim())
l2 = np.max(ax2.get_xlim())
ax2.plot([l1,l2], [l1,l2], ls="--", c=".3")
plt.xlabel('Ynew')
plt.ylabel('Y_step')
plt.title('Stepwise: R^2 = ' + str(R2_step)[:4])

plt.tight_layout()
plt.show()
```

<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/mlr_7.png" title="MLR 7" class="img-fluid z-depth-1"%}
    </div>
</div>
<br>


## Example 4 - Scale up X4

``` python
X4new = 100 + 10*X4
X['X4'] = X4new

Y = a0 + a1*X1 + a2*X2 + a3*X3 + a4*X4new
Ynew = Y + 5*Yrand
```
<br>

### MLR, stepwise, and then MLR with kept predictors
``` python
#MLR
lm_MLR = linear_model.LinearRegression()
model = lm_MLR.fit(X,Ynew)
ypred_MLR = lm_MLR.predict(X) #y predicted by MLR
intercept_MLR = lm_MLR.intercept_ #intercept predicted by MLR
coef_MLR = lm_MLR.coef_ #regression coefficients in MLR model
R2_MLR = lm_MLR.score(X,Ynew) #R-squared value from MLR model


lm_step = linear_model.LinearRegression()
model_step = lm_step.fit(X[result],Ynew)
ypred_step = lm_step.predict(X[result]) #y predicted by MLR
intercept_step = lm_step.intercept_ #intercept predicted by MLR
coef_step = lm_step.coef_ #regression coefficients in MLR model
R2_step = lm_step.score(X[result],Ynew) #R-squared value from MLR model
```

    MLR results:
    a0 = 314.7523028018298
    a1 = 4.02977440388629
    a2 = -3.0570869420780595
    a3 = 1.4549509246271168
    a4 = -1.0341338235550952

    Stepwise results:
    Add  X4                             with p-value 1.66666e-11
    Add  X1                             with p-value 0.000236063
    Add  X2                             with p-value 0.000179307
    Resulting features:
    ['X4', 'X1', 'X2']


### Visualize
``` python
ax1 = plt.subplot(121)
ax1.scatter(Ynew,ypred_MLR)
l1 = np.min(ax1.get_xlim())
l2 = np.max(ax1.get_xlim())
ax1.plot([l1,l2], [l1,l2], ls="--", c=".3")
plt.xlabel('Ynew')
plt.ylabel('Y_MLR')
plt.title('MLR: R^2 = ' + str(R2_MLR)[:4])

ax2 = plt.subplot(122)
ax2.scatter(Ynew,ypred_step)
l1 = np.min(ax2.get_xlim())
l2 = np.max(ax2.get_xlim())
ax2.plot([l1,l2], [l1,l2], ls="--", c=".3")
plt.xlabel('Ynew')
plt.ylabel('Y_step')
plt.title('Stepwise: R^2 = ' + str(R2_step)[:4])

plt.tight_layout()
plt.show()
```

<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/mlr_8.png" title="MLR 8" class="img-fluid z-depth-1"%}
    </div>
</div>
<br>

### Standardize X and redo the analysis
``` python
Xnorm = (X - X.mean())/X.std()
plt.plot(Xnorm)
plt.show()
```
<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/mlr_9.png" title="MLR 9" class="img-fluid z-depth-1"%}
    </div>
</div>
<br>
