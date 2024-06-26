---
title: "Accessing data at the THREDDS Data Server"
author: Patrick Wright
date: 2023-03-10 00:00
classes: wide
categories:
  - Guides
tags: 
  - thredds
  - aws
  - promice
  - cryoclim
---

The GEUS Glaciology and Climate THREDDS data server (TDS) is located here:

[https://thredds.geus.dk](https://thredds.geus.dk)

This guide should be updated with any additional data access methods that are useful!

## Browser GUI

Using the graphic user interface (GUI) on the browser, you can navigate to individual files in the TDS directory. Both csv and [NetCDF](https://www.unidata.ucar.edu/software/netcdf/) files can be accessed as a simple "click and download" via the HTTP file download method.

NetCDF can also be accessed using [OPeNDAP](https://www.opendap.org/) (which has it's own browser GUI page for each station) and other data access methods listed for individual files. For example, see the data access services listed for [KAN_U](https://thredds.geus.dk/thredds/catalog/aws_l3_station_netcdf/level_3/KAN_U/catalog.html?dataset=aws_l3_station_netcdf/level_3/KAN_U/KAN_U_hour.nc)). The services landing page is useful to scrape URLs for use in programmatic access methods. See [here](https://www.ncei.noaa.gov/access/thredds-user-guide) for a concise explanation of the available TDS data access services.

## Command-line

Retrieve single file with `wget`:
```
wget 'https://thredds.geus.dk/thredds/fileServer/aws_l3_time_netcdf/level_3/hour/CEN1_hour.nc'
```

Download an entire catalog with `wget`. In this case, retrieve everything in the `aws_l3_time_netcdf/level_3` directory (includes `day`, `hour` and `month` subdirectories). Note that we also have to exclude (`-X`) unwanted directories, such as `MODIS_*` and `SICE_*`:
```
wget -e robots=off -nH --cut-dirs 4 -nc -r -l5 -A '*.nc' -R 'catalog*' -I /thredds/fileServer/,/thredds/catalog/ 'https://thredds.geus.dk/thredds/catalog/aws_l3_time_netcdf/level_3/catalog.html' -X thredds/catalog/MODIS_* -X thredds/catalog/SICE_*
```

## Using python, pandas (csv)

Read single station (csv) with `pandas`:
```
import pandas as pd

stid = 'NUK_Uv3'

# Use either the "_time" or "_station" directories:
url = "https://thredds.geus.dk/thredds/fileServer/aws_l3_time_csv/level_3/hour/{}_hour.csv".format(stid)
#url = "https://thredds.geus.dk/thredds/fileServer/aws_l3_station_csv/level_3/{}/{}_hour.csv".format(stid,stid)

data = pd.read_csv(url)

# set dataframe index to datetime:
data.set_index(pd.to_datetime(data.time), inplace=True)
data.drop(['time'], axis=1, inplace=True) # drop original time column
```

Read all stations (csv) with `pandas`, make dictionary of dataframes:
```
import pandas as pd

# Read AWS_station_locations.csv to get station names
url_locations = "https://thredds.geus.dk/thredds/fileServer/metadata/AWS_station_locations.csv"
locations = pd.read_csv(url_locations)

data = {}
for stid in locations.stid: # this loop takes ~30 sec
  stn_url = "https://thredds.geus.dk/thredds/fileServer/aws_l3_time_csv/level_3/hour/{}_hour.csv".format(stid)
  df = pd.read_csv(stn_url)
  df.set_index(pd.to_datetime(df.time), inplace=True)
  df.drop(['time'], axis=1, inplace=True) # drop original time column
  data[stid] = df
```

## Using python, xarray, Pydap and OPeNDAP (NetCDF)

The simplest method to open a NetCDF file with `xarray` and the OPeNDAP service is:
```
import xarray as xr

stid = 'NUK_Uv3'

# Use either the "_time" or "_station" directories.
# These "/dodsC" URLs are listed in the "Data URL" field if you click on the station's "OPENDAP" link.

url = "https://thredds.geus.dk/thredds/dodsC/aws_l3_time_netcdf/level_3/hour/{}_hour.nc".format(stid)
#url = "https://thredds.geus.dk/thredds/dodsC/aws_l3_station_netcdf/level_3/{}/{}_hour.nc".format(stid,stid)

ds = xr.open_dataset(url)
```

To make time "slices" use `.sel` or `.isel` methods. For spatial datasets, these methods can also be used to take spatial slices. See the xarray docs for [indexing and selecting data](https://docs.xarray.dev/en/stable/user-guide/indexing.html#indexing-and-selecting-data).

The following will return the single latest obset. Note that for promice data, the latest hour will only return instantaneous vars (`_i`). Averages are assigned to the start of the hour, so `_u` vars are in the previous hour. For more time-slicing examples, see below.
```
ds = xr.open_dataset(url).isel(time=-1)
```

This is equivalent to `xr.open_dataset(url, engine='netcdf4')` (see [docs](https://docs.xarray.dev/en/stable/generated/xarray.open_dataset.html) for `xr.open_dataset`).

**If you encounter errors with the above method**, the following is an alternative. This uses [`xr.backends.PydapDataStore`](https://docs.xarray.dev/en/stable/generated/xarray.backends.PydapDataStore.html) following methods described [here](https://help.marine.copernicus.eu/en/articles/5182598-how-to-consume-the-opendap-api-and-cas-sso-using-python). Using `conda`, install `pydap >= 3.3.0` with:
```
$ conda install -c conda-forge pydap
```

Load entire file for single station:
```
import xarray as xr
from pydap.client import open_url

stid = 'NUK_Uv3'

url = "https://thredds.geus.dk/thredds/dodsC/aws_l3_time_netcdf/level_3/hour/{}_hour.nc".format(stid)
#url = "https://thredds.geus.dk/thredds/dodsC/aws_l3_station_netcdf/level_3/{}/{}_hour.nc".format(stid,stid)

data_store = xr.backends.PydapDataStore(open_url(url, user_charset='utf-8'))

data = xr.open_dataset(data_store)
```

Use the `PydapDataStore` and `.sel` or `.isel` to take time "slices":
```
from datetime import datetime, timezone

# Get all data for a specific day
data_day = xr.open_dataset(data_store).sel(time='2023-03-12')

# Get all data for a specific year
data_year = xr.open_dataset(data_store).sel(time='2023')

# Get the latest single obset for this station, using .isel
data_latest = xr.open_dataset(data_store).isel(time=-1)

# Get the latest single obset using .sel and the 'nearest' method
# Both the source time and target time must have the same tz awareness

target_time = datetime.now(timezone.utc).replace(tzinfo=None)
data_latest = xr.open_dataset(data_store).sel(time=target_time, method='nearest')
```

### Environments and pydap versions

Note that pyenv/virtualenv and `pip` only provides `pydap==3.2.2` (tried with python 3.7, 3.8, 3.9). If you want to use the `xr.backends.PydapDataStore` method, you need to use `conda` which provides `pydap==3.3.0` via `conda-forge`. The above code runs in a py38 conda environment.

### Errors encountered with "standard" xarray and netCDF4 methods

Using `xr.open_dataset(url, engine='netcdf4')` (the default `engine`) can result in:
```
OSError: [Errno -68] NetCDF: I/O failure
```

Using `netCDF4.Dataset(url)` can also result in the same `OSError`. This error is further discussed on this [Unidata github issue](https://github.com/Unidata/netcdf4-python/issues/812).

Using `xr.open_dataset(url, engine='pydap')` can result in:
```
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe2 in position 10711: ordinal not in range(128)
```

If you encounter any of these errors, use the alternative `xr.backends.PydapDataStore` method instead.

## Using python and Siphon

Totally untested, but this looks like it could be great to explore further:

[Exploring the THREDDS catalog with Unidata's Siphon](https://ioos.github.io/ioos_code_lab/content/code_gallery/data_access_notebooks/2017-01-18-siphon-explore-thredds.html)

## Web resources
- [OPeNDAP documention](https://opendap.github.io/documentation/UserGuideComprehensive.html)
- [xarray OPeNDAP documentation](https://xarray-test.readthedocs.io/en/latest/io.html#opendap)
- [Pydap documentation](https://www.pydap.org/en/latest/)
- [Deltares, Reading data from OpenDAP using python](https://publicwiki.deltares.nl/display/OET/Reading+data+from+OpenDAP+using+python)
- [Ocean Observatories Initiative, THREDDS example python script](https://oceanobservatories.org/thredds-quick-start/#python)