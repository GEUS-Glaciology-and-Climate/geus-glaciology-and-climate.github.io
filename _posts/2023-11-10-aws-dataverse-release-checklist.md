---
title: "Procedures for GEUS Dataverse AWS data releases"
author: Penelope How
date: 2023-11-10 00:00
classes: wide
categories:
  - Guides
tags: 
  - pypromice
  - dataverse
---

We release up-to-date PROMICE/GC-Net automated weather station data on our GEUS Dataverse at the beginning of each month. This repository can be found [here](https://doi.org/10.22008/FK2/IW73UU).

**We point users to the data here, so that they can use a version-controlled dataset with a cite-able DOI for publications.**

## Automatic uploading to the GEUS Dataverse

Automatic uploading occurs at the beginning of each month. This is set up on our `azure-aws` server at the time of writing, using the [l3_uploader.py](https://github.com/GEUS-Glaciology-and-Climate/aws-operational-processing/blob/main/l3_uploader.py) and [l3_uploader_wrapper.sh](https://github.com/GEUS-Glaciology-and-Climate/aws-operational-processing/blob/main/l3_uploader_wrapper.sh) scripts.

If a big update to pypromice is occurring then it is best practise wait for the changes to be implemented. All station data should be re-processed if needed these updates affect raw station data (`L0 tx`). To re-process all station data, the [l3_processor.sh](https://github.com/GEUS-Glaciology-and-Climate/aws-operational-processing/blob/main/l3_processor.sh) needs to be run manually, with some changes to prevent older files being filtered out of the processing. Basically, all `-mmin` flags should be removed in instances where file fetching is performed; for example:

```
IMEIs=$(find ./tx -maxdepth 1 -type f -mmin -60 | cut -d"/" -f3)
```

To this: 

```
IMEIs=$(find ./tx -maxdepth 1 -type f | cut -d"/" -f3)
```

After re-processing is completed, the Dataverse uploader ([l3_uploader_wrapper.sh](https://github.com/GEUS-Glaciology-and-Climate/aws-operational-processing/blob/main/l3_uploader_wrapper.sh)) can be manually triggered to perform the upload again:

```
$ ./l3_uploader_wrapper.sh 
```

This creates a draft release of the updated Dataverse repository that needs to be manually verified and published. We could publish automatically, but feel that manual intervention is needed currently.

## Checking a draft release

The draft Dataverse release should be checked, looking at these specific things:

1. Does the last data entry in a given active station file correspond to the end of the preceeding month (i.e. does the dataset cover up to the time of the Dataverse draft?)

2. Make sure there are no duplicate files in the dataset.

3. In the changelog file, does the latest version match the Dataverse release version? Also, does anything need to be added to the changelog (e.g. big pypromice updates, new stations)

4. Is the authorship list up-to-date?

5. Is the pypromice version in the metadata up-to-date?

## Publish and advertise

You can publish the draft release when you have checked it and are happy. Remember to advertise the new release where appropriate, such as Twitter.
