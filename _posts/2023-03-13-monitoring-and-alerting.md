---
title: "Monitoring and alerting"
author: Patrick Wright
date: 2023-03-13 00:00
classes: wide
categories:
  - Guides
tags: 
  - aws
  - azure
  - monitor
  - alert
---

Our python monitoring scripts are located at the [aws-monitor-alert](https://github.com/GEUS-Glaciology-and-Climate/aws-monitor-alert) Github repo. See the README for a description of the monitoring methods. These scripts are simple by design, to hopefully ensure that they continue to run!

All alert emails are sent from geus.aws@gmail.com.

## Glacio01 monitor

This is a version of a simple "ping" monitor to make sure glacio01 is alive (`alert_glacio.py`). Every hour, glacio01 makes an ssh connection to the azure-aws VM and "touches" an empty .txt file (`glacio01_monitor.txt`), which updates the last-updated time for the file. A few minutes afterwards, azure-aws runs a check of the file update time to make sure it is <60 min old.

**E-mail alert subject:** "ALERT: glacio01 down!"

**Only one alert email will be sent**, to prevent repeat messsaging during extended outages.

**Action:** Check if glacio01 is accessible via ssh. If it is currently accessible, it could have been down temporarily. Check to see if known cron processes are still running, and drives such as glaciologi are still mounted. If glacio01 is not accessible, email GEUS IT at geusithjaelp@geus.dk.

You can also check `~/aws-monitor-alert/glacio01_monitor/stdout` on azure-aws, which normally will look like:
```
Running alert_glacio.py at Tue Jan 17 13:32:26 UTC 2023
glacio01_monitor.txt is current. No alert issued.
FINISHED
```

## AWS processing monitors

The DMI BUFR file and `aws-l0` & `aws-l3` file update times are monitored by `alert_processing.py`. These all complete simple checks to verify that file update times are <60 min old.

**E-mail alert subjects:**
- "ALERT: DMI ftp BUFR upload has stopped!"
- "ALERT: aws-l0/tx files are not updating!"
- "ALERT: aws-l3/tx files are not updating!"
- "ALERT: aws-l3/level_3 joined files are not updating!"

Because these alerts need to be addressed as soon as possible, **hourly alert emails will continue to be sent** as long as the alert is still being triggered.

**Action:** For some reason, all or part of the pypromice aws processing has stopped. Time to start investigating...

The best first-stop is to look at `process_stdout` and `process_stderr` at `/data/pypromice_aws/aws-operational-processing` on the aws-azure VM. Error tracebacks should appear in the `stderr` file. If any changes are required in [`pypromice`](https://github.com/GEUS-Glaciology-and-Climate/pypromice), talk to Penny How about how to get fixes into production.

You can also verify that files are not updating (and see how long processing has been down) by looking at `tail` on either the `aws-l3/tx` or `aws-l3/level_3` files. Alternatively, you can grab a data csv file (or `AWS_station_locations.csv`) from the THREDDS server and check update times. You can also check out commits directly on the gitlab repos online.

A successful run with no alerts issued will appear in `~/aws-monitor-alert/aws_processing_monitor/stdout` as:

```
Running alert_processing.py at Tue Jan 17 13:32:43 UTC 2023
Logging into ftpserver.dmi.dk
Latest BUFR: geus_20230117T1307.bufr
DMI BUFR file is current. No alert issued.
aws-l0/tx files are current. No alert issued.
aws-l3/tx files are current. No alert issued.
aws-l3/level_3 files are current. No alert issued.
FINISHED
```

### DMI BUFR backfilling

If the DMI BUFR upload process has been down for an extended period, you should email Bjarne Amstrup at DMI (bja@dmi.dk) and let him know. We collect hourly backup concatenated BUFR files at `/data/pypromice_aws/pypromice/src/pypromice/postprocess/BUFR_backup` for the previous 48 hrs. These files should be manually backfilled into the DMI ftp `upload` directory for the missing time period. See `pypromice_aws/aws-operational-processing/bufr_wrapper.sh` for ftp methods. **TO DO:** pajwr will try to make a dedicated script to do this for a defined time range.

## Azure virtual machines (VMs)

The Azure portal listing all of GEUS GK's resources is found at [https://portal.azure.com/](https://portal.azure.com/). You must be logged into your GEUS account, and have access to Azure. Contact Sune K. Bech (skb@geus.dk) for access. This is where you can create new resources, configure existing resources, and set up Azure monitoring tools.

We currently use the default built-in monitor and alert metrics that can be elected when you create a VM. These include standard VM-specific metrics such as disk space, CPU, memory, etc. When thresholds are exceeded, emails are currently sent to pajwr@geus.dk.

**NOTE:** It is possible to create custom metrics and submit them to Azure (anything that can be calculated in a script, and sent to Azure). This seemed a bit overkill for now, so I have written our own python monitoring scripts instead (see above). There are also very nice dashboard customization options. I created one "Shared dashboard" on our account just to experiment (you can find this on the portal home page).

