---
title: "controlMod.F90"
tags:
- CTSM
- Main
enableToc: false # do not show a table of contents on this page
---
## Overview:
Here we add the logical entry `sectorwater` to the `clm_inparm` namelist group.

We also `mpi_bcast` the `sectorwater` variable, so that its value can be accessed from anywhere. 