Tags: #CTSM #Namelist 

## Overview:
To support sectoral water usage we add the relevant namelist entries.

There are 4 namelist groups which are affected by our changes.
1. `sectorwater_inparm` which is defined in the new module [[SectorWaterMod.F90]]. This namelist group have the following entries: `dom_and_liv_start_time`,  `ind_start_time`,  `dom_and_liv_length`, `ind_length`, `sectorwater_river_volume_threshold`, `limit_sectorwater_if_rof_enabled`,  `use_groundwater_sectorwater` .
2. `clm_inparm` which is defined in [[clm_varctl.F90]]. In this namelist group only one entry is added `sectorwater`
3. `default_settings` where we add `mksrf_fsectorwater` as valid value.
4. `clmexp` where we add `mksrf_fsectorwater` to the mksurfdata category.