---
title: "Introduction to the DD4hep simulation"
teaching: 25
exercises: 15
---

::::::::::::::::::::::::::::::::::::::::::::: questions

- Understand the inputs and outputs of the DD4hep simulation

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: objectives

- Event generation
- Detector description
- MCParticles and detector hits
- Simulation campaign files

:::::::::::::::::::::::::::::::::::::::::::::

## DD4hep simulation

This Geant4-based simulation package propagates particles through magnetic field and materials. Particles and detector hits for each event are saved in the output rootfiles.

### __Input 1: Event generation__

The collision event at ePIC, including the beam particles, vertices, and outgoing particles, are typically generated with a dedicated event generator, e.g. PYTHIA8 for specific physics channels. The outputs are provided to the DD4hep simulation in [HEPMC3 format](https://arxiv.org/pdf/1912.08005).

One can also use the DD4hep's particle gun to generate outgoing particles with given vertex and distribution, see the [Simulations Using npsim and Geant4 tutorial](https://eic.github.io/tutorial-simulations-using-npsim-and-geant4/) on `ddsim`.

### __Input 2: Detector description__

The ePIC detector description in DD4hep is maintained in the [eic/epic repository on GitHub](https://github.com/eic/epic/). On the bottom of each sub-detector compact file under `epic/compact`, the `readout` block specifies how the detector hits are saved in the output rootfile.

Below is an example from `epic/compact/tracking/vertex_barrel.xml`:

```xml
<readouts>
  <readout name="VertexBarrelHits">
    <segmentation type="CartesianGridXY" grid_size_x="0.020*mm" grid_size_y="0.020*mm" />
    <id>system:8,layer:4,module:12,sensor:2,x:32:-16,y:-16</id>
  </readout>
</readouts>
```

All hits from this silicon vertex barrel detector, including their position, energy deposit, time, will be stored under the branch `VertexBarrelHits` in output.
Each detector hit also comes with an assigned 64-bit cell ID, with the last 32 bits from right to left representing the hit location in a 0.020 x 0.020 mm mesh grid. This __segmentation__ often represents the detector granularity (in this case, the silicon pixel sensor size) that will be used later for hit digitization.

### __Output__

The `event` tree in the simulation output contains

- `MCParticles`: records the truth info of primary and secondary particles
- Individual branches for signals from various sub-detector systems e.g. `VertexBarrelHits`

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 1.1: access simulation campaign rootfiles

The simulation campaign [dataset documentation](https://eic.github.io/epic-prod/documentation/default_datasets.html) documents the available datasets and version information.

- __Browse the directory__

  For the stand-alone `xrdfs` command, see the [previous Analysis tutorial](https://eic.github.io/tutorial-analysis/). Here we will proceed with the python interface:

  ```python
  from XRootD import client
  # Create XRootD client
  eic_server = 'root://dtn-eic.jlab.org/'
  fs = client.FileSystem(eic_server)
  # List directory contents
  fpath      = '/volatile/eic/EPIC/RECO/26.02.0/epic_craterlake/SINGLE/e-/10GeV/130to177deg/'
  status, files = fs.dirlist(fpath)
  # Print files
  if status.ok:
      print(files.size)
      for entry in files:
          print(entry.name)
  else:
      print(f"Error: {status.message}")
  ```

- __Open a simulation campaign file__

  ```python
  fname      = eic_server+fpath+'e-_10GeV_130to177deg.0000.eicrecon.edm4eic.root'
  tree_name  = "events"
  # tree_name = "podio_metadata"
  tree       = ur.open(fname)[tree_name]
  print(f"Read {fname}:{tree_name}. \n {tree.num_entries} events in total")
  ```

::::::::::::::: solution

`fs.dirlist` returns the list of files available in the campaign directory, and `ur.open(...)[tree_name]`
opens the `events` tree and reports the number of entries. If the directory listing fails, check that
the campaign version in `fpath` still exists on the server (campaigns roll over and older versions are
removed).

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 1.2: inspect available branches in a rootfile

- use `tree.keys(filter_name="*",recursive=False)` to display all branches
- extract a given branch to dataframe

```python
bname = "MCParticles"
df    = tree[bname].array(library="ak")
df    = ak.to_dataframe(df)
print(df)
```

::::::::::::::: solution

`tree.keys(...)` lists the top-level branches (e.g. `MCParticles` and the per-detector hit
collections). Reading `MCParticles` into an awkward array and converting to a dataframe gives one row
per particle, with columns such as `MCParticles.PDG`, `MCParticles.generatorStatus`, and the momentum
components.

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 1.3: extract momentum distribution of primary electrons

```python
# select electrons
from particle import Particle
part   = Particle.from_name("e-")
pdg_id = part.pdgid.abspid
condition1  = df["MCParticles.PDG"]==pdg_id
# select primary particles
condition2  = df["MCParticles.generatorStatus"]==1
# extract momentum and plot
# all electrons
df_new = df[condition1]
mom    = np.sqrt(df_new["MCParticles.momentum.x"]**2+df_new["MCParticles.momentum.y"]**2+df_new["MCParticles.momentum.z"]**2)
bins   = np.arange(0,20)
_      = plt.hist(mom,bins=bins,alpha=0.5)
# primary electrons
df_new = df[condition1&condition2]
mom    = np.sqrt(df_new["MCParticles.momentum.x"]**2+df_new["MCParticles.momentum.y"]**2+df_new["MCParticles.momentum.z"]**2)
_      = plt.hist(mom,bins=bins,histtype="step", color='r')
```

::::::::::::::: solution

The first histogram (filled) shows the momentum of all electrons, including secondaries; the second
(red outline), obtained by additionally requiring `generatorStatus==1`, shows only the primary
electrons. The primary distribution is a subset of the all-electron distribution, peaked near the
generated beam/scattered-electron momentum.

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: keypoints

- event generator --`dd4hep`--> simulated hits and particles

:::::::::::::::::::::::::::::::::::::::::::::
