---
title: "AWS data checklist whilst in the field"
author: Penelope How
date: 2024-05-13 00:00
classes: wide
categories:
  - Guides
tags: 
  - aws
  - promice
---

# For the field teams

## 1. Tasks before entering the field

- Check the current status of the automated weather station (AWS) site that you are visiting. If you do not have adequate internet to look at the most recent station data then contact the data scientist team who can check for you. The following locations are where you can check station data (in order of preference):
    * [Our thredds server](https://thredds.geus.dk/) for Level 3 near-real-time data
    * [The promice website](https://promice.org/weather-stations/) for Level 0 near-real-time data
    * [Our Level 0 repository](https://geusgitlab.geus.dk/glaciology-and-climate/promice/aws-l0) for Level 0 near-real-time data

- Turn on your GPS tracking and share the link with the data scientist team. The data scientist team can then see which station you are visiting each day without having direct communication


## 2. Urgent tasks in the field

- **You must tell a member of the data scientist team immediately if a logger box has been switched** (i.e. the IMEI modem number associated with the station's Iridium transmissions has changed). This is because others rely on our near-real-time transmissions and this relies on keeping all IMEI modem numbers up-to-date


## 3. Tasks to do when back in the office

- Report the changed sensors on an AWS to the data scientist team. This should be reported on the AWS maintenance sheet anyway, so hopefully it is not any extra work. Sensors of particular interest are:
    * Radiometer - we especially need to know the serial number of the new radiometer, so we can update the associated radiometer calibration parameters for our data processing
    * Pressure Transducer (PTA) - we especially need to know if a new PTA has been installed so that we can update the associated coefficients for data processing

- Report if there have been any other changes to the station, such as:
    * Location - if the station was moved then we would like to know so that we can cross-check it in the station's GPS readings
    * Boom positioning - i.e. if the station's boom direction has been drastically changed
    * Upgrade - for example, if a station has been upgraded from a version 2 to version 3 station
     
- Transfer any raw data collected from the field to the data scientist team



# For the data scientist team
## Data scientist personnel

- Penelope How, [pho@geus.dk](mailto:pho@geus.dk)
- Mads Christian Lund, [maclu@geus.dk](maclu@geus.dk)
- Robert Fausto, [rsf@geus.dk](rsf@geus.dk)
- Baptiste Vandecrux, [bav@geus.dk](bav@geus.dk)


## Updating data configuration for a station

Configurations for all stations can be found in [our Level 0 repository](https://geusgitlab.geus.dk/glaciology-and-climate/promice/aws-l0). 

The config `.toml` files in the `tx` directory are needed to fetch near-real-time messages from a station. Most of the contents of each of these `.toml` files concerns the `L0` to `L3` processing parameters.

To make changes to these files, make a new branch in the repo and submit the modifications as a merge request. The repo has checks implemented to ensure that the new changes are compatible for processing, and another member of the data scientist team should assess the changes before merging. 

### Switching IMEI modems 

The `modem` variable is needed for the `L0 TX` retrieval because:

- It defines the `imei` modem number, which is used to identify `L0 TX` messages from the specified station
- It defines the active period that the modem was at the specified station (this is in cases where modems are swapped between stations, or stations are reconfigured)

The `modem` variable is set at the beginning of a `tx` config `toml` file as follows:

```
modem = [[imei,start,end], [imei,start,end], [imei,start,end]...]
```

Where each `[imei,start,end]` entry designates the active period of a specific modem. In cases where the modem is active up until present, the `end` variable can be disregarded. So a standard modem that has been active for the entire period of a station's occupancy would look like this: 

```
# Modem at active station example
modem = [['123456789', '2021-04-01 00:00:00']]
```

So modem `123456789` has been active from `2021-04-01 00:00:00` and remains active now, in this example. In cases where a modem has been switched out, we can add an entry into the `modem` variable:

```
# Modem switch example
modem = [['1234567890', '2021-04-01 00:00:00', '2021-06-01 00:00:00'],
	  ['2345678901', '2021-06-02 00:00:00']] 

# Modem multiple switches example
modem = [['1234567890', '2021-04-01 00:00:00', '2021-06-01 00:00:00'],
	  ['2345678901', '2021-06-02 00:00:00', '2022-03-15 00:00:00'],
	  ['3456789012', '2022-03-16 00:00:00'] 
```

We can also use this variable to schedule breaks in the `L0 TX` message transmissions, for instance, when we have a station visit and know that maintenance will produce invalid measurements:

```
# Station visit example
modem = [['1234567890', '2021-04-01 00:00:00', '2021-06-01 00:00:00'],
	  ['1234567890', '2021-06-03 00:00:00']] 	  
```

## Updating sensor coefficients

Sensor coefficients can be updated for each `raw`/`tx` data file associated with a config `toml` file.

The radiometer calibration coefficients are defined here with parameters:

- `dsr_eng_coef`: Downwelling shortwave coefficient
- `usr_eng_coef`: Upwelling shortwave coefficient
- `dlr_eng_coef`: Downweling longwave coefficient
- `ulr_eng_coef`: Upwelling longwave coefficient

All radiometers are calibrated before they leave for the field, and Glaciolab should have all the documentation for the calibration coefficients. 

PTA coefficients are defined here with parameters:
 
- `pt_z_coef`: PTA pressure coefficient
- `pt_z_p_coef`: PTA atmospheric pressure coefficient
- `pt_z_factor`: PTA factor
- `pt_antifreeze`: PTA anti-freeze percentage (for fluid density correction)

Here is an example of these parameters defined at NUK_Uv3:

```
['NUK_Uv3_300534063817730_4.txt']
format     = 'TX'
skiprows = 0
latitude = 64.51
longitude = 49.26
hygroclip_t_offset = 0      
dsr_eng_coef = 13.79  
usr_eng_coef = 14.71
dlr_eng_coef = 9.79
ulr_eng_coef = 9.92
pt_z_coef = 0.41073
pt_z_p_coef = 1008.4
pt_z_factor = 2.5
pt_antifreeze = 50
boom_azimuth = 0
tilt_y_factor = -1 # New inclinometer, y direction is opposite than the old PROMICE v2 inclinometer.
columns = ["time", "rec", "p_u", "t_u", "rh_u", "wspd_u", "wdir_u", "dsr",
	"usr", "dlr", "ulr", "t_rad", "z_boom_u", "z_stake", "z_pt",
	"t_i_1", "t_i_2", "t_i_3", "t_i_4", "t_i_5", "t_i_6", "t_i_7", "t_i_8",
	"tilt_x", "tilt_y", "rot", "precip_u", "gps_time", "gps_lat", "gps_lon",
	"gps_alt", "gps_hdop", "fan_dc_u", "batt_v", "p_i",
 	"t_i","rh_i","wspd_i","wdir_i","msg_i"]
```
