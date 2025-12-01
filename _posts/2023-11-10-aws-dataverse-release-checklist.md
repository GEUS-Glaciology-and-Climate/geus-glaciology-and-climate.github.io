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

Automatic uploading occurs at the beginning of each month. This procedure is defined [here](https://geusgitlab.geus.dk/glaciology-and-climate/promice/aws-operational-processing). We also perfom uploads in between these monthly releases if there are big updates to the processing pipeline or new data ingestions. 

The automated procedue creates a draft release of the updated Dataverse repository that needs to be manually verified and published. We could publish automatically, but feel that manual intervention is needed currently.

## Checking a draft release

The draft Dataverse release should be checked, looking at these specific things:

1. Does the last data entry in a given active station file correspond to the end of the preceeding month (i.e. does the dataset cover up to the time of the Dataverse draft?)

2. Make sure there are no duplicate files in the dataset.

3. In the changelog file, does the latest version match the Dataverse release version? Also, does anything need to be added to the changelog (e.g. big pypromice updates, new stations)

4. Is the authorship list up-to-date?

5. Is the pypromice version in the metadata up-to-date?

## Publish and advertise

You can publish the draft release when you have checked it and are happy. Remember to advertise the new release where appropriate.
