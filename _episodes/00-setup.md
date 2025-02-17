---
title: "Set up the Jupyter Notebook Environment"
teaching: 10
exercises: 5
questions:
- "How to access simulation campaign output with python?"
objectives:
- "Set up interactive python platform with Jupyter-notebook."
- "Access the simulation campaign output with `uproot`."
---
## Set up Jupyter-notebook
- on Google Colab (recommended):
  - copy [this notebook](https://colab.research.google.com/drive/1Wn9guq1aIJ8RUW36HHTkeR7-iPPcoBOw?usp=sharing) to your own google drive.
  - run the first cell under "setup" to install `uproot` and `xrootd` (takes ~10 minutes)
- on your local environment:
  - if not yet, install `uproot` and `xrootd` with your local package manager, __or__, in jupyter-notebook, run 
```console
    !pip install xrootd
    !pip install uproot
    !pip install fsspec-xrootd
    !pip install particle ## optional
```

> Note: we will be using other standard python packages such as `numpy` and `pandas`, which are pre-installed on Colab. 
{: .callout}


## Access simulation campaign output 
> The simulation campaign [website](https://eic.github.io/epic-prod/documentation/default_datasets.html) documents the available datasets and version information.
> 
> To browse the directory and download rootfiles with stand-alone `xrdfs` command, see the [previous tutorials](https://eic.github.io/tutorial-analysis/01-introduction/index.html). Here we will proceed with the python interface:
```console
from XRootD import client
# Create XRootD client
eic_server = 'root://dtn-eic.jlab.org/'
fs = client.FileSystem(eic_server)
# List directory contents
fpath      = '/work/eic2/EPIC/RECO/24.10.0/epic_craterlake/DIS/NC/18x275/minQ2=10/'
status, files = fs.dirlist(fpath)
# Print files
if status.ok:
  print(files.size)
    for entry in files:
        print(entry.name)
else:
    print(f"Error: {status.message}")
```

> To open a simulation campaign file
```console
    fname      = eic_server+fpath+'pythia8NCDIS_18x275_minQ2=10_beamEffects_xAngle=-0.025_hiDiv_1.0000.eicrecon.tree.edm4eic.root'
    tree_name  = "events" #"podio_metadata"
    tree       = ur.open(fname)[tree_name]
    print(f"Read {fname}:{tree_name}. \n {tree.num_entries} events in total")
```


> Exercise 1: open a local rootfile
> -  open a local rootfile of your choice
> -  use ```tree.keys(filter_name="*",recursive=False)``` to display all branches
{: .challenge}


{% include links.md %}
