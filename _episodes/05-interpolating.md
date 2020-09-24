---
title: "Interpolating"
teaching: 0
exercises: 0
questions:
- "How do I interpolate data to a different grid"
objectives:
- ""
keypoints:
- ""
---

It is common in climate data analysis to need to interpolate data to a different grid.  One example is that I have model data on a model grid and observed data on a different grid. I want to calculate a difference between model and obs.  We will make a difference between the average SST in a CMIP5 model historical simulation and the average observed SSTs from OISSTv2 dataset we have been working with. 


## Regular Grid

If you are using a regular grid, interpolation is easy using `xarray`.  
A regular grid is one with a 1-dimensional latitude and 1-dimensional longitude. This means that the latitudes are the same for all longitudes and the longitudes are the same for all latitudes.

Most data you encounter will be on a regular grid, but some are not.  We will discuss irregular grids in another class.


## First Steps

Create a new notebook and save it as Interpolating.

Import the standard set of packages we use:

~~~
import xarray as xr
import numpy as np
import cartopy.crs as ccrs
import matplotlib.pyplot as plt
~~~
{: .language-python}

Read in obs and mask data:

~~~
obs_file='/shared/obs/gridded/OISSTv2/monthly/sst.mnmean.nc'
ds_obs=xr.open_dataset(data_file)
ds_obs
~~~
{: .language-python}

~~~
mask_file='mask_file='/shared/obs/gridded/OISSTv2/lmask/lsmask.nc''
ds_mask=xr.open_dataset(mask_file)
ds_mask
~~~
{: .language-python}

This is the original data file we read in and land/sea mask we previously used.

Remember, we want to reverse the lats

~~~
ds_mask=ds_mask.reindex(lat=list(reversed(ds_mask['lat'])))
ds_obs=ds_obs.reindex(lat=list(reversed(ds_obs['lat'])))
plt.contourf(ds_obs['sst'][0,:,:])
~~~

Read in the model data

~~~
model_path='/shared/cmip5/data/historical/atmos/mon/Amon/ts/NCAR.CCSM4/r1i1p1/'
model_file='ts_Amon_CCSM4_historical_r1i1p1_185001-200512.nc'
ds_model=xr.open_dataset(model_file)
ds_model
~~~
{: .language-python}

This data has lats from S to N, so we don't need to reverse it. Take a look at our model data.
This data appears to have different units than the obs data. We can change this.

~~~
ds_model['ts']=ds_model['ts']-273.15
~~~
{: .language-python}

Remember that `xarray` keeps our metadata with our data, so we need to also change the metadata to keep that information correct in our `xarray.Dataset`

~~~
ds_model['ts'].attrs['units']=ds_obs['sst'].attrs['units']
~~~
{: .language-python}

How would you take the time mean of each dataset?

~~~
ds_model_mean=ds_model.mean(dim='time')
ds_obs_mean=ds_obs.mean(dim='time')
ds_model_mean
~~~
{: .language-python}

We are now going to use the `interp_like` function.  The [documentation] (https://xarray.pydata.org/en/stable/generated/xarray.DataArray.interp_like.html) tells us that this fucntion interpolate an object onto the coordinates of another object, filling out of range values with NaN.

One important thing to know that is not clear in the documentation is that the coordinates and variable name to interpolate must all be the same.

Both dataset use lat and lon, but the model dataset uses `ts` and the obs dataset uses `sst`.  Let's change the name of the model variable to `sst`.

~~~
ds_model_mean=ds_model_mean.rename({'ts':'sst'})
~~~
{: .language-python}

We will interpolate the model to be like the obs. 

~~~
model_interp=ds_model_mean.interp_like(ds_obs_mean)
model_interp
~~~
{: .language-python}

Let's take a look and compare with our data before interpolation.

~~~
plt.contourf(model_interp['sst'],cmap='coolwarm')
plt.title('Interpolated')
plt.colorbar()
~~~
{: .language-python}

~~~
plt.contourf(ds_model_mean['sst'],cmap='coolwarm')
plt.title('Original')
plt.colorbar()
~~~
{: .language-python}

Now let's see how different our model mean is from the obs mean. Remember to apply our land/ocean mask.

~~~
diff=(model_interp-ds_obs_mean).where(ds_mask['mask'].squeeze()==1)
diff
~~~
{: .language-python}

And plot it...

~~~
{: .language-python}
plt.title('Model - OBS')
plt.contourf(diff['sst'],clevs,
             cmap='coolwarm')
plt.colorbar()
~~~
{: .language-python}

