---
title: "Masking"
teaching: 15
exercises: 0
questions:
- "How to I maskout land or ocean?"
objectives:
- "Understand how to apply masks to limit the domain where calcuations or plotting occurs."
keypoints:
- "Masking is another powerful tool for working with data."
- "`pcolormesh` plots data as a grid of color shades corresponding to the data's grid"
---

Sometimes we want to maskout data based on a land/sea mask that is provided. 

## First Steps

Create a new markdown cell for this episode:

~~~
## Masking
~~~
{: .language-python}

Read in these two datasets:

~~~
obs_file='/shared/obs/gridded/OISSTv2/monthly/sst.mnmean.nc'
ds_obs=xr.open_dataset(obs_file)
ds_obs
~~~
{: .language-python}

This is the original data file we read in.

~~~
mask_file='/shared/obs/gridded/OISSTv2/lmask/lsmask.nc'
ds_mask=xr.open_dataset(mask_file)
ds_mask
~~~
{: .language-python}

Let's look at this second file.  It contains a lat-lon grid that is the same as our SST data and a single time.  The data are 0's and 1's.  Plot it.

~~~
plt.contourf(ds_mask['mask'])
~~~
{: .language-python}

What does this error mean? 
The data has a time dimension with a single element and coordinate so it appears as 3-D to the plotting function.  
We can drop any single dimensions like this using the `squeeze` method:

~~~
ds_mask=ds_mask.squeeze()
plt.pcolormesh(ds_mask['mask'])
~~~
{: .language-python}

We have used a different plotting function.
`countourf` plotted a discrete set of colors at contour intervals along smooth curves.
`pcolormesh` plots filled grid cells matching the resolution of the data, but using a more continuous color scale.

The latitudes are upside down.  Let's fix that like we did before. In this case, we will
fix it for both the mask and the data.

~~~
ds_mask=ds_mask.reindex(lat=list(reversed(ds_mask['lat'])))
ds_data=ds_data.reindex(lat=list(reversed(ds_data['lat'])))
plt.pcolormesh(ds_mask['mask'])
plt.colorbar()
~~~
{: .language-python}

That's better! We can see that the mask data are 1 for ocean and 0 for land. 
We can use the `where` method in `xarray` to mask the data to only over the ocean.

~~~
da_ocean=ds_data['sst'].mean('time').where(ds_mask['mask']==1)
da_ocean
plt.contourf(da_ocean,cmap='coolwarm')
plt.colorbar()
~~~
{: .language-python}

The `xarray` [documentation](http://xarray.pydata.org/en/stable/api.html) is also worth bookmarking. 
Keep in mind that in Python there is usually a very simple and straightforward function or library that will do anything you want to do.
Often the trick is discovering the right function!
Web searches can be very helpful. 

In particular, most questions you might have are already answered by someone on [Stack Overflow](https://stackoverflow.com), which is an open forum for programming questions and answers. 
However, the quality of the answers varies - always try to understand any appropriate-looking answer before relying on it. 
Use Stack Overflow and other online rexources as learning tools, not a crutch!
