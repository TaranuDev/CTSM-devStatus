## Here we track all modifications done to the MOSART in order to support sectoral water usage

The GitHub fork can be accessed [here](https://github.com/TaranuDev/CTSM)



### List of modifications:
`namelist modifications`
[[CTSM/bld/namelist_files/namelist_definition_ctsm.xml]]
[[CTSM/bld/namelist_files/namelist_defaults_ctsm_tools.xml]]
[[CTSM/bld/namelist_files/namelist_defaults_ctsm.xml]]
[[CTSM/bld/CLMBuildNamelist.pm]]

`connect all forks using corresponding repositories and tags`
[[CTSM/Externals.cfg]]

`changes to the main`
[[CTSM/src/main/clm_driver.F90]]
[[CTSM/src/main/restFileMod.F90]]
[[CTSM/src/main/controlMod.F90]]
[[CTSM/src/main/clm_varpar.F90]]
[[CTSM/src/main/clm_instMod.F90]]
[[CTSM/src/main/clm_varctl.F90]]

`changes to biogeophys`
[[CTSM/src/biogeophys/SectorWaterMod.F90]] `new module`
[[CTSM/src/biogeophys/HydrologyNoDrainageMod.F90]]
[[CTSM/src/biogeophys/BalanceCheckMod.F90]]


`changes to tools`
! Here we don't go in much details. All changes to the tool are done following the instructions. And at the moment, what this does is to add to the surface data new fields corresponding to the sectoral water usage (withdrawal and consumption for each sector).

! Only year 2000 is added at the moment (if we want to have transient/timeseries inputs for longer periods there are better ways to integrate the such data).

! The data have monthly format, so 12 entries for year 2000.
[[CTSM/tools/mksurfdata_map/mksurfdata.pl]]
[[CTSM/tools/mksurfdata_map/src/mkfileMod.F90]]
[[CTSM/tools/mksurfdata_map/src/Srcfiles]]
[[CTSM/tools/mksurfdata_map/src/mkvarctl.F90]]
[[CTSM/tools/mksurfdata_map/src/mksectorwaterMod.F90]] `new module`
[[CTSM/tools/mksurfdata_map/src/mksurfdat.F90]]
[[CTSM/tools/mksurfdata_map/mksurfdata_map.namelist]]
