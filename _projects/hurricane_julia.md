---
layout: page
title: Calculating rain rate from radar reflectivity of Hurricane Julia
description: Skills - python, hdf files, rasterio, ECMWF, weather forecasting, geoproj, numpy, pyproj, cloudsat
img: assets/img/hurricane_julia_reflectivity.png
importance: 3
category: research
---

# Introduction 
This research project was carried out to see radar reflectivity values in storms, and calculate the rain rate from the radar pulses. The raw data files are in the hdf format and obtained from [Cloudsat's Data Center](https://www.cloudsat.cira.colostate.edu/).  
<br>
Related projects: [Hurricane Karl](/_projects/hurricane_karl.md)
<br> 
This work finds the the point at which the ECMWF tempearture= 0 deg C for each radar pulse and overlays that on the reflectivity plot to check to see whether the bright band occurs at the freezing level. It will also plot the rain rate for the storm track along with the corresponding radar reflectivity to study the eye of the storm
<br><br>
### Storm Details <br>
**Name:** Hurricane Julia <br>
**Year:** 2010 <br>
**Region:** Atlantic Basin (did not make landfall)<br>
[Wiki Reference and Storm Details](https://en.wikipedia.org/wiki/Hurricane_Julia_(2010))

The full notebook with this work is at the [github project](https://github.com/Pearl-Ayem/Atmospheric-Radiation-Notebook-Data/blob/master/notebooks/cloudsat_precip_Julia-PA.ipynb) for this analysis. 

<br><br>

### 1. After reading the height and reflectivity raw data from the hdf files, I plot the rain rates:
{% raw %}
```python
rain_rate = HDFvd_read(r_file,'rain_rate',vgroup='Data Fields')
invalid = (rain_rate == -9999.)
rain_rate[invalid] = np.nan #creates runtime warning due to nan value
hit = rain_rate < 0.
rain_rate[hit] = np.abs(rain_rate[hit])
plt.plot(rain_rate)
```
{% endraw %}

<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/hurricane_julia_rain_rate.png" title="hurricane_julia_rain_rate" class="img-fluid z-depth-1"%}
    </div>
</div>

### 2. Creating a masked array of the reflectivity so that pcolormesh will plot it
{% raw %}
```python
hit=(radar_reflectivity == radar_missing)
radar_reflectivity=radar_reflectivity.astype(np.float)
radar_reflectivity[hit]=np.nan
zvals = radar_reflectivity/radar_scale
zvals=ma.masked_invalid(zvals)
```
{% endraw %}

### 3. Find the 3 minutes of the pass that includes the storm, then convert time to distance to get the greatcircle distance between shots

{% raw %}
```python
great_circle=pyproj.Geod(ellps='WGS84')
distance=[0]
start=(storm_lons[0],storm_lats[0])
for index in np.arange(1,len(storm_lons)):
    azi12,azi21,step= great_circle.inv(storm_lons[index-1],storm_lats[index-1],
                                       storm_lons[index],storm_lats[index])
    distance.append(distance[index-1] + step)
distance=np.array(distance)/meters2km
```
{% endraw %}

### 4. Make the plot for rain rate and reflectivity
{% raw %}
```python

%matplotlib inline

from matplotlib import cm
from matplotlib.colors import Normalize
from mpl_toolkits.axes_grid1 import make_axes_locatable
from matplotlib import gridspec

def plot_field2(distance,height,field,fig,cmap=None,norm=None):
    """
    draw a 2 panel plot with different panel sizes.  Put the radar reflectivity in the top panel 
    with a colorbar along the bottom, and pass the second axis back to be filled in later.  Uses 
    the sharex keyword to give both plots the same x axis (distance) and the gridspec class to 
    layout the grid
    """
    
    gs = gridspec.GridSpec(2, 1, height_ratios=[2, 1]) 
    ax1 = fig.add_subplot(gs[0])
    ax2 = fig.add_subplot(gs[1],sharex=ax1)
    if cmap is None:
        cmap=cm.inferno
    col=ax1.pcolormesh(distance,height,field,cmap=cmap, norm=the_norm)
    divider = make_axes_locatable(ax1)
    cax = divider.append_axes("bottom", size="5%", pad=0.55)
    ax1.figure.colorbar(col,extend='both',cax=cax,orientation='horizontal')
    return ax1, ax2

vmin=-30
vmax=20
the_norm=Normalize(vmin=vmin,vmax=vmax,clip=False)
cmap_ref=cm.plasma
cmap_ref.set_over('w')
cmap_ref.set_under('b',alpha=0.2)
cmap_ref.set_bad('0.75') #75% grey

cloud_height_km=radar_height[0,:]/meters2km
fig = plt.figure(figsize=(15, 8)) 
ax1, ax2 = plot_field2(distance,cloud_height_km,storm_zvals.T,fig,cmap=cmap_ref,norm=the_norm)
ax1.set(ylim=[0,17],xlim=(0,1200))
ax1.set(xlabel='distance (km)',ylabel='height (km)',title='equivalent radar reflectivity in dbZe')

ax2.plot(distance,rain_rate)
ax2.set(xlabel='distance (km)',ylabel='rain rate (mm/hour)')
```
{% endraw %}

<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/hurricane_julia_reflectivity.png" title="hurricane_julia_reflectivity" class="img-fluid z-depth-1"%}
    </div>
</div>


### 5. Get ECMWF temps
{% raw %}
```python
z_file= list(a301.data_dir.glob('*ECMWF-AUX_GRANULE*hdf'))[2]
ec_height=HDFvd_read(z_file,'EC_height')
ec_temps, temps_attributes = HDFsd_read(z_file,'Temperature')
ec_missing = temps_attributes['missing']
```
{% endraw %}
### 6. Subsetting the ECMWF data
{% raw %}
```python
bad_temps = (ec_temps == ec_missing)
ec_temps[bad_temps]=np.nan
ec_temps=np.ma.masked_invalid(ec_temps)
ec_temps = ec_temps - 273.15
ec_temps=ec_temps[time_hit,:]
```
{% endraw %}
### 7. ECMWF temperatures for the segment
{% raw %}
```python
def plot_field(distance,height,field,ax,cmap=None,norm=None):
    """
    given an axis, draw a cloudsat cross section
    """
    if cmap is None:
        cmap=cm.inferno
    col=ax.pcolormesh(distance,height,field,cmap=cmap,
                  norm=the_norm)
    ax.figure.colorbar(col,extend='both',ax=ax)
    return ax

fig, ax =plt.subplots(1,1,figsize=(15,4))
vmin=-30
vmax=30
the_norm=Normalize(vmin=vmin,vmax=vmax,clip=False)
cmap_ec= cm.bwr
cmap_ec.set_over('w')
cmap_ec.set_under('b',alpha=0.2)
cmap_ec.set_bad('0.75') #75% grey

ec_height_km=ec_height/meters2km
ax=plot_field(distance,ec_height_km,ec_temps.T,ax,cmap=cmap_ec,
                  norm=the_norm)
ax.set(ylim=[0,10],xlim=(0,1200))
ax.set(xlabel='distance (km)',ylabel='height (km)',title='ECMWF temps in deg C')
```
{% endraw %}

<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/hurricane_julia_ECMWF_temps.png" title="hurricane_julia_ECMWF_temps" class="img-fluid z-depth-1"%}
    </div>
</div>

### 8. Read in heating rate
{% raw %}
```python

hr_file= list(a301.data_dir.glob('*FLXHR*hdf'))[2]
lats,lons,date_times,prof_times,dem_elevation=get_geo(hr_file)
lats=lats.squeeze()
lons=lons.squeeze()
qr, qr_attrs = HDFsd_read(hr_file,'QR')
qr_height, height_attrs = HDFsd_read(hr_file,'Height')
factor = HDFvd_read(hr_file,'QR.factor',vgroup='Swath Attributes')[0][0]
missing = HDFvd_read(hr_file,'QR.missing',vgroup='Swath Attributes')[0][0]
units = HDFvd_read(hr_file,'QR.units',vgroup='Swath Attributes')[0][0]
#set_trace()
hit = (qr == missing)
qr = qr.astype(np.float64)/factor
qr[hit]=np.nan


storm_qr=qr[:,time_hit,:]
storm_height=qr_height[time_hit,:]
```
{% endraw %}

### 9. Split out the long and shortwave heating rates
{% raw %}
```python
shortwave_qr=storm_qr[0,:,:]
longwave_qr=storm_qr[1,:,:]
```
{% endraw %}

### 10. Make a plots for shortwave and longwave heating rates which look like so:

<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/hurricane_julia_lw_sw.png" title="hurricane_julia_lw_sw" class="img-fluid z-depth-1"%}
    </div>
</div>


<br><br><br>
## More about Cloudsat

A detailed overview of the Cloudsat mission is given in this [Bulletin of the AMS article](http://cloudsat.atmos.colostate.edu/BAMS_CloudSat_CR.pdf) article on cloudsat.  

From the [Wikipedia Cloudsat page](https://en.wikipedia.org/wiki/CloudSat) :

> Cloudsat is a NASA Earth observation satellite, which was launched on a Delta II rocket on April 28, 2006. It uses radar to measure the altitude and properties of clouds, adding to information on the relationship between clouds and climate in order to help resolve questions about global warming.
 
From the [Cloudsat Project Page](http://cloudsat.atmos.colostate.edu/) at the Dept. of Atmospheric Science, Colorado State U:

> CloudSat flies in formation in the A Train, with several other satellites: Aqua, Aura, CALIPSO and the French PARASOL. CloudSat was selected as a NASA Earth System Science Pathfinder satellite mission in 1999 to provide observations necessary to advance our understanding of cloud abundance, distribution, structure, and radiative properties. Since 2006, CloudSat has flown the first satellite-based millimeter-wavelength cloud radar—a radar that is more than 1000 times more sensitive than existing weather radars. 


> Unlike ground-based weather radars that use centimeter wavelengths to detect raindrop-sized particles, CloudSat’s radar allows us to detect the much smaller particles of liquid water and ice that constitute the large cloud masses that make our weather.


>CloudSat was co-manifested with the CALIPSO (Cloud-Aerosol Lidar and Infrared Pathfinder Satellite Observations) satellite aboard a Delta II rocket for its launch on 28 April 2006. The satellites fly in a nearly circular orbit with an equatorial altitude of approximately 705 km. The orbit is sun-synchronous, maintaining a roughly fixed angle between the orbital plane and the mean solar meridian. CloudSat maintains a close formation with Aqua and a particularly close formation with CALIPSO, providing near-simultaneous and collocated observations with the instruments on these two platforms.
 



