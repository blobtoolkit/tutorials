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
mamba create -y -n btk_env -c conda-forge -c bioconda -c tolkit \
    python=3.8 snakemake docopt defusedxml psutil pyyaml tqdm ujson urllib3 \
    entrez-direct minimap2=2.17 seqtk diamond=2 busco=5 \
    samtools=1.10 pysam=0.16 mosdepth=0.2.9 tolkein
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

conda activate btk_env
mamba install -y -c conda-forge geckodriver selenium pyvirtualdisplay nodejs=10
pip install fastjsonschema;
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
curl ftp://ftp.ed.ac.uk/incoming/CADCXM01.1.tar.gz | tar xzf -
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
curl ftp://ftp.ed.ac.uk/incoming/JACYVU01.tar.gz | tar xzf -
curl ftp://ftp.ed.ac.uk/incoming/CAJEUD01.tar.gz | tar xzf -

blobtools view --remote ~/blobtoolkit/blobdirs
```

And follow the same instructions as above. Go to http://localhost:8001/view/all to see all three datasets.