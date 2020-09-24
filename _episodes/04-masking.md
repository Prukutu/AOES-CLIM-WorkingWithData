---
title: "Masking"
teaching: 0
exercises: 0
questions:
- "How to I maskout land or ocean?"
objectives:
- ""
keypoints:
- ""
---

Sometimes we want to maskout data based on a land/sea mask that is provided. 

## First Steps

Create a new notebook and save it as Masking.

Import the standard set of packages we use:

~~~
import xarray as xr
import numpy as np
import cartopy.crs as ccrs
import matplotlib.pyplot as plt
~~~
{: .language-python}

Read in these two datasets:

~~~
data_file='/shared/obs/gridded/OISSTv2/monthly/sst.mnmean.nc'
ds_data=xr.open_dataset(data_file)
ds_data
~~~
{: .language-python}

This is the original data file we read in.

~~~
mask_file='/shared/obs/gridded/OISSTv2/lmask/lsmask.nc'
ds_mask=xr.open_dataset(mask_file)
ds_mask
~~~
{: .language-python}

Let's look at this file.  It contains a lat and lon grid that is the same as our SST data and a single time.
The data are 0's and 1's.  Plot it.

~~~
plt.contourf(ds_mask['mask'])
~~~
{: .language-python}

What does this error mean? 
The data has a single time dimension so it appears as 3-D to `contourf`.  We can drop single dimensions using `squeeze`

~~~
ds_mask=ds_mask.squeeze()
plt.contourf(ds_mask['mask'])
~~~
{: .language-python}

The latitudes are upside down.  Let's fix that like we did before. In this case, we will
fix it for the mask and the data.

~~~
ds_mask=ds_mask.reindex(lat=list(reversed(ds_mask['lat'])))
ds_data=ds_data.reindex(lat=list(reversed(ds_data['lat'])))
plt.contourf(ds_mask['mask'])
plt.colorbar()
~~~
{: .language-python}

That's better! We can see that the mask data are 1 for ocean and 0 for land. We can use `xarray.where` to mask the data to only over the ocean.

~~~
{: .language-python}
da_ocean=ds_data['sst'].mean('time').where(ds_mask['mask']==1)
da_ocean
plt.contourf(da_ocean,cmap='coolwarm')
plt.colorbar()
~~~
{: .language-python}
