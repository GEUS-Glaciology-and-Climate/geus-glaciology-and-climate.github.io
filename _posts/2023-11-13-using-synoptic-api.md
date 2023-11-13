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

## Overview
[Synoptic Data PBC](https://synopticdata.com/) operates several services to access and view surface mesonet data. The GEUS AWS network (`network=281`) is accessible via these services, including:
- [Weather API](https://synopticdata.com/weatherapi/)
- [Download service](https://download.synopticdata.com/) (single station, csv output)
- [Push Streaming service](https://synopticdata.com/value-add-services/push-stream-service/)
- [Data Viewer tool](https://viewer.synopticdata.com/)

All public data (including the GEUS network) is accessible free of charge to anyone using the data for non-profit and/or academic or research purposes. Synoptic has paid tiers for corporate users and to access [value-add services](https://synopticdata.com/value-add-services/) such as advanced quality control and the precipitation service.

## Accessing GEUS data using the Weather API service

**NOTE:** All GEUS stations use station IDs in the Synoptic API service with "geus" appended to the ID. For example, South Dome (SDM) is `stid=geussdm`.

### Sign up
Signing up is relatively easy, and will provide you with an API key and token. Follow the instructions [here](https://docs.synopticdata.com/services/welcome-to-synoptic-data-s-web-services). **The examples below use `token=demotoken`. Once you have your own token from signing up, please use that instead.** If you have any issues email support@synopticdata.com.

### Latest service
Documentation: https://docs.synopticdata.com/services/latest

Latest observations from the entire GEUS network, all variables, json output:

https://api.synopticdata.com/v2/stations/latest?network=281&token=demotoken

Latest air temperature for SDM, json output:

https://api.synopticdata.com/v2/stations/latest?var=air_temp&stid=geussdm&token=demotoken

### Timeseries service
Documentation: https://docs.synopticdata.com/services/time-series

Time series for the entire GEUS network, Sept 1 to Nov 1, 2023, json output:

https://api.synopticdata.com/v2/stations/timeseries?network=281&start=202309010000&end=202311010000&token=demotoken

Time series for the entire GEUS network, recent 240 min (4 hrs), json output:

https://api.synopticdata.com/v2/stations/timeseries?network=281&recent=240&token=demotoken

Time series for SDM, Sept 1 to Nov 1, 2023, csv output:

https://api.synopticdata.com/v2/stations/timeseries?stid=geussdm&start=202309010000&end=202311010000&output=csv&token=demotoken

### Nearest time service
Documentation: https://docs.synopticdata.com/services/nearest-time

Nearest observation to Nov 1, 2023 00 UTC, within 90 minutes, entire GEUS network, json output:

https://api.synopticdata.com/v2/stations/nearesttime?network=281&attime=202311010000&within=90&token=demotoken

### Multiple variables
Each GEUS station reports multiple versions of each variable. In the API response, `_set_n` will be appended to each variable key in the [time series service](https://docs.synopticdata.com/services/time-series) response (e.g. `air_temp_set_1`), and `_value_n` is used for the [latest service](https://docs.synopticdata.com/services/latest) response (e.g. `air_temp_value_1`).

The mapping between GEUS variables and `_set_n` or `_value_n` is as follows:

hourly average (upper boom); `_set_1` or `_value_1`
hourly average (lower boom); `_set_2` or `_value_2`
instantaneous (upper boom); `_set_3` or `_value_3`

This assignment was chosen to have representation in the API latest service (and the [Synoptic viewer tool](https://viewer.synopticdata.com/)) for the greatest number of stations, using the default `_value_1`. Currently (as of summer 2023) only a subset of GEUS stations report hourly instantaneous data every hour, whereas all stations report hourly averages every hour in the summer (DOY 100-300). In the winter (DOY 300-100) transmissions can go down to 24 hr transmission intervals due to lack of solar. When battery capacity is increased to allow hourly transmissions for instantaneous and hourly ave, year round for all stations, it will likely be better to make set_1 / value_1 the instantaneous ob, as this will better align with what forecasting offices and the WMO more commonly expect.

### QC
By default, all API responses remove any observations failing the Synoptic range check. However, GEUS internally performs this same check, so any out-of-range data should already be removed from the Thredds server source which is used by the Synoptic ingest process. In addition to the range check, all "free tier" users can enable two other "basic" QC checks: rate of change check, and persistence check.

To enable all basic QC checks, and remove data, include the following in your api call:

`&qc_checks=basic&qc_remove_data=on`

See the [About QC](https://docs.synopticdata.com/services/mesonet-data-qc) documentation for more information on Synoptic's QC, and see the QC arguments for the api within the docs for each service (timeseries, latest, etc).

## Synoptic Viewer tool

## Synoptic ingest methodology


