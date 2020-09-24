---
title: "Aggregating Data"
teaching: 0
exercises: 0
questions:
- "How to I aggregate data over various dimensions?"
objectives:
- ""
keypoints:
- ""
---

The Oceanic Nino Index [ONI](https://origin.cpc.ncep.noaa.gov/products/analysis_monitoring/ensostuff/ONI_v5.php) is used by the Climate Prediction Center to monitor and predict El Nino and La Nina. It is defined as the 3-month running mean of SST anomalies in the Nino3.4 region. We will use aggregation methods from `xarray` to calculate this index. 

## First Steps

Create a new notebook and save it as Aggregating.ipynb

Import the standard set of packages we use:

~~~
import xarray as xr
import numpy as np
import cartopy.crs as ccrs
import matplotlib.pyplot as plt
~~~
{: .language-python}

Read in the dataset we wrote in the last notebook.

~~~
file='/scratch/kpegion/nino34_1982-2019.oisstv2.nc'
ds=xr.open_dataset(file)
ds
~~~
{: .language-python}

## Take the mean over a lat-lon region

~~~
ds_nino34_index=ds.mean(dim=['lat','lon'])
ds_nino34_index
~~~
{: .language-python}

Our data now has only a time dimension. Make a plot.

~~~
plt.plot(ds_nino34_index['time'],ds_nino34_index['sst'])
~~~
{: .language-python}

## Calculate Anomalies

Anomaly means departure from normal (called climatology).  We often calculate anomalies for working with climate data.  We will spend time in a future class learning about calculating climatology and anomalies using another feature of `xarray` called `groupby`.  Today, I will show you the steps with little explanation.

~~~
ds_climo=ds_nino34_index.groupby('time.month').mean()
ds_anoms=ds_nino34_index.groupby('time.month')-ds_climo
ds_anoms
~~~
{: .language-python}

Plot our data.

~~~
plt.plot(ds_anoms['time'],ds_anoms['sst'])
~~~
{: .language-python}

> ## Why do I constantly print and plot the data?
>
> Printing the data so I can see its dimensions after each
> step provides a check on whether my code did what I 
> intended it to do.
>
> Plotting also gives me a quick look to make sure 
> everything makes sense.
>
> I encourage you to do the same when developing and 
> testing new code!
>
>
{: .callout}

## Rolling (Running Means)

The ONI is calculated using a 3-month running mean.  This can be done using the `rolling` function.

> ## Reading and Learning from Documentation
>
> Read the [documentation](http://xarray.pydata.org/en/stable/generated/xarray.DataArray.rolling.html) for the `xarray.rolling` function.
> Following their example, make a 3-month running mean of the `ds_anoms` data.
>
>
>> ## Solution
>>> ~~~
>>> ds_3m=ds_anoms.rolling(time=3,center=True).mean().dropna(dim='time') 
>>> ds_3m
>>> ~~~
>> {: .language-python}
> {: .solution}
{: .challenge}

Let's plot our original and 3-month running mean data together

~~~
plt.plot(ds_anoms['sst'],color='r')
plt.plot(ds_3m['sst'],color='b')
plt.legend(['orig','smooth'])
~~~
{: .language-python}


> ## Some other aggregation functions
>
> There are a number of other aggregate functions such as: `std`,`min`,`max`,`sum`, among others.
>
> Using the original dataset in this notebook `ds`, find and plot the maximum SSTs for each
> gridpoint  over the time dimension.
>
>> ## Solution
>>> ~~~
>>> ds_max=ds.max(dim='time')
>>> plt.contourf(ds_max['sst'],cmap='Reds')
>>> plt.colorbar()
>>> ~~~
>> {: .language-python}
> {: .solution}
>
> Using the original dataset in this notebook `ds`, calculate and plot the standard deviation at each
> gridpoint.
>
>> ## Solution
>>> ~~~
>>> ds_std=ds.std(dim='time')
>>> plt.contourf(ds_std['sst'],cmap='RdBu_r')
>>> plt.colorbar()
>>> ~~~
>> {: .language-python}
> {: .solution}
>
{: .challenge}

