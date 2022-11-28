---
title: "Linking a GitHub repo to the GEUS Dataverse"
author: Penelope How
date: 2022-11-28 16:00
classes: wide
categories:
  - Guides
tags: 
  - dataverse
  - github
---

## Why link your GitHub repo to the GEUS Dataverse?
A GitHub repository (i.e. repo) are great for storing, sharing, and working on scripts. You may want your GitHub repo to have a linked entry on the [GEUS Dataverse](https://dataverse.geus.dk/) if:

- You want your repo to be citeable with a Dataverse DOI
- You want to make an official release that is linked to an associated dataset on Dataverse
- You want to avoid using unsupported platforms within the GEUS organisation, such as Zenodo

Your GitHub repo can be linked to the GEUS Dataverse using [GitHub Actions](https://github.com/features/actions), whereby a pre-defined action in the repo (e.g. pushing, new release) will automatically sync to a designated Dataverse dataset entry.


## Using GitHub Actions to publish to the GEUS Dataverse
[Dataverse](https://github.com/IQSS/dataverse) has provided a [dataverse-uploader](https://github.com/IQSS/dataverse-uploader) and accompanying [walkthrough guide](https://github.com/marketplace/actions/dataverse-uploader-action) on how to set up a GitHub Action. Instructions here are adapted for use with the GEUS Dataverse specifically.

## Ingredients for using GitHub Actions
Several items are needed to set this up:
- `.yml` action workflow file in your GitHub repo
- GEUS Dataverse dataset entry
- GEUS Dataverse API token
- Encrypted secret in your GitHub repo

And the following steps...

### 1. Defining the action workflow
The GitHub Action is defined in a `.yml` file in your GitHub repo. This can be created through the online portal or in your local repo space:

- In the repo online portal at [https://github.com](https://github.com), navigate to the Actions tab on the top menu and click "Create workflow"
- In your local repo space, create a new file at `.github/workflows/`

The file can be named anything, so call it something appropriate such as `dataverse_workflow.yml`. This file will be an action workflow that is either executed at trigger event (such as when you push to the GitHub repo or publish a release) or executed manually. 

The action workflow for publishing the GitHub repo contents to the GEUS Dataverse will look something like this:

```
# Repo to Dataverse workflow
on: 
  release:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Send repo to GEUS Dataverse 
        uses: IQSS/dataverse-uploader@v1.3
        with:
          DATAVERSE_TOKEN: ${{secrets.DATAVERSE_TOKEN}}
          DATAVERSE_SERVER: https://dataverse.geus.dk
          DATAVERSE_DATASET_DOI: doi:10.22008/FK2/ABCDE1
          GITHUB_DIR: src
          DELETE: True
          PUBLISH: False
```

Change the input parameters accordingly, which have been adapted from the [dataverse-uploader walkthrough guide](https://github.com/IQSS/dataverse-uploader/blob/master/README.md):

| Parameter | Required | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| --------- | -------- |--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `DATAVERSE_TOKEN` | **Yes** | This is your personal access token that you can create from your GEUS Dataverse account. We'll set this up later [here](#2.-defining-the-geus-dataverse-dataset-entry). |
| `DATAVERSE_SERVER` | **Yes** | The URL of your Dataverse installation, so for the GEUS Dataverse this is [https://dataverse.geus.dk]([https://dataverse.harvard.edu](https://dataverse.geus.dk)).                                                                                                                                                                                                                                                                                                                                                                  |
| `DATAVERSE_DATASET_DOI` | **Yes** | This action requires that a dataset (with a DOI) exists on the GEUS Dataverse server. Make sure to specify your DOI in this format: `doi:<doi>`, i.e., `doi:10.22008/FK2/ABCDE1`.                                                                                                                                                                                                                                                                                                     |
| `GITHUB_DIR` | No | Use `GITHUB_DIR` if you would like to upload files from only a specific subdirectory in your GitHub repository (e.g. `src/` directory only).                                                                                                                                                                                                                                                                                                                                           |
| `DELETE` | No | Can be `True` or `False` (by default `True`) depending on whether all files should be deleted in the dataset on Dataverse before upload.                                                                                                                                                                                                                                                                                                                                       |
| `PUBLISH` | No | Can be `True` or `False` (by default `False`) depending on whether you'd like to automatically create a new version of the dataset upon upload. If `False`, the uploaded dataset will be a `DRAFT`.                                                                                                                                                                                                                                                                            |

Commit and push this file to your repo once you are happy. For the optional input parameters, you can either delete or comment them out (with `#`) from the action workflow `.yml` file if you do not wish to use them. So, in the example above we are only uploading to an entry at `doi:10.22008/FK2/ABCDE1` (`DATAVERSE_DATASET_DOI: doi:10.22008/FK2/ABCDE1`), from the `src` directory (`GITHUB_DIR: src`), removing Dataverse contents on new uploads (`DELETE: True`), and only uploading to a Dataverse draft entry (`PUBLISH: False`). These can be changed according to your needs at any time.

The action on which this workflow will be executed is defined after `on:`. In this example, the workflow is executed on a repo [release](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository), signified by the input keyword `release`. This can be changed to other keywords, such as `push` to execute the action on a repo [push](https://github.com/git-guides/git-push).


### 2. Defining the GEUS Dataverse dataset entry
In order to sync your GitHub repo to the GEUS Dataverse, a Dataverse dataset entry needs to be assigned with a DOI to reference to. This could either be:

- A pre-existing Dataverse entry, which you add your repo contents to (**remember** to set the `DELETE` parameter to `True` in the `.yml` action workflow if you want it to sync without deleting the exisiting Dataverse contents) 
- A new Dataverse dataset entry, which you can create a draft of and edit the metadata. Metadata is not populated automatically from the GitHub repo (e.g. authors, description, keywords etc.), so this has to be done separately


### 3. Generating the GEUS Dataverse API token
A Dataverse API token needs to be generated in order to access the GEUS Dataverse from GitHub. Log on to the [GEUS Dataverse](https://dataverse.geus.dk/loginpage.xhtml?redirectPage=%2Fdataverse.xhtml) and navigate to `API token` through the header menu on the portal. An API token should be automatically generated when you navigate to this page, or can be generated by clicking `Create token` or `Recreate token`. Make a copy of this API token for the [next step](#adding-the-api-token-to-your-github-repo-as-an-encrypted-secret).

See the [Dataverse guide](https://guides.dataverse.org/en/latest/user/account.html#how-to-create-your-api-token) for more information about API tokens.


### 4. Adding the API token to your GitHub repo as an encrypted secret 
The Dataverse API token is added to your GitHub repo as an encrypted secret, which is a safe environment variable for your repo. Encrypted secrets are used specifically in GitHub Action workflows. Whilst any collaborator can re-define a secret, its existing contents cannot be viewed and/or copied by anyone once it is created.

To add the API token as an encrypted secret, navigate to your repo online, and under the repo name, click `Settings`. In the `Security` section of the sidebar menu, click `Secrets` and then `Actions`. 

You can create a new repository secret, defining its names to match the name in the `.yml` action workflow. In this example, we will name our secret `DATAVERSE_TOKEN` to match its definition under the `DATAVERSE_TOKEN` parameter - `DATAVERSE_TOKEN: ${{secrets.DATAVERSE_TOKEN}}`. Copy and paste your GEUS Dataverse API token into the file contents and click `Add secret`.


### 5. Execute the action workflow
In this example, the action workflow will be executed when a repo release is made. After creating a release, you will find the repo contents in your Dataverse dataset entry (either published or in draft form depending on your `PUBLISH` input parameter).


## Are there any alternatives?
[Zenodo](https://zenodo.org) provides simple publishing from GitHub repos, as described [here](https://docs.github.com/en/repositories/archiving-a-github-repository/referencing-and-citing-content). However, it makes more sense to use the Dataverse when handling GEUS content to keep data and workflows centralised.


## Links
A working example of this set-up is available to view here:
- pypromice Dataverse entry - https://doi.org/10.22008/FK2/IPOHT5
- pypromice GitHub repo - https://github.com/GEUS-Glaciology-and-Climate/pypromice
