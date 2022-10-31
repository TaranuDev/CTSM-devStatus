---
title: "clm_driver.F90"
tags:
- CTSM
- Main
enableToc: false # do not show a table of contents on this page
---


## General discussion:
This module provides the main CTSM/CLM driver physics calling sequence.  Most computations occurs over `clumps` of gridcells (and associated subgrid scale entities) assigned to each MPI process. Computation is further parallelized by looping over clumps on each process using shared memory OpenMP.

We will not go in detail over the entire code, but the reader is invited to do so if curious on the corresponding [github page](https://github.com/ESCOMP/CTSM/blob/master/src/main/clm_driver.F90). Despite this, there are 2 important things to notice:
	1. When it comes to the hydrology part in the driver loop, it is important to ask ourselves when the human water abstractions should happen? The way I see it at the moment, is that sectoral water usage should be seen as a forcing, similar to precipitation, and therefore should be "done or requested" before solving for any emerging hydrological process, as canopy interception or snow formation.
	2. As mentioned before in [Model Development](Model_Development_for_Sectoral_Water_Usage.md), at the moment when we started working on sectoral water abstractions, irrigation was already implemented (since 2013 in fact). At the moment we also do not plan to change the irrigation module. Therefore, if we want the sectoral competition algorithm to work properly, we had to call the subroutine related to sectoral abstractions before the irrigation one, since in our current implementation, irrigation is supposed to be last in priority.

## What changed:

To run sectoral water usage in the driver sequence, we make a single call to the subroutine: `CalcAndWithdrawSectorWaterFluxes()` which is defined in the [CTSM/src/biogeophys/HydrologyNoDrainageMod.F90](CTSM/HydrologyNoDrainageMod.md) module. 

This subroutine do everything required for sectoral water usage functioning from land model perspective: 
- compute the withdrawal and consumption for the day (based on monthly values from input data assuming uniform distribution across the days);
- compute actual withdrawal and consumption by accounting for available river water;
- compute actual return flow (actual withdrawal - actual consumption);
- update the water instance object which contains the fields (actual withdrawal and return flow) which is accessed then by the river model MOSART through the coupler and mediator;
- apply the consumption flow on surface soil over the subgrid column covered with natural vegetation;

The call is done before `CalcAndWithdrawIrrigationFluxes()` subroutine. The order is important, since in our current implementation, the irrigation has the least priority between sectors to satisfy water requirements.

In addition to the new subroutine related to human water use, we also changed one of the inputs of the subroutine related to irrigation (see [Irrigation in CTSM](Irrigation/irrig2013.md)).  In order to establish how much water can actually be provided for irrigation (upper limit), the `CalcIrrigationNeeded()` subroutine will receive as input the variable `volr` corresponding to the volume of water in the entire river network in the corresponding gridcell (measured in m3).  But because human water abstractions for the sectors other than irrigation result in a total negative balance in terms of river volume (because withdrawal is always higher than return flow), then the `volr` quantity received by the irrigation subroutine need to be updated at the level of the driving sequence as `volr - this_day_total_sectoral_withdrawal + this_day_total_sectoral_return_flow`. This way, on a given day, the irrigation will not consume more water, than actually available after all other sectors were satisfied. 


## Possible improvements:

At the moment, for all the sectors except irrigation, water is supplied only from the surface water (rivers).

At the same time, in theory, CTSM already supports water withdrawal from unconfined groundwater (only for irrigation). Therefore one of the things we intend to add is the possibility to partially satisfy all sectoral demand from groundwater.

The way this is done at the moment in CTSM:
1. We satisfy as much as possible from surface water.
2. We satisfy the rest from unconfined groundwater

This strategy do not seem optimal, since it may underestimate groundwater withdrawal for regions heavily relying on this source. So in the future we may think of new ways to improve groundwater abstraction algorithm, but also add support for groundwater usage for all sectors.



