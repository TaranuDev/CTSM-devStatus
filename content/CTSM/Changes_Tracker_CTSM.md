---
title: "Changes Tracker CTSM"
tags:
enableToc: false # do not show a table of contents on this page
---

## Here we track all modifications done to the land component CTSM/CLM in order to support sectoral water usage

The GitHub fork can be accessed [here](https://github.com/TaranuDev/CTSM)



## List of modifications:
### Namelist modifications
- [CTSM/bld/namelist_files/namelist_definition_ctsm.xml](CTSM/namelist_definition_ctsm.md)
- [CTSM/bld/namelist_files/namelist_defaults_ctsm_tools.xml](CTSM/namelist_defaults_ctsm_tools.md)
- [CTSM/bld/namelist_files/namelist_defaults_ctsm.xml](CTSM/namelist_defaults_ctsm.md)
- [CTSM/bld/CLMBuildNamelist.pm](CTSM/CLMBuildNamelist.md)

### Connect all forks using corresponding repositories and tags
- [Externals.cfg](CTSM/Externals.md)

### Changes to the main
- [CTSM/src/main/clm_driver.F90](CTSM/clm_driver.md)
- [CTSM/src/main/restFileMod.F90](CTSM/restFileMod.md)
- [CTSM/src/main/controlMod.F90](CTSM/controlMod.md)
- [CTSM/src/main/clm_varpar.F90](CTSM/clm_varpar.md)
- [CTSM/src/main/clm_instMod.F90](CTSM/clm_instMod.md)
- [CTSM/src/main/clm_varctl.F90](CTSM/clm_varctl.md)

### Changes to biogeophys
- [CTSM/src/biogeophys/SectorWaterMod.F90](CTSM/SectorWaterMod.md) `new module`
- [CTSM/src/biogeophys/HydrologyNoDrainageMod.F90](CTSM/HydrologyNoDrainageMod.md)
- [CTSM/src/biogeophys/BalanceCheckMod.F90](CTSM/BalanceCheckMod.md)
- [CTSM/src/biogeophys/Waterlnd2atmType.F90](CTSM/Waterlnd2atmType.F90)

%%
### Changes to tools
Here we don't go in much details. All changes to the tool are done following the instructions. And at the moment, what this does is to add to the surface data new fields corresponding to the sectoral water usage (withdrawal and consumption for each sector).

Only year 2000 is added at the moment (if we want to have transient/timeseries inputs for longer periods there are better ways to integrate the such data).

The data have monthly format, so 12 entries for year 2000.
- [CTSM/tools/mksurfdata_map/mksurfdata.pl](Documentation/CTSM/mksurfdata.md)
- [CTSM/tools/mksurfdata_map/src/mkfileMod.F90](Documentation/CTSM/mkfileMod.md)
- [CTSM/tools/mksurfdata_map/src/Srcfiles](Documentation/CTSM/Srcfiles.md)
- [CTSM/tools/mksurfdata_map/src/mkvarctl.F90](Documentation/CTSM/mkvarctl.md)
- [CTSM/tools/mksurfdata_map/src/mksectorwaterMod.F90](Documentation/CTSM/mksectorwaterMod.md) `new module`
- [CTSM/tools/mksurfdata_map/src/mksurfdat.F90](Documentation/CTSM/mksurfdat.md)
- [CTSM/tools/mksurfdata_map/mksurfdata_map.namelist](Documentation/CTSM/mksurfdata_map.md)
