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