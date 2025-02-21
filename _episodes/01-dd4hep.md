---
title: "Introduction to the DD4hep simulation"
teaching: 25
exercises: 15
questions:
- "Understand the inputs and outputs of the DD4hep simulation"
objectives:
- "Event generation"
- "Detector description"
- "MCParticles and detector hits"
- "Simulation campaign files"
keypoints:
- "event generator --`dd4hep`--> simulated hits and particles"
---


## DD4hep simulation
This Geant4-based simualtion package propagates particles through magnetic field and materials. Particles and detector hits for each event are saved in the output rootfiles.

### __Input 1: Event generation__
  
  The collision event at ePIC, including the beam particles, vertices, and outgoing particles, are typically generated with a dedicated event generator, e.g. PYTHIA8 for specific physics channels. The outputs are provided to the DD4hep simulation in [HEPMC3 format](https://arxiv.org/pdf/1912.08005).

  One can also use the DD4hep's particle gun to generate outgoing particles with given vertex and distribution, see the [previous tutorial](https://eic.github.io/tutorial-simulations-using-ddsim-and-geant4/aio/index.html) on `ddsim`.


### __Input 2: Detector description__

  The ePIC detector description in DD4hep is maintained [on github](https://github.com/eic/epic/). On the bottom of each sub-detector compact file under `epic/compact`, the `readout` block specifies how the detector hits are saved in the output rootfile.

  Below is an example from epic/compact/tracking/vertex_barrel.xml:

```console
  <readouts>
    <readout name="VertexBarrelHits">
      <segmentation type="CartesianGridXY" grid_size_x="0.020*mm" grid_size_y="0.020*mm" />
      <id>system:8,layer:4,module:12,sensor:2,x:32:-16,y:-16</id>
    </readout>
  </readouts>
```
  All hits from this silicon vertex barrel detector, including their position, energy deposit, time, will be stored under the branch `VertexBarrelHits` in output.
  Each detector hit also comes an assigned 64-bit cell ID, with the last 32 bits from right to left represents the hit localtion in a 0.020 x 0.020 mm mesh grid. This __segmentation__ often represents the detector granularity (in this case, the silicon pixel sensor size) that will be used later for hit digitization.

### __Output__
  The `event` tree in the simulation output contains
  - `MCParticles`: records the truth info of primary and secondary particles
  -  Individual branches for signals from various sub-detector systems e.g. `VertexBarrelHits`

> Exercise 1.1: access simulation campaign rootfiles 
> The simulation campaign [website](https://eic.github.io/epic-prod/documentation/default_datasets.html) documents the available datasets and version information.
>
> -__Browse the directory__ 
  > For stand-alone `xrdfs` command, see the [previous tutorials](https://eic.github.io/tutorial-analysis/01-introduction/index.html). Here we will proceed with the python interface:
  ```console
from XRootD import client
# Create XRootD client
eic_server = 'root://dtn-eic.jlab.org/'
fs = client.FileSystem(eic_server)
# List directory contents
fpath      = '/work/eic2/EPIC/RECO/24.12.0/epic_craterlake/SINGLE/e-/10GeV/130to177deg/'
status, files = fs.dirlist(fpath)
# Print files
if status.ok:
  print(files.size)
    for entry in files:
        print(entry.name)
else:
    print(f"Error: {status.message}")
  ```
> -__Open a simulation campaign file__
  ```console
fname      = eic_server+fpath+'e-_10GeV_130to177deg.0307.eicrecon.tree.edm4eic.root'
tree_name  = "events" #"podio_metadata"
tree       = ur.open(fname)[tree_name]
print(f"Read {fname}:{tree_name}. \n {tree.num_entries} events in total")
  ```
{: .challenge}


> Exercise 1.2: inspect available branches in a rootfile
> -  use ```tree.keys(filter_name="*",recursive=False)``` to display all branches
> -  extract a given branch to dataframe
  ```console
bname = "MCParticles" 
df    = tree[bname].array(library="ak")
df    = ak.to_dataframe(df)
print(df)
  ```
{: .challenge}


> Exercise 1.3: extract momentum distribution of primary electrons
>
> Make selection
> ```console
  from particle import Particle
  part   = Particle.from_name("e-")
  pdg_id = part.pdgid.abspid
  condition1  = df["MCParticles.PDG"]==pdg_id # select electrons
  condition2  = df["MCParticles.generatorStatus"]==1  # select primary particles
 ```
> extract momentum
> ```console 
  df_new = df[condition1]   # all electrons
  mom    = np.sqrt(df_new["MCParticles.momentum.x"]**2+df_new["MCParticles.momentum.y"]**2+df_new["MCParticles.momentum.z"]**2)
  bins   = np.arange(0,20)
  _      = plt.hist(mom,bins=bins,alpha=0.5)

  df_new = df[condition1&condition2]   # primary electrons
  mom    = np.sqrt(df_new["MCParticles.momentum.x"]**2+df_new["MCParticles.momentum.y"]**2+df_new["MCParticles.momentum.z"]**2)
  _      = plt.hist(mom,bins=bins,histtype="step", color='r')
  ```
{: .challenge}

{% include links.md %}
