---
title: "Introduction to the simulation workflow"
teaching: 15
exercises: 5
questions:
- "How the simulation output is generated"
objectives:
- "Event generation"
- "DD4hep simulation"
- "EICrecon Reconstruction"
keypoints:
- "Point 1 -"
- "Point 2 -"
- "Point 3 -"
---

## Event generation
> The collision event at ePIC, including the beam particles, vertices, and outgoing particles, are typically generated with a dedicated event generator, e.g. PYTHIA8 for specific physics channels. The results are provided to the DD4hep simulation in [HEPMC3 format](https://arxiv.org/pdf/1912.08005).
>
> One can also use the DD4hep particle gun to generate outgoing particles with given vertex and distribution.
>
## DD4hep simulation
> This Geant4-based simualtion package propagate particles through magnetic field and materials. See [previous tutorial](https://eic.github.io/tutorial-simulations-using-ddsim-and-geant4/aio/index.html) on running the simulation.
> 
> The ePIC detector description in DD4hep is maintained [on github](https://github.com/eic/epic/). In each sub-detector file under `epic/compact`, the `readout` block on the bottom specifies the detector branch name in sismulation output.   
>
> 

{% include links.md %}
