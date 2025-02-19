---
title: "Introduction to the simulation workflow"
teaching: 15
exercises: 5
questions:
- "Understand the inputs and outputs of the ePIC simulation"
objectives:
- "DD4hep simulation"
- "EICrecon Reconstruction"
keypoints:
- "event generator --`dd4hep`--> simulated hits and particles"
- "Simulated hits --`EICrecon`--> reconstructed particles and vertices"
---


## DD4hep simulation
> This Geant4-based simualtion package propagates particles through magnetic field and materials. Particles and detector hits for each event are saved in the output rootfiles.

- __Input 1: Event generation__
  > The collision event at ePIC, including the beam particles, vertices, and outgoing particles, are typically generated with a dedicated event generator, e.g. PYTHIA8 for specific physics channels. The outputs are provided to the DD4hep simulation in [HEPMC3 format](https://arxiv.org/pdf/1912.08005).

  > One can also use the DD4hep's particle gun to generate outgoing particles with given vertex and distribution, see the [previous tutorial](https://eic.github.io/tutorial-simulations-using-ddsim-and-geant4/aio/index.html) on ddsim.


- __Input 2: Detector description__
  > The ePIC detector description in DD4hep is maintained [on github](https://github.com/eic/epic/). In each sub-detector compact file under `epic/compact`, the `readout` block on the bottom specifies how the detector hits are saved in the output rootfile.

  > Below is an example from epic/compact/tracking/vertex_barrel.xml:

```console
  <readouts>
    <readout name="VertexBarrelHits">
      <segmentation type="CartesianGridXY" grid_size_x="0.020*mm" grid_size_y="0.020*mm" />
      <id>system:8,layer:4,module:12,sensor:2,x:32:-16,y:-16</id>
    </readout>
  </readouts>
```
  > All hits from this silicon vertex barrel detector, including their position, energy deposit, time, will be stored under the branch `VertexBarrelHits` in output.
  > Each detector hit also comes an assigned 64-bit cell ID, with the last 32 bits from right to left represents the hit localtion in a 0.020 x 0.020 mm mesh grid. This __segmentation__ often represents the detector granularity (in this case, the pixel sensor size) that will be used later for hit digitization.

- __Output__
  > The `event` tree in the simulation output contains
  - `MCParticles`: records the truth info of primary and secondary particles
  -  detector hits 

> Exercise 1: extract the initial momentum distribution from a simulation output
> - open a simulation campaign file for single particle simulation 
> - extract the MCParticles branch, and calculate momentum
{: .challenge}


{% include links.md %}
