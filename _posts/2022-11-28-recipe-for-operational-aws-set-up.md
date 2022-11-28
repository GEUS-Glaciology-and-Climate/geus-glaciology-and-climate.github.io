title: "Recipe for an operational PROMICE AWS set-up"
author: Penelope How
date: 2022-11-28 18:30
classes: wide
categories:
  - Documentation
tags: 
  - aws

This is a walkthrough guide of the current set-up for fetching `L0 tx` messages and performing `L0` to `L3` processing on both `tx` and `raw` data collected from PROMICE automated weather stations (AWS).

## Ingredients
There are several ingredients you need to perform:

- email message >> `L0 tx` >> `L3 tx` processing 
- `L0 raw` >> `L3 raw` processing

These ingredients and their directory structuring is shown below (be aware that only important subfolder components are listed here):

```
pypromice_aws
│   l3_processor.sh
│
└───aws-l0
│   │	    
│   └───level_0
│   │       config
│   └───tx
│	    config
│
└───aws-l3
│   │	    
│   └───level_0
│   │       config
│   └───tx
│	    config
│
└───credentials
│       accounts.ini
│       credentials.ini
│       last_aws_uid.ini   
│   
└───pypromice
    │   setup.py
    │
    └───bin
    │       getL0TX
    │       getL3
    │       getData
    │
    └───src/pypromice
            L0toL1.py
            L1toL2.py
            L2toL3.py
            __init__.py
            aws.py
            data_urls.csv
            get.py
            metadata.csv
            payload_formats.csv
            payload_types.csv
            tx.py
            variables.csv
```

### AWS L0 Gitlab repo

This is the Gitlab repo space where PROMICE and GC-Net AWS Level 0 data collected in the field (`raw` `stm`) and transmitted (`tx`) can be found. You can make a local copy of this repo using `git clone`. *This is only open to GEUS Glaciology and Climate personnel.*

```
git clone https://geusgitlab.geus.dk/glaciology-and-climate/promice/aws-l0
```

And to grab the latest copy of this data, use `git pull` after navigating to the repo's local directory
 
```
git pull
```


### AWS L3 Gitlab repo

This is the Gitlab repo space where PROMICE and GC-Net AWS Level 3, `L3`, processed data is held. You can make a local copy of this repo using `git clone`. *This is only open to GEUS Glaciology and Climate personnel.*

```
git clone https://geusgitlab.geus.dk/glaciology-and-climate/promice/aws-l3
```

And to grab the latest copy of this data, use `git pull` after navigating to the repo's local directory
 
```
git pull
```


### pypromice

The pypromice toolbox is for retrieving, processing and handling PROMICE datasets. To run pypromice, some packages are required so you need to make a new Python 3.8 environment and install these packages. We typically make a Python environment and install these packages using conda.

```
conda create --name py38 python=3.8
conda activate py38
conda install xarray pandas pathlib
pip install netCDF4
```

Then clone the pypromice Github repo and perform a local pip install.

```
git clone https://github.com/GEUS-Glaciology-and-Climate/pypromice.git
cd pypromice
pip install .
```

Once the pypromice toolbox is cloned and installed, the toolbox can be checked with its in-built unittesting:

```
python -m unittest tx.py aws.py get.py
```

Note that for WMO data ingestion development, [eccodes](https://confluence.ecmwf.int/display/ECC/ecCodes+installation), the official package for BUFR encoding and decoding, is needed to perform the BUFR formatting. This can be a tricky installation, so first try to install it with conda-forge:

```
conda install -c conda-forge eccodes
```

If the environment cannot resolve the eccodes installation then follow the steps documented [here](https://gist.github.com/MHBalsmeier/a01ad4e07ecf467c90fad2ac7719844a) to download eccodes, and then install eccodes' python bindings using pip:

```
pip3 install eccodes-python
```


### Credentials for email access

Make a directory called `credentials` and place the following three files in this space:
1. `accounts.ini` - contains the account information for retrieving TX messages
2. `credentials.ini` - contains the account password
3. `last_aws_uid.ini` - contains a number which indicates the id of the last read `tx` message from email

*Account and credential information is for GEUS Glaciology and Climate personnel only and should never be shared with anyone else.* Please ask the data scientist team for access to these files.


### L3 processing bash scripts

One bash script is used to perform:
1. Check for remote changes to the `L0` and `L3` Gitlab repositories
2. `L0 tx` message retrieval
3. `L3 raw` processing (if any changes are detected in the L0 RAW files
4. `L3 tx` processing (for files where new L0 TX messages are present)
5. `L3 raw` and `L3 tx` data merging
6. Push changes to the `L0` and `L3` Gitlab repositories
If you don't want to perform one of these steps then please comment these sections out.

This bash script can run routinely using `cron`, which can be accessed using `crontab -e`. 

```
0 */1 * * * . /home/USR/.bashrc; cd /mnt/data/USR/pypromice_aws; ./l3_processor.sh > stdout 2>stderr

```

In our department, `l3_processor.sh` is set to run once an hour. If you intend to set-up your own operational processing then just make sure to update the directories (signified by the `USR` string).


#### l3_processor.sh

```
#!/usr/bin/env bash

source /home/USR/miniconda3/etc/profile.d/conda.sh
conda activate py38
echo "Running l3_processor at `/bin/date +%Y-%m-%d\ %T`"

echo 'Checking for updates on aws-l0...'
cd /mnt/data/USR/pypromice_aws/aws-l0
git checkout main
git pull

echo "Checking for updates on aws-l3"
cd /mnt/data/USR/pypromice_aws/aws-l3
git checkout main
git pull

#------------------

echo "Running AWS L0 raw >> L3 processing"

echo "Finding recently added files..."
cd /mnt/data/USR/pypromice_aws/aws-l0/raw/config
station_configs=$(find . -maxdepth 1 -type f -mmin -1440 | cut -d"/" -f2)
echo ${station_configs}

if [[ -n "$station_configs" ]]
then
    echo "Processing L3..."
    parallel --bar "getL3 -v ../../../pypromice/src/pypromice/variables.csv -m ../../../pypromice/src/pypromice/metadata.csv -c ./{} -i ../ -o ../../../aws-l3/raw" ::: $(ls $station_configs) 

else 
    echo "No new raw updates found"
fi

echo "Finished AWS L0 raw >> L3 processing"

#--------------------

echo "Running AWS TX >> L0 >> L3 processing"

echo "Fetching new L0 TX messages..."
cd /mnt/data/USR/pypromice_aws/aws-l0
getL0tx -a ../credentials/accounts.ini -p ../credentials/credentials.ini -c tx/config -f ../pypromice/src/pypromice/payload_formats.csv -t ../pypromice/src/pypromice/payload_types.csv -u ../credentials/last_aws_uid.ini -o tx

echo "Finding recently fetched files..."
cd /mnt/data/pypromice_aws/aws-l0/tx
IMEIs=$(find . -maxdepth 1 -type f -mmin -60 | cut -d"/" -f2)
imei_list=""
for i in $(echo $IMEIs | tr ' ' '\n'); do
  imei_list=$(echo $imei_list; grep -l $i config/*.toml)	
done
echo ${imei_list}   

if [[ -n "$imei_list" ]]
then
    echo "Processing L3..."
    parallel --bar "getL3 -v ../../pypromice/src/pypromice/variables.csv -m ../../pypromice/src/pypromice/metadata.csv -c ./{} -i . -o ../../aws-l3/tx" ::: $(ls $imei_list)

else 
    echo "No new TX updates found"
fi

echo "Finished AWS L0 raw >> L3 processing"

#-------------

echo "Running AWS L3 RAW and TX joiner"
cd /mnt/data/pypromice_aws/aws-l3

echo "Finding all unique AWS data by name..."
names=$(find . -maxdepth 3 -type f -name "*.csv" | cut -d"/" -f3 | sort | uniq)
echo $names

echo "Joining L3 data..."
parallel --bar --xapply ' ' "joinL3 -s ./raw/{}/{}_10min.csv -t ./tx/{}/{}_hour.csv -o level_3 -v ../pypromice/src/pypromice/variables.csv -m ../pypromice/src/pypromice/metadata.csv -d raw" ::: $names

#--------------

echo "Pushing new L0 data to aws-l0..."
cd /mnt/data/pypromice_aws/aws-l0
git add *
git commit -m "L0 update `/bin/date +%Y-%m-%d\ %T`"
git push

echo "Pushing new L3 products to aws-l3..."
cd /mnt/data/pypromice_aws/aws-l3
git add *
git commit -m "L3 update `/bin/date +%Y-%m-%d\ %T`"
git push

echo "Finished at `/bin/date +%Y-%m-%d\ %T`"
```
