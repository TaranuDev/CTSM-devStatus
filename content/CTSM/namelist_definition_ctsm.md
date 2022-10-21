---
title: "namelist_definition_ctsm.xml"
tags:
- CTSM
- Namelist
enableToc: false # do not show a table of contents on this page
---

## General discussion:
When new parameterization is introduced in any Earth System Model, it is very common for the modeler to set some of parameters as user defined. Usually, these free parameters values can then be changed through a namelist file. Following this approach, the modeler and the users, can then investigate the model sensititivity to the new parameterization by changing the free parameters values and tune the parameterization for better results. Another application being full control over the represented processes and their complexity (e.g. by setting 'irrigate = .true. or .false' one can run the model with or withouth irrigation).

In this regard, CESM model is not an exception. For example the current release version of the land component of CESM, CLM5, have a total of 413 of such namelist variables ([follow link](https://www.cesm.ucar.edu/models/cesm2/settings/current/clm5_0_nml.html)).

In our development of human-water interface in CESM, we follow the same strategy, allowing the users to change the model functionality by providing a set of user defined namelist variables.

## New namelist variables:
To add support for the new development, we had to add some changes to 3 existing namelist groups, and add a new one specific only to the sectoral water abstractions:

1. `sectorwater_inparm`  is a new namelist group which is defined in the module [CTSM/src/biogeophys/SectorWaterMod.F90](CTSM/SectorWaterMod.md). This namelist group have the following entries: `dom_and_liv_start_time`,  `ind_start_time`,  `dom_and_liv_length`, `ind_length`, `sectorwater_river_volume_threshold`, `limit_sectorwater_if_rof_enabled`,  `use_groundwater_sectorwater` .
2. `clm_inparm` which is defined in [CTSM/src/main/clm_varctl.F90](Documentation/CTSM/clm_varctl.md). In this namelist group only one new variable is added `sectorwater`.
3. `default_settings` where we add `mksrf_fsectorwater`.
4. `clmexp` where we add `mksrf_fsectorwater` to the mksurfdata category.

For a more detailed description of all namelist variables, as well as a guide on how to run the CESM model with the new features, please go to the [Tutorial: how to run CESM with active human water abstractions](Tutorials/tutorial_run_new_model.md).

**Attention:** You will not be able to find the namelist variables presented here on the CESM website, because this development is not yet part of any release version.