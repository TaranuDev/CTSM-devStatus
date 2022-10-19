---
title: "clm_driver.F90"
tags:
- CTSM
- Main
enableToc: false # do not show a table of contents on this page
---


## Overview:
This module provides the main CLM driver physics calling sequence.  Most computations occurs over `clumps` of gridcells (and associated subgrid  scale entities) assigned to each MPI process. 

Computation is further parallelized by looping over clumps on each process using shared memory OpenMP. 

For irrigation there are two calls done in this module: CalcAndWithdrawIrrigationFluxes() and irrigation_inst%CalcIrrigationNeeded()

In the case of sectoral water usage, we will have only one call to the subroutine: `CalcAndWithdrawSectorWaterFluxes()` which is defined in the [CTSM/src/biogeophys/HydrologyNoDrainageMod.F90](Documentation/CTSM/HydrologyNoDrainageMod.md) module. 

This subroutine do everything required for sectoral water usage implementation: 
- compute the withdrawal and consumption for the day (monthly value from input data devided by number of days in a month);
- compute actual withdrawal and consumption by accounting for available river water;
- compute actual return flow (actual withdrawal - actual consumption);
- update the water instance object which contains the fields (actual withdrawal and return flow) to be send through the coupler so it gets to the river model

The call is done before CalcAndWithdrawIrrigationFluxes() subroutine. The order is important, since irrigation has the least priority between sectors to satisfy water requirements.


## Code:
```fortran
...
subroutine clm_drv(doalb, nextsw_cday, declinp1, declin, rstwr, nlend, rdate, rof_prognostic)

...
if (sectorwater) then

	call t_startf('sectorwater_calc_and_withdraw')

	call CalcAndWithdrawSectorWaterFluxes( &
               bounds = bounds_clump, &
               soilhydrology_inst = soilhydrology_inst, &
               sectorwater_inst = sectorwater_inst, &
               water_inst = water_inst, &
               volr       = water_inst%wateratm2lndbulk_inst%volrmch_grc(bounds_clump%begg:bounds_clump%endg), &
               rof_prognostic = rof_prognostic)

	call t_stopf('sectorwater_calc_and_withdraw')

end if

...

```



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



