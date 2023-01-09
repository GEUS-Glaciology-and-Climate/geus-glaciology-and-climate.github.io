---
title: "WMO BUFR processing: background and guide"
author: Patrick WRight
date: 2023-01-09 13:00
classes: wide
categories:
  - Guides
tags: 
  - aws
  - pypromice
  - wmo
---

The following is a general background and guide to the processing of BUFR files for submission of PROMICE and GC-Net data to the World Meteorological Organization (WMO), via ftp upload to the Danish Meteorological Institue (DMI).

## Project background and motivation

The submission of near-real-time (NRT) data from PROMICE and GC-Net to the WMO has been a long-standing goal at GEUS. The data is intended to help improve numerical weather prediction (NWP) models, and will establish a role for GEUS as a data provider for an operational product. This integration of the GEUS weather station network will increase funding potential and awareness of the data product in the glaciology and meteorology communities.

Primary developers at GEUS are Patrick Wright and Penny How, in collaboration with Bjarne Amstrup and Erna Mourentza Beckmann at DMI. Project oversight by Robert Fausto and Andreas Ahlstrom at GEUS.

Project timeline (to be updated as work progresses):

- Fall 2021: Initial communications with DMI and stub out initial BUFR processing code. Penny How and Ken Mankoff determined that the PROMICE processing code first needed to be rewritten as a python3 package ([pypromice](https://github.com/GEUS-Glaciology-and-Climate/pypromice)) before continuing.
- October - December 2022: BUFR processing code re-written to production state by Patrick Wright.
- December 2022 - January 2023: Work with Erna Beckmann at DMI to register stations with [OSCAR/Surface](https://oscar.wmo.int/surface/#/) using [WIGOS identifiers](https://community.wmo.int/activity-areas/WIGOS/implementation-WIGOS/WIGOS-station-identifier).
- January 2023: BUFR files posted hourly to DMI ftp directory for further review by Bjarne Amstrup at DMI (data not yet submitted to WMO). Code debugging and updates as needed.

## BUFR processing workflow

### getBUFR

The BUFR processing is "driven" by `pypromice/bin/getBUFR`, which is called in `l3_processor.sh` as:

```
echo '======================================='
echo "Running BUFR file export for DMI/WMO..."

echo 'Removing all bufr files from BUFR_out'
cd /home/pwright/GEUS/pypromice-dev/pypromice/
rm src/pypromice/postprocess/BUFR_out/*.bufr

echo 'Running getBUFR...'
getBUFR --positions

echo 'Running bufr_wrapper.sh...'
cd /data/pypromice_aws/
./bufr_wrapper.sh

echo 'DONE WITH BUFR!'
echo '======================================='
```

`getBUFR` imports modules from `pypromice.postprocess.csv2bufr` and config objects from `pypromice.postprocess.wmo_config`, and completes the following:

- Reads transmitted data files from `aws-l3/tx/*/*_hour.csv`
- Uses the most recent single (and valid) observation from each station to create a BUFR file, written to `src/pypromice/postprocess/BUFR_out/`. Included variables are air temperature, relative humidity, pressure, wind speed and wind direction. In the future, we can consider including radiation variables and precipitation.

`getBUFR` has several command-line args which can be configured in the `argparse` section at the top of the code (also viewable with `getBUFR --help`).

Running with the `--positions` flag (currently default in production) will write the most recent transmitted station positions to `aws-l3/AWS_station_locations.csv`. These positions are from a linear regression fit using the previous 3 months of transmitted data for each station. GPS-transmitted position data is used if available, otherwise modem-derived positions scraped from the email text are used (if modem-derived positions are used, no elevation is available). These are the same positions written to the BUFR files. The linear best fit procedure uses `sklearn` and is completed in `csv2bufr.linear_fit`.

### bufr_wrapper.sh

`bufr_wrapper.sh` completes the following:

- If the `BUFR_out` directory exists, and is empty, then `exit`. Nothing to do.
- If BUFR files are present, concat all files to a single BUFR file named with the convention `geus_YYYYMMDDThhmm.bufr`.
- Use in-line python to read `credentials/credential.ini` using `configparser`. Seems a bit silly to use python like this, but I could not find a clean or easy way to read a .ini file with bash.
- Use credentials to upload concatenated BUFR file to DMI ftp `upload` directory.

Note that we are currently using plain ftp to complete this upload as specified by DMI. However, this is not a very secure method and we may want to consider asking DMI to switch to sftp or ssh methods in the future.

### Usage of wmo_config.py

In addition to the `argparse` args in `getBUFR`, the WMO BUFR processing is intended to be controlled by editing `wmo_config.py`.

`stid_to_skip` and `vars_to_skip` are used to defined stations and variables to be skipped in the BUFR processing. See the comments in these sections for further instructions. **It will be important to maintain this section and review for changes monthly or bimonthly!** As stations are discontinued, suspect/bad data is corrected, and v3 stations rotate into the mix, we will need to update these dicts.

`ibufr_settings` contains all the parameters set in the BUFR files. Almost all of our stations are considered mobile and use the "synopMobil" template (parameters set under the `mobile` key). Two stations (`WEG_B` and `KAN_B`) are land-based and use the "synopLand" template (parameters set under the `land` key).

The station numbers we are registering (listed under `shipOrMobileLandStationIdentifier` and `stationNumber` keys), are the 5-digit DMI identifiers, which correspond to part of the registered [WIGOS identifiers](https://community.wmo.int/activity-areas/WIGOS/implementation-WIGOS/WIGOS-station-identifier). For example, `CEN` is ID 04407, which has the WIGOS ID of 0-208-0-04407.

The only exception to `wmo_config.py` being the single location for BUFR processing setup, is some station-specific code in `csv2bufr.setStation`. For the purposes of looking up the correct station ID in `wmo_config.ibufr_settings['station']` this section renames `THU_U2` to `THU_U`, `JAR_O` and `SWC_O` to `JAR` and `SWC`, and `CEN2` to `CEN`. This code should not need to change, but is good to be aware of.

### Timestamps pickle file and usage of --dev flag

We use a `latest_timestamps.pickle` file to keep track of the most recent timestamp for each station. Pickle files are serialized python objects used for fast read/write times and compressed file size. I use this approach out of good practice and general habit, though this is a very small file that could easily have just been `.csv` or another plain-text format. The pickle file is created from the `current_timestamps` dictionary.

The pickle file approach uses the following logic:

- If the file is not present, create it and write the most recent timestamps for each station from the current processing run.
- If the file is present, load it into a python dictionary as `latest_timestamps`. Initiate a new empty `current_timestamps` dict.
- For each station, use logic (lines ~140-180 in `getBUFR`) to find the most recent valid timestamp, and write an entry to the `current_timestamp` dict.
- We proceed with BUFR processing only `if (current_timestamp > latest_timestamp) and (current_timestamp > two_days_ago)`
- After processing, the `current_timestamps` dict is written back to the pickle file on disk, and will then be the `latest_timestamps` for the next run.

There is no need to manually create this pickle file. If missing it is automatically created, and it is over-written with each processing run. If you are running `getBUFR` for the first time (e.g. after setting up a fresh processing environment), the first run will not see the pickle file and will therefore create the file with the most recent timestamps (and no BUFR files are created). If subsequent runs are made before further transmissions are received, `getBUFR` will see that the times in the pickle file are the exact same as the time in the observations to be processed, so no BUFR processing will be completed. In this case, the BUFR processing will not proceed until new transmissions are received (usually the second hourly processing run).

You can run `getBUFR` with the `--dev` flag to over-ride the timestamp restrictions. This is useful for running `getBUFR` repeatedly for development, where you want to have station observations run through the full BUFR processing each time. In this case, the timestamp checking logic is modified as:

```
        if args.dev is True:
          # If we want to run repeatedly (before another transmission comes in), then don't
          # check the actual latest timestamp, and just set to two_days_ago
          latest_timestamp = two_days_ago

        if (current_timestamp > latest_timestamp) and (current_timestamp > two_days_ago):
```
Using the `--dev` flag, as long as you have transmission within the last two days, observations will run through every time.

## Data latency

Reducing data latency can be very important in real-time applications, especially when related to high-impact weather events. As of early January 2023, we have a latency of approximately 7 minutes. This is defining latency as the time elapsed between the time an observation is made on the icesheet, to the the time the concatenated BUFR file is uploaded to DMI.

Overall, 7 minutes is a very low latency considering we are dealing with satellite-transmitted data from a remote location. Running the processing at 5 minutes after the hour seems to be optimal timing to collect top-of-hour transmissions, with the BUFR file processing finishing about 2 minutes into the processing run, resulting in 7 minutes of total latency. Rough testing has shown that transmitted messages from the top of the hour are finished sending by minutes 3 and 4 after the hour, so running at 5 minutes after the hour will collect all transmitted messages. Further testing could be done here to establish exact timing.

Some improvements to consider in the future that could further reduce latency include:

- Increase CPUs on the Microsoft Azure server. This will allow much faster `parallel` processing. However, this greatly increases server cost. This could possibly be offset using the Azure CLI to deallocate the server between processing runs (which will stop the VM's compute costs).
- Change the processing to only pipe through a limited window of recent time necessary to complete processing (e.g. the time needed for smoothing procedures in `pypromice`). Perhaps just the previous week of data, rather than processing the entire station history with every hourly run. Then, take the processed recent data and append to the long-term L3 record for each station. This may require some substantial rewrites, as well as a method to enable full station history re-processing when `pypromice` code is changed.
- Use a "data-driven" process to commence processing. For example, keep checking the email server for new messages after the top of the hour, and when 30 seconds (or maybe 1 minute) has elapsed with no new messages, commence processing.

## Web pages and documentation resources

To see the registered GEUS stations at OSCAR/Surface, go to https://oscar.wmo.int/surface/#/search/station, under the "Organization" drop-down search for and select "GEUS", then click "Search".

The WIGOS Data Quality Monitoring system map can be viewed [here](https://wdqms.wmo.int/nwp/land_surface/six_hour/availability/pressure/all/2022-11-03/18).

Documentation for ECMWF `ecCodes` library:
https://confluence.ecmwf.int/display/ECC/Documentation

`ecCodes` BUFR element table for WMO master table version 32:
https://confluence.ecmwf.int/display/ECC/WMO%3D32+element+table

Processing steps orignally based on this example:
https://confluence.ecmwf.int/display/UDOC/How+do+I+create+BUFR+from+a+CSV+-+ecCodes+BUFR+FAQ

See here for a step-by-step guide on the eccodes set-up:
https://gist.github.com/MHBalsmeier/a01ad4e07ecf467c90fad2ac7719844a

WMO Guide to Instruments and Methods of Observation:
https://library.wmo.int/index.php?id=12407&lvl=notice_display#.Y7wtStLMIUG
