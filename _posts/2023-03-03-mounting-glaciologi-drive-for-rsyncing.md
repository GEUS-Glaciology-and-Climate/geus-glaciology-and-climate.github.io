---
title: "Re-mounting the Glaciologi drive for AWS data syncing"
author: Penelope How
date: 2023-01-09 13:00
classes: wide
categories:
  - Guides
tags: 
  - aws
---

## Where does our Level 3 AWS data reside?

The `Level 3` automated weather station (AWS) datasets from the [PROMICE and GC-Net monitoring programmes](https://promice.org) live in numerous places:

1. The [GEUS Dataverse](https://dataverse.geus.dk/dataverse/AWS), where monthly product updates are made. This is the final, curated version of the AWS data. 
2. Our [thredds server](https://thredds.geus.dk), where near-real-time products can be found. This is considered an operational, uncurated version of the AWS data.
3. Our Gitlab [aws-l3](https://geusgitlab.geus.dk/glaciology-and-climate/promice/aws-l3) and [aws-l3-dev](https://geusgitlab.geus.dk/glaciology-and-climate/promice/aws-l3-dev) repos. *Internal use and access only!*
4. Our internal server drive, `glaciologi`. *Internal use and access only!*


## What data is on the glaciologi drive?

The `glaciologi` drive holds a mirror copy of our [aws-l3-dev](https://geusgitlab.geus.dk/glaciology-and-climate/promice/aws-l3-dev) repo. **This is Level 3 data produced by our developmental processing workflow.** Therefore, the data held here may not be the same as those found in our publicly available spaces. 

The data is mirrored from the GEUS processing server to the `glaciologi` drive, in the `glaciologi/AWS` directory. This updates every hour with the newest Level 3 data from the transmitted data (`tx`) from each AWS. 


## What if I cannot see new data on the glaciologi drive?

There can be some reasons that new data is not showing up in the `glaciologi` drive.

1. The most common reason is because there has been an outage in our servers, which has unmounted the drive for mirroring
2. Another reason may be that new additions are being tested on the development side that are not processing correctly and uploading to [aws-l3](https://geusgitlab.geus.dk/glaciology-and-climate/promice/aws-l3).


### How to re-mount the glaciologi drive

If the drive is unmounted, there are some simple steps that you can perform to re-mount the drive and fetch the latest data. First, connect to the `glacio01` server where the developmental processing workflow is being performed.

```
$ ssh glacio01
```

It is likely that you will be prompted for your username and password. Please remember that these are credentials for access to the server, not your GEUS credentials.

Next, we can remount the `glaciologi` drive as so:

```
$ sudo mount -rw -t cifs -o username=<USERNAME>,uid=<UID>,gid=<GID> //172.23.254.160/glaciologi /media/aws/glaciologi
```

Where `<USERNAME>` is your `glacio01` username, and `<UID>` and `<GID>` are the identifiers associated with your username and your group. If you do not know the `<UID>` and `<GID>` then you can look them up with this command:

```
id
```

Once the `glaciologi` drive is remounted, perform the following `rsync` routine to fetch the latest data:

```
sudo rsync -r --exclude .git /mnt/data/aws/pypromice_aws/aws-l3 /media/aws/glaciologi/AWS
```

Be aware that you need sudo privileges in order to do this. You may be prompted for two passwords - one for your `glacio01` user and another for your GEUS account (i.e. in order to access the `glaciologi` drive).

If no new data appears on the `glaciologi` drive after performing these steps then it is likely that the problems lies in the developmental processing workflow.

