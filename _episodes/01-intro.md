---
title: "Getting and Reading Data for Today's Class"
teaching: 20
exercises: 5
questions:
- "How do I find and read in the data for today's class?"
- "What is an xarray.Dataset?"
objectives:
- "Discern DataArrays from Datasets in xarray"
keypoints:
- "A Dataset may contain one or more DataArrays"
---

### Getting Started

1. Launch a Jupyter notebook on Hopper.  As a reminder, its best to launch it from your home directory, then you can get to any other directory from within your notebook.
2. Be sure the environment is set to `clim_data`.
3. Create a new notebook and save it as `Subsetting.ipynb`.
4. Import the standard set of packages we use:

~~~
import xarray as xr
import numpy as np
import cartopy.crs as ccrs
import matplotlib.pyplot as plt
~~~
{: .language-python}

### Find our Dataset
 
Today we will work with monthly Sea Surface Temperature (SST) data. 
They are part of the [COLA Data Catalog](https://kpegion.github.io/COLA-DATASETS-CATALOG/),
but we have temporarily placed them on Hopper for use in class.

We will use the monthly global OISST version 2 dataset:
* It is 1deg x 1deg
* Latitudes go from 89.5N to 89.5S
* It goes from Dec 1981 to Apr 2020
* It is currently located in the `/home/pdirmeye/classes/clim680_2022/OISSTv2` directory

Now we will take a look at the data by opening a terminal session in JupyterLab (under the "File" tab) and looking in the directory where the data are located:

~~~
$ ls /home/pdirmeye/classes/clim680_2022/OISSTv2
~~~
{: .language-bash}

~~~
lmask  monthly  weekly
~~~
{: .output}

Since we are looking for monthly data, let's look in the monthly sub-directory.  Remember, you can use the `up-arrow` to avoid having to re-type:
~~~
$ ls /home/pdirmeye/classes/clim680_2022/OISSTv2/monthly
~~~
{: .language-bash}

~~~
sst.mnmean.nc
~~~
{: .output}

> ## Quick look at Metadata for our Dataset
>
> What command can you use to look at the metadata for our dataset?
>
> > ## Solution
> > ~~~
> > ncks -M /home/pdirmeye/classes/clim680_2022/OISSTv2/monthly/sst.mnmean.nc
> > ~~~
> > {: .language-bash}
> > 
> > But remember to load the NCO module first 
> > (if you have not already added it to your `.bashrc` file to load it automatically), 
> > so that you will have access to the commands in the NCO library:
> > 
> > ~~~
> > module load nco
> > ncks -M /home/pdirmeye/classes/clim680_2022/OISSTv2/monthly/sst.mnmean.nc
> > ~~~
> > {: .language-bash}
> {: .solution}
{: .challenge}

We can now use cut and paste to put the file and directory information into our notebook and read our dataset using `xarray`

~~~
file = '/home/pdirmeye/classes/clim680_2022/OISSTv2/monthly/sst.mnmean.nc'
ds = xr.open_dataset(file)
ds
~~~
{: .language-python}

When we run our cells, we get output that looks exactly like the results from `ncks -M`

It tells us that we have an `xarray.Dataset` and gives us all the metadata associated with our data. 

## What is an `xarray.Dataset`?

In climate data analysis, we typically work with <u>multidimensional</u> data. By multidimensional data (also often called N-dimensional), we mean data with many independent dimensions or axes. For example, we might represent Earth's surface temperature T as a three dimensional variable:

`T(x,y,t)`

where x is longitude, y is latitude, and t is time.

![N-dimensional Data Schematic](../fig/dataset-diagraml.png)

`Xarray` has two data structures:
* a `DataArray`, which holds a single multi-dimensional variable (or 'grid') and its coordinates
* a `Dataset`, which can hold multiple variables (DataArrays) that share some or all of the same coordinates

When we read in our data using `xr.open_dataset`, we read it in as an `xr.Dataset` object containing one or more DataArrays.  

A `DataArray` contains:
* values: a `numpy.ndarray` holdy the array's values (usually 1 or more dimensions)
* dims: dimension names for each axis (e.g. `lon`,`lat`,`lev`,`time`)
* coords: a container of arrays that label each point
* attrs: a container of arbitrary metadata

If we access an individual variable within an `xarray.Dataset` object, we have an `xarray.DataArray` object. Here's an example:

~~~
ds['sst']
~~~
{: .language-python}

you will also see this syntax used

~~~
ds.sst
~~~
{: .language-python}

The latter is a little less flexible - it is harder to specify DataArray names as variables, and may fail if there are special characters in the name. The `ds['sst']` construct is more robust. 


Compare the output for the `DataArray` and the `Dataset`

We can access individual attribues `attrs` of our `Dataset` using the following syntax:

~~~
units = ds['sst'].attrs['units']
print(units)
~~~
{: .language-python}

> ## Using `xarray.Dataset.attrs` to label figures
>
> Given the following lines of code, how would you use `attrs` to add units to
> the colorbar and a title to the map based on the `units` and `long_name` attributes?
>
> ~~~
> plt.contourf(ds['sst'][0,:,:])
> plt.title(FILLINLONGNAMEHERE)
> plt.colorbar(label=FILLINUNITSHERE) 
> ~~~
> {: .language-python}
{: .challenge}

The `xarray` package provides many convenient functions and tools for working with N-dimensional datasets. We will learn some of them today. 

{% include links.md %}

