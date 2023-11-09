---
title: "Procedures for a new pypromice release"
author: Penelope How
date: 2023-11-09 12:00
classes: wide
categories:
  - Guides
tags: 
  - aws
  - python
---

There are a couple of steps for making a new pypromice release. This is just a reminder checklist of what these steps are. **Please make sure that all steps are completed, and ask if in doubt.**

## 1. Change the pypromice package version
When a new pypromice version is due, make a new branch on the [pypromice Github repo](https://github.com/GEUS-Glaciology-and-Climate/pypromice) and change the `version` in the [setup.py](https://github.com/GEUS-Glaciology-and-Climate/pypromice/blob/main/setup.py) file. Remember to use semantic versioning, specifying a major (e.g. `v1.0.0`), minor (e.g.`v0.1.0`) or patch (e.g. `v0.0.1`) version depending on the nature of the updates to pypromice

## 2. Make a PR to the main branch
Make a pull request (PR) and merge it with the main branch once all checks have passed and someone else has reviewed it

## 3. Make a new pypromice release
Make a new [pypromice release](https://github.com/GEUS-Glaciology-and-Climate/pypromice/releases). The naming convention for these releases are:

- Title: `pypromice v<version>` e.g. `pypromice v1.0.0`
- Tag: `v<version>` e.g. `v1.0.0`

You can auto-generate the release notes, which will contain all merged PRs since the last release. Releases should **always** be set as the latest release

## 4. Check the pypromice pypi distribution is built and uploaded
The [pypi distribution of pypromice](https://pypi.org/project/pypromice/) should automatically be built and uploaded after a release is published. Check that the Github Action for this has run successfully [here](https://github.com/GEUS-Glaciology-and-Climate/pypromice/actions/workflows/pypi-publish.yml). If it has not run successfully, it could be:
- You have not updated the version in the [setup.py](https://github.com/GEUS-Glaciology-and-Climate/pypromice/blob/main/setup.py), and pypi cannot overwrite versions. If this has happened, delete the existing release you made, update the version in setup.py in a new PR, and make a new release
- The API token for pypi has expired. This should be updated in pypi and in the Github repo secrets

## 5. Check the pypromice Dataverse entry is updated and publish
The [GEUS Dataverse pypromice entry](https://doi.org/10.22008/FK2/3TSBF0) should also automatically update with the latest version after a pypromice release is published. This will create a new draft release in the GEUS Dataverse. Check the draft, then update the title to reflect the latest version:

- Title: `pypromice v<version>` e.g. `pypromice v1.0.0`

Update the authorship list and metadata if needed, and then publish the update.

## 6. Update the pypromice conda feedstock
The [pypromice conda feedstock](https://github.com/conda-forge/pypromice-feedstock) contains the conda-forge build for pypromice (which is available [here](https://anaconda.org/conda-forge/pypromice)). To update the build to the latest version, make a fork of the feedstock repo (there might be one already [here](https://github.com/GEUS-Glaciology-and-Climate/pypromice-feedstock), make a new branch, and update the `recipe/meta.yaml` file. This can be updated automatically with the `grayskull` package as so:

```
grayskull pypi pypromice
```

This will generate the `meta.yaml` file based on the latest pypi release of pypromice. It is mainly the `version` and `sha256` entries that will be changed with the updates. Once generated, make sure that the tests and meta information is also updated. The file should look something like this:

```
{% set name = "pypromice" %}
{% set version = "1.3.1" %}

package:
  name: {{ name|lower }}
  version: {{ version }}

source:
  url: https://pypi.io/packages/source/{{ name[0] }}/{{ name }}/{{ name }}-{{ version }}.tar.gz
  sha256: 6934511933611f7d6792c8c75e7306974f367cfef1692adf4e730c3e3b8f4d69

build:
  entry_points:
    - get_promice_data = pypromice.get.get_promice_data:get_promice_data
    - get_l0tx = pypromice.tx.get_l0tx:get_l0tx
    - get_l3 = pypromice.process.get_l3:get_l3
    - join_l3 = pypromice.process.join_l3:join_l3
    - get_watsontx = pypromice.tx.get_watsontx:get_watsontx
    - get_bufr = pypromice.postprocess.get_bufr:get_bufr
    - get_msg = pypromice.tx.get_msg:get_msg
  noarch: python
  script: {{ PYTHON }} -m pip install . -vv
  number: 0

requirements:
  host:
    - python >=3.8
    - pip
  run:
    - python >=3.8
    - numpy >=1.23.0
    - pandas >=1.5.0
    - xarray >=2022.6.0
    - toml
    - scipy >=1.9.0
    - scikit-learn >=1.1.0
    - bottleneck
    - netcdf4
    - pydataverse

test:
  imports:
    - pypromice
  commands:
    - pip check
    - python -m unittest discover pypromice
    - get_promice_data --help
    - get_l0tx --help
    - get_l3 --help
    - join_l3 --help
    - get_watsontx --help
    - get_msg --help
  requires:
    - pip

about:
  home: https://github.com/GEUS-Glaciology-and-Climate/pypromice
  summary: A toolbox for handling and processing PROMICE (Programme for Monitoring of the Greenland Ice Sheet) and GC-Net (Greenland Climate Network) automated weather station data
  license: GPL-2.0
  license_file: LICENSE.txt
  doc_url: https://pypromice.readthedocs.io
  dev_url: https://github.com/GEUS-Glaciology-and-Climate/pypromice

extra:
  recipe-maintainers:
    - PennyHow
    - ladsmund
    - patrickjwright
    - mankoff
```

Note the import and command tests (needed to test the conda build), and the links and description. 

Make a PR between your forked branch and the conda-forge feedstock main branch. Once all checks have passed and the branch has been merged, then the new conda package version should be built.
