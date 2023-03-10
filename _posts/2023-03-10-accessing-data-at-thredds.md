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

NetCDF can also be accessed using [OPeNDAP](https://www.opendap.org/) and other data access methods listed for individual files (e.g. see data access services listed for [KAN_U](https://thredds.geus.dk/thredds/catalog/aws_l3_station_netcdf/level_3/KAN_U/catalog.html?dataset=aws_l3_station_netcdf/level_3/KAN_U/KAN_U_hour.nc)). See [here](https://www.ncei.noaa.gov/access/thredds-user-guide) for a concise explanation of the available TDS data access services.

## Using python, csv

The methods below have been tested with python >= 3.8. Required imports are listed with each method.

```
# Read single station, csv with pandas
# ===================================
import pandas as pd

stid = 'NUK_Uv3'
url = "https://thredds.geus.dk/thredds/fileServer/aws_l3_station_csv/level_3/{}/{}_hour.csv".format(stid,stid)
data = pd.read_csv(url)

# Pandas dataframes are much more useful after setting the index to datetime:
data.set_index(pd.to_datetime(data.time), inplace=True)
data.drop(['time'], axis=1, inplace=True) # drop original time column
```

```
# Make dict of dataframes for all stations, csv with pandas
# =========================================================
import pandas as pd

# Read AWS_station_locations.csv to get station names
url = "https://thredds.geus.dk/thredds/fileServer/metadata/AWS_station_locations.csv"
locations = pd.read_csv(url)

data = {}
for stid in locations.stid: # this loop takes ~30 sec
  stn_url = "https://thredds.geus.dk/thredds/fileServer/aws_l3_station_csv/level_3/{}/{}_hour.csv".format(stid,stid)
  df = pd.read_csv(stn_url)
  df.set_index(pd.to_datetime(df.time), inplace=True)
  df.drop(['time'], axis=1, inplace=True) # drop original time column
  data[stid] = df
```

## Using python, NetCDF OPeNDAP

With initial testing, pajwr encountered errors with decoding NetCDF files served via OPeNDAP on THREDDS via standard methods using `xarray`, `netcdf4`, or `pydap` (see errors below). This issue is present with the GEUS OPeNDAP NetCDF URLs, as well as other THREDDS servers (such as [thredds.ucar.edu](https://thredds.ucar.edu/thredds/catalog/catalog.html)).

The following successful solution uses `xr.backends.PydapDataStore` as described [here](https://help.marine.copernicus.eu/en/articles/5182598-how-to-consume-the-opendap-api-and-cas-sso-using-python#h_33df7ebcce).

```
import xarray as xr
from pydap.client import open_url

stid = 'NUK_Uv3'
url = "https://thredds.geus.dk/thredds/dodsC/aws_l3_station_netcdf/level_3/{}/{}_hour.nc".format(stid,stid)

data_store = xr.backends.PydapDataStore(open_url(url, user_charset='utf-8')) # needs PyDAP >= v3.3.0

data = xr.open_dataset(data_store)
```

`data` now looks like:
```
In [1]: data
Out[1]: 
<xarray.Dataset>
Dimensions:       (time: 22100)
Coordinates:
  * time          (time) datetime64[ns] 2020-08-31T15:00:00 ... 2023-03-10T10...
Data variables: (12/63)
    p_u           (time) float64 ...
    t_u           (time) float64 ...
    rh_u          (time) float64 ...
    rh_u_cor      (time) float64 ...
    qh_u          (time) float64 ...
    wspd_u        (time) float64 ...
    ...            ...
    wspd_i        (time) float64 ...
    wdir_i        (time) float64 ...
    msg_i         (time) float64 ...
    lon           float64 ...
    lat           float64 ...
    alt           float64 ...
Attributes: (12/66)
    station_id:                      NUK_Uv3
    id:                              dk.geus.promice:2ca065e1-8dca-3fe2-a2bf-...
    history:                         Generated on 2023-03-10T10:08:09.669905
    date_created:                    2023-03-10T10:08:09.669919
    date_modified:                   2023-03-10T10:08:09.669919
    date_issued:                     2023-03-10T10:08:09.669919
    ...                              ...
    publisher_url:                   https://promice.dk
    references:                      Fausto, R. S., van As, D., Mankoff, K. D...
    references_bib:                  @article{Fausto2021, doi = {10.5194/essd...
    standard_name_vocabulary:        CF Standard Name Table (v77, 19 January ...
    summary:                         \"The Programme for Monitoring of the Gr...
    _NCProperties:                   version=2,netcdf=4.9.0,hdf5=1.12.2
```

Make sure you use `conda` and install `pydap` with:

```
$ conda install -c conda-forge pydap
```

Currently you must use a `conda` (or miniconda) env, since pyenv/virtualenv and `pip` only provides `Pydap==3.2.2` (tried with python 3.7, 3.8, 3.9). `conda` provides `Pydap==3.3.0`, which is required to use `xr.backends.PydapDataStore`. I am using a py38 conda environment.

### Errors encountered with other xarray and netCDF4 methods

Using `xr.open_dataset(url,engine='pydap')` results in:
`UnicodeDecodeError: 'ascii' codec can't decode byte 0xe2 in position 10711: ordinal not in range(128)`

Using `xr.open_dataset(url,engine='netcdf4')` results in:
`OSError: [Errno -68] NetCDF: I/O failure`

Using `netCDF4.Dataset(url)` also results in the same `OSError`.

## Using python and Siphon

Totally untested, but this looks like it could be great to explore:

[Exploring the THREDDS catalog with Unidata's Siphon](https://ioos.github.io/ioos_code_lab/content/code_gallery/data_access_notebooks/2017-01-18-siphon-explore-thredds.html)

## Web resources
- [xarray OPeNDAP documentation](https://xarray-test.readthedocs.io/en/latest/io.html#opendap)
- [Deltares, Reading data from OpenDAP using python](https://publicwiki.deltares.nl/display/OET/Reading+data+from+OpenDAP+using+python)
- [Ocean Observatories Initiative, THREDDS example python script](https://oceanobservatories.org/thredds-quick-start/#python)