# 2021-04-28 blobtoolkit commandline tutorial for Mollusc Genomics Community

## Key concepts:

- A *BlobDir* can be loaded and visualised in the *Viewer*
- The *blobtools* command creates a *BlobDir* from *intermediate files* such as blast results, read mapping files, etc.
- A *snakemake pipeline* runs all the intermediate steps needed to create the *intermediate files* and the final *BlobDir*

## Install all BlobToolKit dependencies

These sections are identical to https://github.com/blobtoolkit/pipeline#dependencies

### BlobToolKit components

The various BlobToolKit components can be downloaded from their respecive Github repositories:

```
VERSION=release/v2.5.0
mkdir -p ~/blobtoolkit
cd ~/blobtoolkit
git clone -b $VERSION https://github.com/blobtoolkit/blobtools2
git clone -b $VERSION https://github.com/blobtoolkit/specification
git clone -b $VERSION https://github.com/blobtoolkit/pipeline
git clone -b $VERSION https://github.com/blobtoolkit/viewer
```
### Pipeline dependencies

Most pipeline dependencies can be installed using conda. The mamba replacement for conda is faster and more stable:

```
conda install -y -n base -c conda-forge mamba
```
Use mamba where you would normally use conda for creating environments and installing packages:

```
mamba env create -f https://raw.githubusercontent.com/blobtoolkit/blobtoolkit-docker/develop/env.yaml
```
Activate this environment:
```
conda activate btk_env
```

### Additional viewer dependencies

If these are not installable on your local compute environment (e.g. a shared cluster), it may be necessary to run the viewer steps separately:

```
sudo apt update && sudo apt-get -y install firefox xvfb
# apt-get install instruction is only for ubuntu linux. Use the equivalent for other linux distros
```

Now set up the viewer:
```
cd ~/blobtoolkit/viewer
npm install
```

### PATH setup
The commands below assume that BLobToolKit executables and scripts are available in you PATH:
```
export PATH=~/blobtoolkit/blobtools2:~/blobtoolkit/specification:~/blobtoolkit/insdc-pipeline/scripts:$PATH
```

## Run a viewer for a BlobDir

Create a new directory and download a pre-made BlobDir:
```
mkdir -p ~/blobtoolkit/blobdirs
cd ~/blobtoolkit/blobdirs
wget -O CADCXM01.1.tar.gz "https://github.com/blobtoolkit/tutorials/blob/main/CADCXM01.1.tar.gz?raw=true"
tar xzf CADCXM01.1.tar.gz
```
Run the blobtools view command:
```
blobtools view --remote ~/blobtoolkit/blobdirs
```

If you are running it on the same machine where you have a browser (eg a laptop or personal desktop) then just open a browser and go to http://localhost:8001/view/all (the port 8001 might change if it was not free when you ran this command, so just follow the instructions on your screen)

If you are running `blobtools view` on a server, you should:

1. Leave that command running in a terminal
2. Open a new terminal and connect using the ssh command provided which looks like `ssh -L 8001:127.0.0.1:8001 -L 8000:127.0.0.1:8000 username@remote_host`. The actual port numbers might change depending on which ones were free when you ran the `blobtools view` command
3. Open a browser and go to the URL suggested, typically http://localhost:8001/view/all

Now, download some more BlobDirs:
```
# kill the existing blobtools view command first by pressing Ctrl+c
cd ~/blobtoolkit/blobdirs
wget -O JACYVU01.tar.gz "https://github.com/blobtoolkit/tutorials/blob/main/JACYVU01.tar.gz?raw=true"
wget -O CAJEUD01.tar.gz "https://github.com/blobtoolkit/tutorials/blob/main/CAJEUD01.tar.gz?raw=true"
tar xzf JACYVU01.tar.gz
tar xzf CAJEUD01.tar.gz

blobtools view --remote ~/blobtoolkit/blobdirs
```

And follow the same instructions as above. Go to http://localhost:8001/view/all to see all three datasets.

## Run blobtools to create a BlobDir

### Set up databases
Normally you would create / download the full databases using the steps in https://github.com/blobtoolkit/pipeline#databases

For this demo, download these files (~1GB)
```
cd ~/blobtoolkit
wget ftp://ftp.ed.ac.uk/incoming/demodatabases.tar.gz
wget ftp://ftp.ed.ac.uk/incoming/demodata.tar.gz
tar xzf demodatabases.tar.gz
tar xzf demodata.tar.gz
```
We'll first see what files already exist:
```
ls -al
```

To run the pipeline, set some BASH environment variable names:
```
DATA_DIR=~/blobtoolkit/data/
SNAKE_DIR=~/blobtoolkit/pipeline
THREADS=16
ACCESSION=
```
Now run any of the steps in https://github.com/blobtoolkit/pipeline#sub-pipelines

If the steps have already been run, snakemake will report "Nothing to do":
```
TOOL=minimap
snakemake -p \
          -j $THREADS \
          --directory $DATA_DIR/$ACCESSION/$TOOL \
          --configfile $DATA_DIR/$ACCESSION/config.yaml \
          --latency-wait 60 \
          --stats $DATA_DIR/$ACCESSION/$TOOL.stats \
          -s $SNAKE_DIR/$TOOL.smk

TOOL=windowmasker
snakemake -p \
          -j $THREADS \
          --directory $DATA_DIR/$ACCESSION/$TOOL \
          --configfile $DATA_DIR/$ACCESSION/config.yaml \
          --latency-wait 60 \
          --stats $DATA_DIR/$ACCESSION/$TOOL.stats \
          -s $SNAKE_DIR/$TOOL.smk

TOOL=blobtoolkit
snakemake -p \
          -j $THREADS \
          --directory $DATA_DIR/$ACCESSION/$TOOL \
          --configfile $DATA_DIR/$ACCESSION/config.yaml \
          --latency-wait 60 \
          --stats $DATA_DIR/$ACCESSION/$TOOL.stats \
          -s $SNAKE_DIR/$TOOL.smk
```

