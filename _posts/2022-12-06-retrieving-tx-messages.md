---
title: "Retrieving Level 0 transmission data (L0 TX) from a weather station"
author: Penelope How
date: 2022-12-06 11:00
classes: wide
categories:
  - Documentation
tags: 
  - aws
  - pypromice
---

## Fetching L0 TX messages

Level 0 transmission messages, `L0 TX`, are collected and transmitted in near-real-time every hour to a designated account space. We can read and process these messages with `pypromice`.

Generally, we read `L0 TX` messages on an operational basis so only performing this on the most recent messages. However, there may be instances where we need to read older messages and/or reconstruct `L0 TX` datasets.

We can do this with the `tx` module in `pypromice`.

## Configuring pypromice

The `pypromice` package should be used to fetch transmission messages, specifically the `tx` module. Make a Python environment with the `pypromice` package dependencies and clone the `pypromice` repo from GitHub.

```
# Create environment with conda
conda create --name pypromice python=3.11
conda activate pypromice

# Clone repository
git clone https://github.com/GEUS-Glaciology-and-Climate/pypromice.git

# Navigate to tx module
cd pypromice

# Local install the package
pip install .
```

## Using pypromice to fetch L0 TX messages

You will need certain files with pieces of account information to fetch `L0 TX` messages:

- The IMAP address and account name for where transmission messages are collected, stored in an `.ini` file (e.g. `accounts.ini`)
- The account password, also stored in an `.ini` file (e.g. `credentials.ini`) 
- The directory of the config `.toml` files for all the AWS' that you wish to grab `L0 TX` messages from
- `payload_formats.csv` and `payload_types.csv` files for decoding messages, which are provided in the `pypromice` repo
- The message number you want to start grabbing messages from, in a file called something like `last_aws_uid.ini`.
- The output directory (e.g. `out_dir`) for fetched `L0 TX` files


Run the script from the command line as follows, exchanging each variable input for your chosen file names and directories, and the transmission messages should begin to appear in the output directory:

```
get_l0tx -a accounts.ini -p credentials.ini -c config/ -u last_aws_uid.ini -o out_dir/
```

The output files will be `.txt` formatting, comma delimited, with one message per line.


### Config .toml file formatting

A config `.toml` file is needed to fetch messages from a station. Most of its contents concerns the `L0` to `L3` processing parameters. However, the `modem` variable is needed for the `L0 TX` retrieval because
- It defines the `imei` modem number, which is used to identify `L0 TX` messages from the specified station
- It defines the active period that the modem was at the specified station (this is in cases where modems are swapped between stations, or stations are reconfigured)

The `modem` variable is set as follows:

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


### Config .toml file example
Here is an example config `.toml` file from `SDM` station, where modem `500534060529221` has been active since `2021-04-01 00:00:00`.

```
station_id = 'SDM'
logger_type = 'CR1000X'
number_of_booms = 2
nodata     = ['-999', 'NAN'] # if one is a string, all must be strings
modem = [['500534060529221', '2021-04-01 00:00:00']]

['SDM_500534060529221_1.txt']
format     = 'TX'
skiprows = 0
latitude = 63.15 
longitude = 44.82
hygroclip_t_offset = 0 
dsr_eng_coef = 14.75
usr_eng_coef = 14.44
dlr_eng_coef = 9.71
ulr_eng_coef = 8.76 
tilt_y_factor = -1 
columns = ['time','rec','p_l','p_u','t_l','t_u','rh_l','rh_u',
 'wspd_l','wdir_l','wspd_u','wdir_u','dsr',
 'usr','dlr','ulr','t_rad','z_boom_l','z_boom_u',
 't_i_1','t_i_2','t_i_3','t_i_4','t_i_5','t_i_6','t_i_7',
 't_i_8','t_i_9','t_i_10','t_i_11','tilt_x','tilt_y','rot',
 'precip_l','precip_u','gps_time','gps_lat','gps_lon',
 'gps_alt','gps_hdop','fan_dc_l','fan_dc_u','batt_v', 'p_i',
 't_i','rh_i','wspd_i','wdir_i','msg_i']
```
