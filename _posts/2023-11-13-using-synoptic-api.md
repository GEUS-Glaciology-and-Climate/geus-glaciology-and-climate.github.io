---
title: "Using the Synoptic API service to retreive GEUS data"
author: Patrick Wright
date: 2023-11-13 00:00
classes: wide
categories:
  - Guides
tags: 
  - synoptic
  - aws
---

# Overview
[Synoptic Data PBC](https://synopticdata.com/) operates several services to access and view surface mesonet data. The GEUS AWS network (`network=281`) is accessible via these services, including:
- [Weather API](https://synopticdata.com/weatherapi/)
- [Download service](https://download.synopticdata.com/) (single station, csv output)
- [Push Streaming service](https://synopticdata.com/value-add-services/push-stream-service/)
- [Data Viewer tool](https://viewer.synopticdata.com/)

All public data (including the GEUS network) is accessible open access and free of charge to anyone using the data for non-profit and/or academic or research purposes. Synoptic has paid tiers for corporate users and to access [value-add services](https://synopticdata.com/value-add-services/) such as advanced quality control and the Synoptic precipitation service.

As with the THREDDS server, any data available via Synoptic services should be considered provisional. Synoptic archives historical data, but this archive will only represent data as it was received (no historical re-processing occurs). The API (and other services) should be used primarily for near real-time data applications. Users should go to Dataverse for historical/citable datasets.

# Using the Weather API service
Documentation: [https://docs.synopticdata.com/services/](https://docs.synopticdata.com/services/)

**NOTE:** All GEUS stations use station IDs in the Synoptic API service with "geus" appended to the ID. For example, South Dome (SDM) is `stid=geussdm`. Station elevation by default is returned in feet, and cannot be changed to metric (this is a very unfortunate legacy attribute of the Synoptic API service, and hopefully will change soon).

## Sign up
[Signing up](https://docs.synopticdata.com/services/welcome-to-synoptic-data-s-web-services) is relatively easy, and will provide you with an API key and token. **The examples below use `token=demotoken`. Once you have your own token from signing up, please use that token instead.** If you have any issues email support@synopticdata.com.

## Latest service
Returns the most recent observation from a station or set of stations.

Documentation: [https://docs.synopticdata.com/services/latest](https://docs.synopticdata.com/services/latest)

**Examples:**

Latest observations from the entire GEUS network, all variables, json output:

[https://api.synopticdata.com/v2/stations/latest?network=281&token=demotoken](https://api.synopticdata.com/v2/stations/latest?network=281&token=demotoken)

Latest air temperature for SDM, json output:

[https://api.synopticdata.com/v2/stations/latest?var=air_temp&stid=geussdm&token=demotoken](https://api.synopticdata.com/v2/stations/latest?var=air_temp&stid=geussdm&token=demotoken)

## Timeseries service
Returns data for a station or set of stations based on a time span.

Documentation: [https://docs.synopticdata.com/services/time-series](https://docs.synopticdata.com/services/time-series)

If you prefer csv output, you can use `output=csv`, but only for a single station.

**Examples:**

Timeseries for the entire GEUS network, Sept 1 to Nov 1, 2023, json output:

[https://api.synopticdata.com/v2/stations/timeseries?network=281&start=202309010000&end=202311010000&token=demotoken](https://api.synopticdata.com/v2/stations/timeseries?network=281&start=202309010000&end=202311010000&token=demotoken)

Time series for the entire GEUS network, recent 240 min (4 hrs), json output:

[https://api.synopticdata.com/v2/stations/timeseries?network=281&recent=240&token=demotoken](https://api.synopticdata.com/v2/stations/timeseries?network=281&recent=240&token=demotoken)

Time series for SDM, Sept 1 to Nov 1, 2023, csv output:

[https://api.synopticdata.com/v2/stations/timeseries?stid=geussdm&start=202309010000&end=202311010000&output=csv&token=demotoken](https://api.synopticdata.com/v2/stations/timeseries?stid=geussdm&start=202309010000&end=202311010000&output=csv&token=demotoken)

## Nearest time service
Returns the observation closest to the time requested.

Documentation: [https://docs.synopticdata.com/services/nearest-time](https://docs.synopticdata.com/services/nearest-time)

**Examples:**

Nearest observation to Nov 1, 2023 00 UTC, within 90 minutes, entire GEUS network, json output:

[https://api.synopticdata.com/v2/stations/nearesttime?network=281&attime=202311010000&within=90&token=demotoken](https://api.synopticdata.com/v2/stations/nearesttime?network=281&attime=202311010000&within=90&token=demotoken)

## Accessing multiple variables (instantaneous and hourly averages)
Each GEUS station reports multiple versions of each variable. In the API response, `_set_n` will be appended to each variable key in the [time series service](https://docs.synopticdata.com/services/time-series) response (e.g. `air_temp_set_1`), and `_value_n` is used for the [latest service](https://docs.synopticdata.com/services/latest) response (e.g. `air_temp_value_1`).

The mapping between GEUS variables and `_set_n` or `_value_n` is as follows:

- hourly average (upper boom), e.g. `t_u`; `_set_1` or `_value_1`
- hourly average (lower boom), e.g. `t_l`; `_set_2` or `_value_2`
- instantaneous (upper boom), e.g. `t_i`; `_set_3` or `_value_3`

This assignment was chosen to have representation in the API latest service (and the [Synoptic viewer tool](https://viewer.synopticdata.com/)) for the greatest number of stations, using the default `_value_1`. Currently (as of summer 2023) only a subset of GEUS stations report hourly instantaneous data every hour, whereas all stations report hourly averages every hour in the summer (DOY 100-300). In the winter (DOY 300-100) transmissions can go down to 24 hr transmission intervals due to lack of solar. When battery capacity is increased to allow hourly transmissions for instantaneous and hourly ave, year round for all stations, it will likely be better to make `_set_1` / `_value_1` the instantaneous ob, as this will better align with what forecasting offices and the WMO more commonly expect as default.

## QC
By default, all API responses remove any observations failing the Synoptic range check. However, GEUS internally performs this same check, so any out-of-range data should already be removed from the Thredds server source which is used by the Synoptic ingest process. In addition to the range check, all open access users can enable two other "basic" QC checks: rate of change check, and persistence check.

To enable all basic QC checks (range, rate and persistence), and remove flagged data, include the following in your api call:

`&qc_checks=basic&qc_remove_data=on`

See the [About QC](https://docs.synopticdata.com/services/mesonet-data-qc) documentation for more information on Synoptic's QC, and see the QC arguments for the api within the docs for each service (timeseries, latest, etc). Any observations flagged by Synoptic's QC are also color-indicated in the Viewer tool.

# Synoptic Viewer tool

Synoptic recently developed a Viewer tool to enable map-based views and timeseries graphs for weather data. For the GEUS network, see:

[https://viewer.synopticdata.com/map/data/now/air-temperature#stationdensity=0&map=3%2F75.38%2F-29.67&networks=281](https://viewer.synopticdata.com/map/data/now/air-temperature#stationdensity=0&map=3%2F75.38%2F-29.67&networks=281)

This default view uses the latest service with `within=90`, so if there is not data within the last 90 minutes, no observations will appear. I find that this is a useful tool for making sure the GEUS network is alive and well! Note that only `value_1` (hourly ave, upper boom, e.g. `t_u`) are displayed here. During DOY 300-100 (winter), any station that is only reporting daily (00 UTC) will only be visible for 90 minutes after the report.

**Station thinning** is implemented for viewing large areas, so unfortunately not all GEUS stations are visible together (you have to zoom in to see all stations in a particular area). There is a "station density" slider, but it is currently restricted at large spatial extents. Hopefully Synoptic will enable the "station density" slider to be increased to show all stations when just a single network is selected!

There is a lot more functionality to explore in the Viewer tool (and Synoptic is actively developing new features), so poke around and share any feedback. You can provide feedback directly with a button in the upper right corner.

# Synoptic ingest methodology

Synoptic reads data from the [GEUS Thredds server](https://thredds.geus.dk/). This occurs as a cron job which runs at :15, :20 and :30 minutes after the hour. The 15 and 20 minute runs are intended to collect processed data that was reported for the most recent hour. The 30 minute run is a "clean up" to make sure all observations for most recent hour were collected. Each run looks for any new station observations that were not collected on a previous run.

Instantaneous data (i.e. `_i` variables) is retrieved for each station (where `s` is a station ID) at the following URL:

https://thredds.geus.dk/thredds/dodsC/aws_l3_station_netcdf/tx/{s}/{s}_hour.nc

Hourly ave (i.e. `_u` and `_l` variables) is retrieved for each station at the following URL:

https://thredds.geus.dk/thredds/dodsC/aws_l3_station_netcdf/level_3/{s}/{s}_hour.nc

For each station URL, Synoptic retrieves recent data using:

`xr.open_dataset(url).isel(time=isel_time)`

`isel_time` can be `-1` (most recent observation) which corresponds to instantaneous observations, or `-2` which corresponds to one hour back, which contains hourly average obs. This follows the convention that GEUS assigns hourly averages to the beginning of the hour. Synoptic then rolls the hourly ave observations forward one hour to comply with Synoptic's end-of-hour reporting convention for hourly averages. Therefore, when you retrieve data from the Synoptic API for the most recent hour, instantaneous and hourly ave will have the same timestamp.

The assigment using `time=-1` and `time=-2` to collect data expects the following structure for an hourly reporting station (using csv file snippet here as an example), where the last row is missing `_u` and `_l` observations:

```
2023-11-13 17:00:00,864.0,-6.3,65.15,69.2564,1.7929,7.335,138.4,4.8699,-5.4851,140.3916,55.7879,51.2848,51.2848,,203.683,275.1206,0.1248,-8.694,-17.8499,49.6969,2.1913,1.0912,10.9234,12.2713,233.6,241.2071,0.0,-3.55,-1.86,-1.273,-1.1,-1.09,-1.04,-1.24,-1.16,-10.87,-7.7006,207.2,64.509107,-49.285333,1119.0,1700.0,,0.9,13.73,74.31,,-6.008,64.53195,-49.30826
2023-11-13 18:00:00,864.0,-6.95,72.13,77.1634,1.8882,9.75,136.9002,6.6619,-7.1191,7.7665,7.4763,6.8729,6.8729,,225.4683,277.8614,0.3813,-8.1825,-23.7226,34.133,2.1955,1.0899,10.9234,12.2713,233.6,241.2071,0.0,-3.563,-1.862,-1.28,-1.1,-1.09,-1.04,-1.24,-1.16,-10.9371,-7.6696,210.1,64.509077,-49.285334,1115.0,1759.59,,0.92,13.79,74.31,,-6.525,64.5108,-49.2451
2023-11-13 19:00:00,,-6.467,72.35,77.0354,,8.01,178.3002,0.2376,-8.0065,-1.2661,0.0,0.274,0.0,,257.6328,281.7453,0.6956,-7.4721,,,2.1985,1.0928,10.9234,12.2713,233.6,241.2071,0.0,-3.573,-1.87,-1.28,-1.1,-1.09,-1.04,-1.24,-1.16,-10.9883,-7.6533,207.5,64.509068,-49.285332,1112.0,1859.59,,0.84,13.76,80.6,,-6.082,64.53534,-49.19065
2023-11-13 20:00:00,862.0,-7.833,90.1,97.2205,2.2079,18.38,105.6999,17.6943,-4.9736,-1.0493,0.0,-0.4799,0.0,,260.2719,282.4309,0.7839,-7.3252,-22.5591,-23.5247,2.1663,1.089,10.8952,12.2619,234.0,241.6151,0.408,-3.582,-1.87,-1.28,-1.1,-1.09,-1.04,-1.238,,-10.852,-7.5906,197.0,64.509073,-49.285326,1118.0,1959.59,,0.95,13.75,72.05,,-7.185,64.53195,-49.30826
2023-11-13 21:00:00,863.0,-7.7,88.6,95.4781,2.1911,19.0,105.2,18.3353,-4.9816,-1.0783,0.0,-0.3263,0.0,,252.1617,282.7353,0.6933,-7.1928,-28.9547,-24.1119,2.1983,1.0913,10.9046,12.2619,234.8,242.4311,0.816,-3.592,-1.878,-1.28,-1.1,-1.09,-1.04,-1.238,-1.16,-10.972,-7.5472,207.2,64.509094,-49.285319,1119.0,2059.59,,0.79,13.73,82.9,,-7.008,64.56849,-49.37155
2023-11-13 22:00:00,,,,,,,,,,,0.0,,0.0,,,,,,,,,,,,235.0,242.6351,0.204,,,,,,,,,-11.562,-7.7846,,64.509079,-49.285439,1120.0,2200.0,,0.79,13.72,81.0,,,64.5108,-49.2451
```

When stations switch to daily reporting at 00 UTC during DOY 300-100 (winter), instantaneous and hourly averages are reported together, both at 00 UTC. In that case, we use `time=-1` and assign the 00 UTC timestamp to both.

All station metadata reported in the Synoptic API (lat, lon, elev, station IDs) is read from https://thredds.geus.dk/thredds/fileServer/metadata/AWS_latest_locations.csv. Synoptic periodically reviews for new stations, or relocated/moved stations, and will update their databases. This will occur at least once per year (but usually more frequently) to account for icesheet movement or any other station relocation.

