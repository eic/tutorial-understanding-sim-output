---
title: "Set up the Jupyter Notebook Environment"
teaching: 5
exercises: 5
questions:
- "How to access simulation campaign output with python?"
objectives:
- "Set up interactive python platform with Jupyter-notebook."
- "Access simulation campaign output through xrootd."
- "Open rootfiles with python's uproot package."
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
    !pip install particle
```
> Note: we will be using other standard python packages such as `numpy` and `pandas`, which is pre-installed on Colab. 
{: .callout}


## Locate simulation campaign output 

## Access rootfiles with uproot
> Open a simulation campaign file


> Exercise 1: open a local rootfile
> -  tbd
> - 
{: .challenge}


{% include links.md %}
