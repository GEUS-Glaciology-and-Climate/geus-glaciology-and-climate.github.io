---
title: "THREDDS Data Server: background and guide"
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
<u>**!! Note this is an old document and all new THREDDS updates can be found [here](https://geusgitlab.geus.dk/glaciology-and-climate/promice/thredds) !!**</u>

## THREDDS background

The GEUS Glaciology and Climate THREDDS data server (TDS) is located here:

[https://thredds.geus.dk](https://thredds.geus.dk)

This web server is intended to provide metadata and data access for near-real-time (NRT) and archived datasets using OPeNDAP, OGC WMS and WCS, HTTP, and other remote data access protocols. The TDS is developed and supported by [Unidata](https://www.unidata.ucar.edu/), a division of the University Corporation for Atmospheric Research (UCAR), and is sponsored by the U.S. National Science Foundation.

The GEUS TDS was set up following the [TDS Online Tutorial](https://docs.unidata.ucar.edu/tds/current/userguide/index.html). This tutorial was closely followed for almost every setup step, so if in doubt refer to this resource!

The TDS was built in Feb 2023 by [Patrick Wright](https://github.com/patrickjwright) with assistance on setting up DNS and the Apache reverse proxy provided by [Jakob Molander](https://github.com/jakobmolandergeus). General project oversight provided by [Robert Fausto](https://github.com/robertfausto).

The Unidata THREDDS support is active and very helpful! Contact via email at:

support-thredds@unidata.ucar.edu

## Resource locations on the Azure Thredds VM

- Azure fileshare mount: `/data/geusgk/awsl3-fileshare`
- TDS content (data and config files): `/data/content/thredds`
- [thredds-git](https://github.com/GEUS-Glaciology-and-Climate/thredds-git) repo for TDS files: `/data/thredds-git`
- Tomcat web server: `/opt/tomcat`
- Apache web server: `/etc/apache2`
- Java install: `/usr/lib/jvm/java-1.11.0-openjdk-amd64`

## Troubleshooting
If the Thredds server is down, the first thing to try is restarting the Tomcat web server. First, switch to root user with `su -`.
```
./ /opt/tomcat/bin/shutdown.sh
./ /opt/tomcat/bin/startup.sh
```
If that doesn't work, then try restarting the Apache web server, and then restart the Tomcat server again.
```
sudo service apache2 restart
```
Also keep in mind that if you have not implemented any log file rotation, the server can easily fill disk space. You may need to clear log files and free up disk space.

If things are still down, your best bet it to start looking through logs at `/opt/tomcat/bin/log`, check out the FAQs at the Thredds tutorial site, and potentially contact Thredds support.

## Adding new data

To add any new dataset, you will need ssh access to the Azure Thredds VM instance. Contact [Penny How](https://github.com/PennyHow) for obtaining an ssh key. Any user with the ssh key can access the server through the open port 22. This allows access from home, from Greenland, etc. Please treat these ssh keys carefully, and only share via secure methods (internal geus email addresses, over Slack, etc). The current key is `aws-dec2022.pem` and should occasionally be rotated by creating a new key on the Azure portal.

Many resources on the TDS are only allowed for the `tomcat` or `root` user, but you will be logging in as the `aws` user. For many tasks, you will need to switch to the `root` user (`sudo su`).

### 1. Migrate data to the Azure fileshare

GEUS G&K has an Azure storage account (`geusgk`), with an [Azure fileshare](https://learn.microsoft.com/en-us/azure/storage/files/storage-files-introduction) named `awsl3-fileshare`. If you have access, all Azure resources can be viewed at [https://portal.azure.com/](https://portal.azure.com/#home).

The fileshare is mounted on the Azure Thredds VM at `/data/geusgk/awsl3-fileshare`. The fileshare is also mounted on the Azure AWS VM at the same location, and on the "glacio01" server at `/media/aws/geusgk/awsl3-fileshare`. The capacity of the fileshare is 5 TB. If you need to mount the fileshare to an additional location, see the [Azure file share mounting documentation](https://learn.microsoft.com/en-us/azure/storage/files/storage-how-to-use-files-linux?tabs=smb311).

Any data written to a mounted instance of the fileshare is almost instantly replicated both to the cloud (the Azure `geusgk` storage account), and to any other mounted instance. To add new data, you first need to figure out how to write the data to the fileshare. For example, you could set up a process on glacio01 to download an archival (static) dataset from another web location, and then write the data to the mounted fileshare (perhaps using parallel methods in a `screen` session for large datasets).

#### On-demand vs automatic mount methods

For both the Azure aws processing server and the thredds server, pajwr mounted the fileshare using the "on-demand" instructions (see mounting documentation link above). If the server is shutdown and rebooted, this mount probably will not remain. In which case, it might be better to follow the instructions to "Automatically mount file shares". This might also give you better control over permissions on the mounted instance.

#### az login

If you want to mount the Azure fileshare at another location, the commands in the mounting documentation (linked above) assume you have logged in with `az login` (on glacio01, had to do `az login --use-device-code`). This will take you through the Azure login and auth procedure via a URL and code provided on the command line. You then have to select an account and
authenticate using Microsoft Authenticator. Successful login should display something like:

```
[
  {
    "cloudName": "AzureCloud",
    "homeTenantId": "3a854c6f-9945-45ae-8f72-9f3efcf015ed",
    "id": "0aead173-793e-4b77-8c6a-e0734a2f61a4",
    "isDefault": true,
    "managedByTenants": [],
    "name": "FPJ kreditkort - Betalt efter forbrug",
    "state": "Enabled",
    "tenantId": "3a854c6f-9945-45ae-8f72-9f3efcf015ed",
    "user": {
      "name": "pajwr@geus.dk",
      "type": "user"
    }
  }
]
```

The Azure Command Line Interface (CLI) is installed on both Azure VMs (for any command starting with `az`). The Azure Powershell interface is also installed on the thredds VM.

### 2. Make symlinks to the dataset

Once a new dataset is written to the fileshare, you need to create symlinks to the data so it is visible in the TDS `public` content directory. You can symlink individual files (e.g. `AWS_station_locations.csv`) or directories (e.g. `aws-l3`). For example, a symlink was created for the `aws-l3` directory with:

```
$ ln -s /data/geusgk/awsl3-fileshare/aws-l3 /data/content/thredds/public/aws-l3
```

### 3. Modify catalog xml files

Once the data is linked to `/data/content/thredds/public`, you need to modify the catalog xml files. Symlinks for xml catalog and config files point from `/data/content/thredds` to source files in the `/data/thredds-git` repository. Any new xml files should be created in the git repository, with corresponding symlinks created at `/data/content/thredds`. Modification of existing files can be done directly in the repo, or by opening the symlinks. Commit, add and push any changes to the `thredds-git` repo.

`threddsConfig.xml` controls overall configuration for the TDS, and should not normally need modification for adding new data.

For an entirely new dataset, you will want to create a new catalog file. This catalog file will create a top-level directory on the TDS (e.g. `awsl3Catalog.xml` for `Automatic_Weather_Stations` and `cryoclimCatalog.xml` for `CryoClim`). It will be easiest to copy an existing catalog file, and then modify as needed.

You will need to make a reference to the new catalog file at the bottom of `catalog.xml`, such as:

```
  <catalogRef xlink:title="Automatic_Weather_Stations" xlink:href="awsl3Catalog.xml" name=""/>
  <catalogRef xlink:title="CryoClim" xlink:href="cryoclimCatalog.xml" name=""/>
  <catalogRef xlink:title="SICE" xlink:href="siceCatalog.xml" name=""/>
```

You will also need to add any new dataset to the "exclude" list for the AWS "metadata" directory in `awsl3Catalog.xml`, such as:

```
    <datasetScan name="metadata" id="metadata" path="metadata" location="content/">

      <filter>
        <include wildcard="*.csv" />
        <exclude wildcard="aws-l3" atomic="false" collection="true" />
        <exclude wildcard="aws-l3-flat" atomic="false" collection="true" />
        <exclude wildcard="cryoclim" atomic="false" collection="true" />
        <exclude wildcard="SICE" atomic="false" collection="true" />
        <exclude wildcard="testdata" atomic="false" collection="true" />
      </filter>

    </datasetScan>
```

Getting the data links to display as intended can be difficult (good luck!), but you can **start by following existing patterns in the other catalog files**. The `datasetScan` and `filter` elements are very handy for scanning all files within a directory. Refer to the [TDS tutorial](https://docs.unidata.ucar.edu/tds/current/userguide/index.html) for further information.

The approach I used which replicates `metadata` elements within each `dataset` element seems redundant, but I couldn't get it to work any other way.

### 4. Restart the Tomcat server

For any changes to take effect, you must execute the following:

```
root@thredds:/opt/tomcat/bin# ./shutdown.sh
root@thredds:/opt/tomcat/bin# ./startup.sh
```

Then refresh the production thredds page and you should see your changes. If you are getting a loading wheel for longer than 10-15 seconds, probably something is wrong. There is no information provided at the terminal! You must refer to relevant log files at `/data/content/thredds/logs` or `opt/tomcat/logs`. `opt/tomcat/logs/catalina.out` is a key log file. Often, I just resort to trial and error and restarting Tomcat until content successfully loads.

**NOTE:** Log file rotation needs to be implemented (at least for `catalina.out`), otherwise we will eventually eat up disk space!

## Updating terms of service text

There is a built-in method to insert custom user-defined text into the TDS pages, detailed [here](https://docs.unidata.ucar.edu/tds/current/userguide/customizing_tds_look_and_feel.html#thymeleaf-templates). This uses the `tdsTemplateFragments.html` file to "inject" text into the base Tomcat html templates. It seems like a great option, however I simply could not get it to work! Therefore, I have edited the source Tomcat html template file directly. The current "Terms of service" text is located here:

`/opt/tomcat/webapps/thredds/WEB-INF/templates/commonFragments.html`

The text is also backed up in the `thredds-git` repo in [terms-of-service-text.html](https://github.com/GEUS-Glaciology-and-Climate/thredds-git/blob/main/terms-of-service-text.html). If the Tomcat and/or TDS software is ever upgraded, this edit to the template file will likely be lost! But it will be easy to add back using the git backup.

**NOTE:** In addition to the Terms of Service text, pajwr made one other modification to the `commonFragments.html` file. The links for the "GEUS THREDDS Data Server" text (in both the header and footer) were incorrect using the defaults. In both the `header-content` and `footer-content` sections, I replaced `th:href="${installUrl}"` with `th:href="${contextPath}"`.

## DNS and Apache reverse proxy

Jakob Molander at GEUS set up the DNS entry allowing thredds.geus.dk to point at the public IP of the Azure VM running the TDS. 

Jakob also set up the Apache reverse proxy. This allows incoming requests to first hit the Apache server, which are then securely routed to the TDS Tomcat server. This allows for the following URLs to all redirect to the same location:

- [http://thredds.geus.dk](http://thredds.geus.dk)
- [https://thredds.geus.dk](https://thredds.geus.dk)
- [http://thredds.geus.dk/thredds](http://thredds.geus.dk/thredds)
- [https://thredds.geus.dk/thredds](https://thredds.geus.dk/thredds)

I did go through all steps in [Running The TDS Behind a Proxy Server](https://docs.unidata.ucar.edu/tds/current/userguide/tds_behind_proxy.html) including installation of Apache2 (`apt install apache2`) and installing security certificates from letsencrypt (following [this guide](https://medium.com/@mashrur123/a-step-by-step-guide-to-securing-a-tomcat-server-with-letsencrypt-ssl-certificate-65cd26290b70)), but ultimately I had to bring in Jakob to get this done correctly!

For any questions regarding DNS, Apache or security certificates, contact Jakob Molander (jm@geus.dk).

**NOTE:** See [Restrict Access to the TDS](https://docs.unidata.ucar.edu/tds/current/userguide/restict_access_to_tds.html) in the TDS tutorial if you need to restrict IPs, or restrict access to certain datasets. IP restriction could probably also be done at the "front door" with the Apache reverse proxy.

## Matomo

[Matomo](https://matomo.org/) is used in place of Google Analytics for tracking usage of the TDS. This is an open-source tool that is internally hosted at GEUS. There is a 500 GB data disk mounted at `/db` that hosts the Matomo MySQL database. Access Matomo at the following URL:

[https://thredds.geus.dk/matomo/](https://thredds.geus.dk/matomo/)

user: `admin`, password is the same all-powerful aws user password as used elsewhere.


## Admin and manager pages

These pages provide access to tweak config settings and do all sort of other amazing things I have not yet explored! The endpoints are located here:

- [https://thredds.geus.dk:8443/admin/](https://thredds.geus.dk:8443/admin/)
- [https://thredds.geus.dk:8443/manager/](https://thredds.geus.dk:8443/manager/)

user: `admin`, password is the same all-powerful aws user password as used elsewhere.

## Software version updates

It will be wise to keep the software updated over time, in particular the THREDDS Data Server and Tomcat. We are currently running:

- TDS Version 5.4
- Tomcat 8.5.85
- Java 11 (installed with `sudo apt-get install openjdk-11-jdk`)

If there is a new major version for TDS, this should be adopted. Check the TDS tutorial and upgrade to the recommended Tomcat and Java versions when appropriate.

Thanks for reading, and enjoy your THREDDS journey!
