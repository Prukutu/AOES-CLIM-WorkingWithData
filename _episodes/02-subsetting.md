---
title: "Subsetting Data"
teaching: 0
exercises: 0
questions:
- "How do I extract a particular region from my data?"
- "How do I extract a specific time slice from my data?"
objectives:
- ""
keypoints:
- ""
---

### Getting Started

1. Launch a Jupyter notebook on a COLA server.  As a reminder, its best to launch it from your home directory, then you can get to any other directory from within your notebook.

2. Create a new notebook and save it as Subsetting.

3. Import the standard set of packages we use:

~~~
import xarray as xr
import numpy as np
import cartopy.crs as ccrs
import matplotlib.pyplot as plt
~~~
{: .language-python}

### Find our Dataset
 
Today we will work with datasets that are on the COLA servers and findable using the COLA Data Catalog.
We will start by using monthly Sea Surface Temperature (SST) data.

Let's go to the [COLA Data Catalog](https://kpegion.github.io/COLA-DATASETS-CATALOG/)
Browse the main catalog and follow the links to `obs->gridded->ocn->sst->oisstv2_monthly`

Let's take a look at our dataset and what we can learn about it from the catalog:
* It is 1deg x 1deg
* It goes from Dec 1981 to Apr 2020
* It was last updated on the COLA Servers on Jun 25, 2020
* It is located in the '/shared/obs/gridded/OISSTv2` directory

Now we will take a look at the data on COLA by opening a terminal in our Jupyter Notebook and looking in the directory wehere the data are located:

~~~
$ ls /shared/obs/gridded/OISSTv2
~~~
{: .language-bash}

~~~
lmask  monthly  weekly
~~~
{: .output}

Since we are looking for monthly data, let's look in the monthly sub-directory.  Remember, you can use the `up-arrow` to avoid having to re-type:
~~~
$ ls /shared/obs/gridded/OISSTv2/monthly
~~~
{: .language-bash}

~~~
sst.mnmean.nc
~~~
{: .output}

> ## Quick look at Metadata for our Dataset
>
> What command can you use to look at the metadata for our dataset and confirm that it
> matches the COLA Data Catalog?
>
> > ## Solution
> > ~~~
> > ncdump -h /shared/obs/gridded/OISSTv2/monthly/sst.mnmean.nc
> > ~~~
> > {: .language-bash}
> {: .solution}
{: .challenge}

We can now use cut and paste to put the file and directory information into our notebook and read our dataset using `xarray`

~~~
file='/shared/obs/gridded/OISSTv2/monthly/sst.mnmean.nc'
ds=xr.open_dataset(file)
ds
~~~
{: .language-python}

When we run our cells, we get output that looks exactly like the COLA Data Catalog and the results from `ncdump -h`
It tells us that we have an `xarray.Dataset` and gives us all the metadata associated with our data. 

## What is an `xarray.Dataset`?



I always make it a habit to make a quick contour plot of my data to make sure it looks reasonble.  Most of the timewhen I have to spend time to debug an analysis issue, it happened very early on in my code.  Its best to make sure my data are correct before I start doing any analysis.

~~~
plt.contourf(ds['sst'][0,:,:]
~~~
{: .language-python}

Note: `ds['sst'] and `ds.sst` are two ways of accessing 

### Subsetting data in Space

Often in our research, we want to look at a specific region defined by a set of latitudes and longitudes. 





{% include links.md %}

