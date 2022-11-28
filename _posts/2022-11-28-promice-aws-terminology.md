---
title: "PROMICE Automated Weather Station (AWS) terminology and file naming conventions"
author: Penelope How
date: 2022-11-28 17:00
classes: wide
categories:
  - Documentation
tags: 
  - aws
---

Within the [PROMICE](https://promice.dk/) and [GC-Net](http://cires1.colorado.edu/steffen/gcnet/) monitoring programmes, we have a lot of different terminology when referring to our data processing and products. Here is an outline of this terminology.


## Level 0 (L0) to Level 3 (L3) data
`L0`: Level 0 data, the original data that is collected from a given weather station. This means that all observation are unmodified.

`L1`: Level 1 data, the output of the first processing step from `L0`, including conversion of observations from engineering to physical units and basic corrections

`L2`: Level 2 data, the output of the second processing step from `L1`, including cross-sensor corrections

`L3`: Level 3 data, the output of the third and final processing step from `L2`, including the calculation of derived variables such as sensible and latent heat flux


## Raw data vs. transmitted (tx) data
`raw`: 10-minute data stored on the weather station CF-card (an external module on the station logger). *This data can only be collected in the field during a station visit*

`STM`: Hourly-averaged slim table data stored on the internal logger memory of the weather station. *This data can only be collected in the field during a station visit*

`tx`: Hourly- and/or daily-averaged data transmitted via Iridium. The frequency of data transmission depends on the station type and the day of year. *This data can be accessed remotely*

The level (`L0` `L1` `L2` `L3`) and type (`raw` `STM` `tx`) can be combined to form a detailed description of the data being referred to. For example, `L3 tx` is transmitted data processed to a Level 3 product.


## Static vs. near-real-time data
`static`: A static `L3` data product, where data has been collected and processed up til a given point in time. This data does not change over time. For example, our Dataverse weather station data products are `static`

`nrt`: A near-real-time `L3` data product, where data is collected and processed on an hourly basis. This data changes over time as new data (mainly `tx` data) is available 


## One-boom and two-boom stations
`one-boom`: A tripod station design with most sensors attached to one boom, mainly found on the peripheral of the ice sheet and associated with PROMICE

`two-boom`: A mast station design with sensors attached to two booms, mainly used for monitoring the in-land ice sheet conditions and associated with GC-Net


## Processing components
`config` or `toml` file: A TOML-formatted file associated with a given weather station, containing processing parameters for processing its `L0` data to a `L3` data product. These parameters include instrument calibration coefficients, logger type and observation column names

`imei`: The modem number of a given weather station, needed to identify and read transmission messages containing `L0 tx` data

`last_aws_uid` or `uid`: The ID of the last transmission received from any weather station
 
`pypromice`: The Python package that contains all of the processing steps and workflows for weather station data processing and handling


## Further reading

[Fausto et al. (2021)](https://doi.org/10.5194/essd-13-3819-2021) documents the PROMICE one-boom weateher station design and data products in more detail

[GEUS resumes responsibility of GC-Net](https://eng.geus.dk/about/news/news-archive/2020/december/geus-takes-over-american-climate-stations-on-the-greenland-ice-sheet) press release
