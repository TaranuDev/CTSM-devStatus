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
	2. As mentioned before in [Model Development](Model_Development_for_Sectoral_Water_Usage.md), at the moment when we started working on sectoral water abstractions, irrigation was already implemented (since 2013 in fact). At the moment we also do not plan to change the irrigation module. Therefore, if we want the sectoral competition algorithm to work properly, we had to call the subroutine related to sectoral abstractions just before the irrigation one, since in our current implimentation, irrigation is supposed to be last in priority.


In the case of sectoral water usage, we will have only one call to the subroutine: `CalcAndWithdrawSectorWaterFluxes()` which is defined in the [CTSM/src/biogeophys/HydrologyNoDrainageMod.F90](Documentation/CTSM/HydrologyNoDrainageMod.md) module. 

This subroutine do everything required for sectoral water usage implementation: 
- compute the withdrawal and consumption for the day (monthly value from input data devided by number of days in a month);
- compute actual withdrawal and consumption by accounting for available river water;
- compute actual return flow (actual withdrawal - actual consumption);
- update the water instance object which contains the fields (actual withdrawal and return flow) to be send through the coupler so it gets to the river model

The call is done before CalcAndWithdrawIrrigationFluxes() subroutine. The order is important, since irrigation has the least priority between sectors to satisfy water requirements.


## Things which may require change:
### Accounting for sectoral priority when limited water resources
Calling `CalcAndWithdrawSectorWaterFluxes()` before `CalcAndWithdrawIrrigationFluxes()` is done because if water is limited we would like to have the following priority for water usage: domestic (highest), livestock, thermoelectric, manufacturing, mining, irrigation (lowest). But at the moment this is not really done, as I don't update the `volr` (avaialable river water) after withdrawing for the sectors. I think one way to do this is to maybe change the input of `irrigation_inst%CalcIrrigationNeeded()` with instead of giving the `volr`, provide `volr - [total actual withdrawal - total actual return flow]`, where `total actual withdrawal / return flow` is the sum of the contributions of the 5 sectors (domestic, livestock, thermoelectric, manufacturing and mining). We can think of other solution too, but in general something in this spirit.

### How to deal with consumpted water?
Also at the moment, we don't do anything about the consumed water. But the plan is to add it to the surface soil layer over natural vegetation and pervious roads columns. The choice of columns over which we will add the consumed water may change, but in any case we should avoid to place this water on the irrigated land (to not interfere with irrigation process).

The application of consumed water should be distributed proportionally (for example if in one grid there is x=20% of pervious roads, and y=25% of natural vegetation, and other land is something else, then all consumed water should be distribute proportionally between x and y. Since y/x=1.25, the pervious roads will receive 100/(1+1.25) => 44.45% of consumption and the natural vegetation the 55.55% remaining.

### Usage of groundwater as a source for sectoral fluxes
At the moment, water is taken only from the surface water (rivers).
The CTSM model already supports water withdrawal from unconfined groundwater (already available for the irrigation). Therefore one of the things we intend to add is partially satisfy sectoral demand from groundwater.

The way this is done at the moment in CTSM:
1. We satisfy as much as possible from surface water.
2. We satisfy the rest from unconfined groundwater (if not enough take as much as available)

This strategy do not seem optimal, since it may underestimate groundwater withdrawal for regions havily relying on  this source. So in the future we may wish to change this.



