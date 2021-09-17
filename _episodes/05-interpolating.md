---
title: "Interpolating"
teaching: 25
exercises: 5
questions:
- "How do I interpolate data to a different grid"
objectives:
- "Lean how to compare and inspect datasets."
keypoints:
- "Interpolating different data sets to a common grid is necessary to perform quantitative comparisons or to combine data"
- "Don't automatically accept your result. Scrutinize, try to understand the features you see. Do they mean what you think they should mean?
---

It is common in climate data analysis to need to interpolate data to a different grid.  One example is that I have model data on a model grid and observed data on a different grid. I want to calculate a difference between model and obs. 

In this lesson, We will make a difference between the average SST in a CMIP5 model historical simulation and the average observed SSTs from the OISSTv2 dataset we have been working with. 


## Regular Grid

If you are using a regular grid, interpolation is easy using `xarray`.  

A regular grid is one with a 1-dimensional latitude and 1-dimensional longitude. This means that the latitudes are the same for all longitudes and the longitudes are the same for all latitudes. This does not mean the spacing of either grid needs to be uniform! Global climate model output, in particular, is often non-uniform in latitude. This is not a problem for `xarray`.

Most of the data you encounter will be on a regular grid, but some are not.  We will discuss irregular grids later in the course.

In a new markdown cell, create a new header for this episode:

~~~
## Interpolating
~~~
{: .language-python}


Read in some model data

~~~
model_path='/shared/cmip5/data/historical/atmos/mon/Amon/ts/CCCma.CanCM4/r1i1p1/'
model_file='ts_Amon_CanCM4_historical_r1i1p1_196101-200512.nc'
ds_model=xr.open_dataset(model_path+model_file)
ds_model
~~~
{: .language-python}

This data has latitudes from S to N, so we don't need to reverse it.
Take a look at our model data.  
It is much lower resolution that the observations we have been using - there are only 64 points in latitude, 128 in longitude.

Also, this data appears to have different units than the obs data. 
We can change this.

~~~
ds_model['ts']=ds_model['ts']-273.15
ds_model['ts'].attrs['units']=ds_obs['sst'].attrs['units']
~~~
{: .language-python}

Remember that `xarray` keeps our metadata with our data, so we also had to change the `units` metadata to keep that information correct in our `xarray.Dataset`

How would you take the time mean of each dataset?

~~~
ds_obs_mean=ds_obs.mean(dim='time')
ds_model_mean=ds_model.mean(dim='time')
ds_model_mean
~~~
{: .language-python}

We are now going to use the `interp_like` function.  The [documentation](http://xarray.pydata.org/en/stable/generated/xarray.Dataset.interp_like.html) tells us that this fucntion interpolates an object onto the coordinates of another object, filling out of range values with `NaN`.

One important thing to know that is not clear in the documentation is that the coordinates and variable name to interpolate must all be the same.

All variables in the `Dataset` that have the same name as the one we are interpolating to will be interpolated.

Both of our datasets use lat and lon, but the model dataset uses `ts` (because it is actually _surface temperature_ over all surfaces) and the obs dataset uses `sst`.  
Let's change the name of the model variable to `sst`.

~~~
ds_model_mean=ds_model_mean.rename({'ts':'sst'})
~~~
{: .language-python}

We will interpolate the model to match the obs. 

~~~
model_interp=ds_model_mean.interp_like(ds_obs_mean)
model_interp
~~~
{: .language-python}

Let's take a look and compare with our data before interpolation. 
We will make use of `cartopy` again to make more pleasing figures.

~~~
fig = plt.figure(figsize=(11,8.5))

ax=plt.axes(projection=ccrs.PlateCarree(central_longitude=180.0))

cs = ax.pcolormesh(model_interp['lon'],model_interp['lat'],
                   model_interp['sst'],cmap='coolwarm',transform = ccrs.PlateCarree())
ax.coastlines()
plt.title('Interpolated')
~~~
{: .language-python}

~~~
fig = plt.figure(figsize=(11,8.5))

ax=plt.axes(projection=ccrs.PlateCarree(central_longitude=180.0))

cs = ax.pcolormesh(ds_model_mean['lon'],ds_model_mean['lat'],
                   ds_model_mean['sst'],cmap='coolwarm',transform = ccrs.PlateCarree())
ax.coastlines()
plt.title('Original')
~~~
{: .language-python}

You can see the coarse _pixelation_ clearly in the original grid.


Now let's see how different our model mean is from the obs mean. 
Remember to apply our land/ocean mask this time as we are only concerned with SST.

~~~
diff=(model_interp-ds_obs_mean).where(ds_mask['mask']==1)
diff
~~~
{: .language-python}

And plot it...

~~~
fig = plt.figure(figsize=(11,7))

plt.pcolormesh(diff['sst'], cmap='summer')
plt.colorbar()
plt.title('Model - OBS')
~~~
{: .language-python}

> ## A critical eye toward your results
>
> Does this difference map look OK? A little stange?  Maybe even alarming?
>
> There are very large differences at high latitudes - the model is as much as 30˚C colder than observations.
> What is going on?
>
>> ## Solution
>> Recall that the model variable is not actually SST but surface temperature.
>> The green areas are where sea ice is present. In the model output, this is the mean temperature of the top of the sea ice.
>> 
>> Look back at the map of the masked observational SST you produced in the **Masking** section of your notebook.
>> Ocean water can get below 0˚C due to its salt content, but only a couple degrees below.
>> In the observational dataset, it is actual ocean temperature, below the ice, that is reported.
>> 
>> Thus we are not comparing the same quantities at high latitudes. 
>> Where there is no sea ice, this is a valid comparison.
>> Always be careful to consider what a result means, and why it looks the way it does!
> {: .solution}
{: .challenge}
