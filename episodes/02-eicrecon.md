---
title: "Reconstruction workflow"
teaching: 25
exercises: 15
---

::::::::::::::::::::::::::::::::::::::::::::: questions

- Understand the EICrecon workflow

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: objectives

- Hit digitization
- data model
- Reconstruction algorithms

:::::::::::::::::::::::::::::::::::::::::::::

## Reconstruction workflow

The ePIC reconstruction framework `EICrecon` is maintained in the [eic/eicrecon repository on GitHub](https://github.com/eic/eicrecon/). It processes simulated hits from various detectors to reconstruct trajectory, PID, etc, and eventually reconstruct the simulated particle and physics observables at the vertex. In this section, we will use __track reconstruction__ as an example. Please refer to the [Lehigh reconstruction workfest presentations](https://indico.bnl.gov/event/20727/sessions/7433/#20240725) for the reconstruction workflow of other systems.

Each reconstruction step involves 3 components:

- the algorithm,
- the JOmniFactory where algorithm and data type are declared,
- the factory generator to execute the algorithm.

### __Digitization__

All simulated detector hits are digitized to reflect certain detector specs e.g. spatial and time resolution, and energy threshold. For example, the `VertexBarrelHits` from simulation are digitized through the `SiliconTrackerDigi` factory in `EICrecon/src/detectors/BVTX/BVTX.cc`:

```c++
    // Digitization
  app->Add(new JOmniFactoryGeneratorT<SiliconTrackerDigi_factory>(
      "SiBarrelVertexRawHits", // factory name
      {
        "VertexBarrelHits"  // input
      },
      {
        "SiBarrelVertexRawHits",  // outputs
        "SiBarrelVertexRawHitAssociations"
      },
      {
          .threshold = 0.54 * dd4hep::keV, // configurations
      },
      app
  ));
```

The actual algorithm locates at `EICrecon/src/algorithms/digi/SiliconTrackerDigi.cc`, with its input and output specified in `SiliconTrackerDigi_factory.h`:

```c++
  class SiliconTrackerDigi_factory : public JOmniFactory<SiliconTrackerDigi_factory, SiliconTrackerDigiConfig> {

  public:
      using AlgoT = eicrecon::SiliconTrackerDigi;
  private:
      std::unique_ptr<AlgoT> m_algo;

      PodioInput<edm4hep::SimTrackerHit> m_sim_hits_input {this};

      PodioOutput<edm4eic::RawTrackerHit> m_raw_hits_output {this};
      PodioOutput<edm4eic::MCRecoTrackerHitAssociation> m_assoc_output {this};

      ParameterRef<double> m_threshold {this, "threshold", config().threshold};
      ParameterRef<double> m_timeResolution {this, "timeResolution", config().timeResolution};
      ...
```

By comparing the two blocks of code above, we can see that the digitized hits, `SiBarrelVertexRawHits`, are stored in the data type `RawTrackerHit` that is defined in the [edm4eic data model](https://github.com/eic/EDM4eic/blob/main/edm4eic.yaml):

```yaml
edm4eic::RawTrackerHit:
    Description: "Raw (digitized) tracker hit"
    Author: "W. Armstrong, S. Joosten"
    Members:
      - uint64_t          cellID      // The detector specific (geometrical) cell id from segmentation
      - int32_t           charge
      - int32_t           timeStamp
```

In addition, the one-to-one relation between the sim hit and its digitized hit is stored as an `MCRecoTrackerHitAssociation` object:

```yaml
edm4eic::MCRecoTrackerHitAssociation:
    Description: "Association between a RawTrackerHit and a SimTrackerHit"
    Author: "C. Dilks, W. Deconinck"
    Members:
      - float                 weight        // weight of this association
    OneToOneRelations:
      - edm4eic::RawTrackerHit rawHit       // reference to the digitized hit
      - edm4hep::SimTrackerHit simHit       // reference to the simulated hit
```

which is filled in `SiliconTrackerDigi.cc`:

```c++
  auto hitassoc = associations->create();
  hitassoc.setWeight(1.0);
  hitassoc.setRawHit(item.second);
  hitassoc.setSimHit(sim_hit);
```

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 2.1

Please find other detector systems that use `SiliconTrackerDigi` for digitization.

::::::::::::::: solution

Search the detector setup files under `EICrecon/src/detectors/` for `SiliconTrackerDigi_factory`.
Besides the barrel vertex tracker (`BVTX`), the other silicon tracker subsystems (e.g. the silicon
barrel and endcap trackers) use the same digitization factory, each with its own input hit collection
and threshold configuration.

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

### __Track Reconstruction__

By default, we use the Combinatorial Kalman Filter from the ACTS library to handle track finding and fitting. This happens in the `CKFTracking` factory.

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 2.2

Please find the inputs and outputs of `CKFTracking`, and draw a flow chart from `CentralTrackerMeasurements` to `CentralTrackVertices`.

::::::::::::::: solution

Look up the `CKFTracking` factory in the tracking detector setup (`EICrecon/src/global/tracking/`).
Its input is the collection of 2D measurements (`CentralTrackerMeasurements`) and its outputs are the
track candidates/trajectories that are then turned into track parameters and, downstream, the vertices.
The flow chart follows the chain measurements → trajectories → track parameters → vertices.

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

### __Reconstruction output__

- __events__ tree:
  - `MCParticles` and detector sim hits are copied from simulation output to recon output
  - outputs from each step of recon algorithms must be either an `edm4hep` or `edm4eic` object if you want to save them in recon output
  - the default list of saved objects in recon output is defined in `EICrecon/src/services/io/podio/JEventProcessorPODIO.cc`. It can be configured on the command line.
- __podio_metadata__ tree:
  - `events___CollectionTypeInfo` provides a lookup table between output collection name and IDs (its `collectionID` and `name` members line up index-by-index; older files exposed this as `events___idTable`).

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 2.3

The __vector member__ or __relation__ of a given data collection is saved in a separate branch that starts with "_".

- Please use

  ```python
  tree.keys(filter_name="_CentralCKFTrajectories*", recursive=False)
  ```

  to list those members in `CentralCKFTrajectories`
- for a given event, the vector member `_CentralCKFTrajectories_measurementChi2` provides a list of chi2 for each measurement hit respectively. If multiple trajectories are found for one event, you can use `CentralCKFTrajectories.measurementChi2_begin` to locate the start index of a given trajectory (subentry).

::::::::::::::: solution

The `tree.keys(...)` call returns the branches prefixed with `_CentralCKFTrajectories_`, including
`_CentralCKFTrajectories_measurementChi2`. To read the chi2 values belonging to a particular
trajectory, use the `measurementChi2_begin`/`measurementChi2_end` indices of that trajectory as the
slice into the flat `_CentralCKFTrajectories_measurementChi2` vector.

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 2.4

`CentralTrackerMeasurements` saves all available space points for tracking as 2D measurements attached to representing surfaces.

- Please use the relation `_CentralTrackerMeasurements_hits` to trace back to the original detector hit collection (hint: use the collection ID lookup table), and obtain the 3D coordinate of the hit.

::::::::::::::: solution

Each entry of `_CentralTrackerMeasurements_hits` is an object ID holding a collection ID
(`_CentralTrackerMeasurements_hits.collectionID`) and an index (`_CentralTrackerMeasurements_hits.index`).
Use the `events___CollectionTypeInfo` branch in the `podio_metadata` tree to build a
`collectionID -> name` map (zip its `.collectionID` and `.name` members), map the collection ID back to
the original hit collection name, then index into that collection to read the hit's 3D position.

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

## What's next

- Generate your own simulation and reconstruction rootfiles: [Simulations Using npsim and Geant4 tutorial](https://eic.github.io/tutorial-simulations-using-npsim-and-geant4/)
- Contribute to reconstruction algorithms: [Reconstruction Algorithms tutorial](https://eic.github.io/tutorial-reconstruction-algorithms/)
- Develop analysis benchmarks: [Developing Benchmarks tutorial](https://eic.github.io/tutorial-developing-benchmarks/)

::::::::::::::::::::::::::::::::::::::::::::: keypoints

- simulated hits--`EICrecon`--> reconstructed quantities

:::::::::::::::::::::::::::::::::::::::::::::
