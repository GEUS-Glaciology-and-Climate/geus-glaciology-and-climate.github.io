---
title: "THREDDS Data Server"
author: Patrick Wright
date: 2023-03-06 00:00
classes: wide
categories:
  - Guides
tags: 
  - thredds
  - aws
  - promice
  - cryoclim
---

## THREDDS background

The GEUS Glaciology and Climate THREDDS data server (TDS) is located at:

[https://thredds.geus.dk](https://thredds.geus.dk)

This web server is intended to provide metadata and data access for near-real-time (NRT) and archived datasets using OPeNDAP, OGC WMS and WCS, HTTP, and other remote data access protocols. The TDS is developed and supported by [Unidata](https://www.unidata.ucar.edu/), a division of the University Corporation for Atmospheric Research (UCAR), and is sponsored by the U.S. National Science Foundation.

The TDS was set up following the [TDS Online Tutorial](https://docs.unidata.ucar.edu/tds/current/userguide/index.html). This tutorial was closely followed for almost every setup step, so if in doubt refer to this resource!

The TDS was built in Feb 2023 by [Patrick Wright](https://github.com/patrickjwright) with assistance on setting up an Apache reverse proxy provided by [Jakob Molander](https://github.com/jakobmolandergeus). The original GEUS Thredds server (no longer supported) was built and maintained by [Robert Fausto](https://github.com/robertfausto).

## Technical stack

The diagram below shows the Azure virtual machine (VM) hosting the THREDDS data server in context with the rest of the technical stack for AWS processing (current as of March, 2023).

![tech_stack](https://raw.githubusercontent.com/GEUS-Glaciology-and-Climate/geus-glaciology-and-climate.github.io/master/assets/images/AWS_server_resources.png)

## Resource locations on the Azure Thredds VM

Azure fileshare mount: `/data/geusgk/awsl3-fileshare`

TDS content (data and config setup): `/data/content/thredds`

git repo for TDS files: `/data/thredds-git` ([https://github.com/GEUS-Glaciology-and-Climate/thredds-git](https://github.com/GEUS-Glaciology-and-Climate/thredds-git))

Tomcat web server: `/opt/tomcat`

Apache web server: `/etc/apache2`

Java install: `/usr/lib/jvm/java-1.11.0-openjdk-amd64`

## Adding new data

To add any new dataset, you will need ssh access to the Azure Thredds VM instance. Contact Penny How for obtaining an ssh key. Any user with the ssh key can access the server through the open port 22. This allows access from home, from Greenland, etc. Please treat these ssh keys carefully, and only share via secure methods (internal geus email addresses, over Slack, etc). The current key is `aws-dec2022.pem` and should occasionally be rotated by creating a new key on the Azure portal.

Many resources on the TDS are only allowed for the `tomcat` or `root` user, but you will be logging in as the `aws` user. For many tasks, you will need to switch to the `root` user (`sudo su`) to get things done.

### Migrate data to the Azure fileshare

GEUS G&K has an Azure storage account (`geusgk`), with an [Azure fileshare](https://learn.microsoft.com/en-us/azure/storage/files/storage-files-introduction) named `awsl3-fileshare`. This is mounted on the Azure Thredds VM at `/data/geusgk/awsl3-fileshare`. The fileshare is also mounted on the Azure AWS VM at `/data/geusgk/awsl3-fileshare`, and on the "glacio01" server at `/media/aws/geusgk/awsl3-fileshare`. The capacity of the fileshare is 5 TB. If you need to mount the fileshare to an additional location, see the [Azure file share mounting documentation](https://learn.microsoft.com/en-us/azure/storage/files/storage-how-to-use-files-linux?tabs=smb311).

Any data written to a mounted instance of the fileshare is replicated both to the cloud (the `geusgk` storage account), and to any other mounted instance. To add new data, you first need to figure out how to write the data to the fileshare. For example, you could set up a process on glacio01 to download an archival (static) dataset from another web location, and write the data to the mounted fileshare (perhaps using parallel methods in a `screen` session for large datasets).

NOTE: For both the Azure aws processing server and the thredds server, pajwr mounted the fileshare using the "on-demand" instructions (see mounting documentation link above). If the server is shutdown and rebooted, this mount probably will not remain. In which case, it might be better to follow the instructions for "Automatically mount file shares" (i.e. register the mount in `fstab`).

The Azure Command Line Interface (CLI) is installed on both Azure VMs. The Azure Powershell interface is also installed on the thredds VM.

### Make symlinks to the dataset

Once a new dataset is written to the fileshare, you need to create symlinks to the data so it is visible in the TDS content directory. You can symlink individual files (e.g. `AWS_station_locations.csv`) or directories (e.g. `aws-l3`). For example, a symlink can be created for the `aws-l3` directory with:

`$ ln -s /data/geusgk/awsl3-fileshare/aws-l3 /data/content/thredds/public/aws-l3`

### Modify catalog xml files

Once the data is located in `/data/content/thredds/public`, you need to modify the catalog xml files. These files should be created and/or modified in the `/data/thredds-git` repository. Symlinks for xml catalog and config files point from `/data/content/thredds` to the repo versions. Commit, add and push any changes.

`threddsConfig.xml` controls overall configuration for the TDS, and should not normally need modification for adding new data.

For an entirely new dataset, you will want to create a new catalog file. This catalog file will create a top-level directory on the TDS (e.g. `awsl3Catalog.xml` for `Automatic_Weather_Stations` and `cryoclimCatalog.xml` for `CryoClim`). It will be easiest to copy an existing catalog file, and then modify as needed.

You will also need to make a reference to the new catalog file at the bottom of `catalog.xml`, such as:

```
  <catalogRef xlink:title="Automatic_Weather_Stations" xlink:href="awsl3Catalog.xml" name=""/>
  <catalogRef xlink:title="CryoClim" xlink:href="cryoclimCatalog.xml" name=""/>
```

Getting the data links to display as intended can be difficult (good luck!), but you can start by following existing patterns in the other catalog files. The `datasetScan` and `filter` elements are very handy for scanning all files within a directory. Refer to the [TDS tutorial](https://docs.unidata.ucar.edu/tds/current/userguide/index.html) for further information.

### Restart the Tomcat server

For any changes to take effect, you must execute the following:

```
root@thredds:/opt/tomcat/bin# ./shutdown.sh
root@thredds:/opt/tomcat/bin# ./startup.sh
```

Then refresh the production thredds page and you should see your changes. If you are getting a loading wheel for longer than 10-15 seconds, probably something is wrong. There is no information provided at the terminal! You must refer to relevant log files at `/data/content/thredds/logs` or `opt/tomcat/logs`. `opt/tomcat/logs/catalina.out` is a key log file. Often, I just resort to trial and error and restarting Tomcat until content successfully loads.

## Updating terms of service text

There is a built-in method to insert custom user-defined text into the TDS pages, detailed [here](https://docs.unidata.ucar.edu/tds/current/userguide/customizing_tds_look_and_feel.html#thymeleaf-templates). This uses templates to "inject" text into the base Tomcat html templates. It seems like a great tool, however I simply could not get it to work! Therefore, I have edited the Tomcat html file directly. The current "Terms of service" text is located here:

`/opt/tomcat/webapps/thredds/WEB-INF/templates/commonFragments.html`

The text is also backed up in the `thredds-git` repo ([terms-of-service-text.html](https://github.com/GEUS-Glaciology-and-Climate/thredds-git/blob/main/terms-of-service-text.html)). If the Tomcat and/or TDS software is ever upgraded, this edit to the template file will be lost! But it will be easy to add back using the git backup.

## Accessing data at the THREDDS server

Stay tuned...

## Apache reverse proxy

## Software updates


