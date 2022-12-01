---
title: "Linking a Gitlab repo to the GEUS Dataverse"
author: Penelope How
date: 2022-12-01 14:00
classes: wide
categories:
  - Guides
tags: 
  - dataverse
  - gitlab
---

This is a similar post to the one described [here](https://geus-glaciology-and-climate.github.io/guides/linking-a-github-repo-to-the-geus-dataverse/) about linking a GitHub repo to the [GEUS Dataverse](https://dataverse.geus.dk/). I would recommend reading that post first for a general overview of the steps and set-up components needed. 

The steps for setting this up with Gitlab are pretty similar for linking a repo to the GEUS Dataverse, but there are some key differences.

## Using Gitlab CI/CD to construct a pipeline
Gitlab CI/CD is similar to GitHub Actions in that you can construct jobs to run automatically at set times. This should be a `yml` file located at the top directory of your repo and named `.gitlab-ci..yml`. The contents of this job is fairly similar to the GitHub `yml` action workflow.

```
# Gitlab-to-Dataverse uploader 

image: python:3.9

dataverse_uploader:
  variables:
    DATAVERSE_TOKEN: "$DATAVERSE_API_TOKEN"
    DATAVERSE_SERVER: "https://dataverse.dataverse.dk"
    DATAVERSE_DATASET_DOI: "doi:10.22008/FK2/IPOHT5"
    GITHUB_DIR: "./"
    DELETE: "true"
    PUBLISH: "false"

  script:
    - apt-get update && apt-get install -y git python3-pip
    - git clone https://github.com/IQSS/dataverse-uploader.git dataverse-uploader
    - cd dataverse-uploader
    - pip install -r requirements.txt
    - echo "$DATAVERSE_TOKEN" "$DATAVERSE_SERVER" "doi:$DATAVERSE_DATASET_DOI" "$CI_PROJECT_URL" "$DELETE" "$PUBLISH"
    - python dataverse.py "$DATAVERSE_TOKEN" "$DATAVERSE_SERVER" "doi:$DATAVERSE_DATASET_DOI" "$CI_PROJECT_URL" -r "$DELETE" -p "$PUBLISH"
```

## Scheduling a pipeline
Once a pipeline has been created, how often it runs can be set using the `Schedules` settings under the `CI/CD` menu in Gitlab. This is essentially a cron job syntax, so say we wanted a job to run every day at 12.00 then it would look something like this:

```
0 12 * * *
```

Another option is to have the pipeline run after a trigger event has happened on the repo. This can either be defined in `Settings >> CI/CD >> Pipeline triggers` or can be added to the top of the `.gitlab-ci..yml` file with the `workflow` and `rules` parameters

```
workflow:
    rules:
        - if: $CI_PIPELINE_SOURCE == "push"
```

This example above executes the pipeline after something has been pushed to the repo, for example.

## Further reading
[Introduction to Gitlab CI/CD](https://docs.gitlab.com/ee/ci/quick_start/)

[Terminology for CI/CD yml workflow](https://docs.gitlab.com/ee/ci/yaml/index.html)

[Gitlab CI/CD workflow rules documentation](https://docs.gitlab.com/ee/ci/jobs/job_control.html)

[Converting GitHub Actions to Gitlab CI/CD](https://forum.gitlab.com/t/converting-github-action-to-gitlab-ci-cd/60912)

[The Dataverse uploader repo](https://github.com/IQSS/dataverse-uploader)
