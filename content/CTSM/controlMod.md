---
title: "controlMod.F90"
tags:
- CTSM
- Main
enableToc: false # do not show a table of contents on this page
---
## Overview:
Here we add the `clm_inparm` namelist group the logical entry `sectorwater`.

Also we `mpi_bcast` the `sectorwater` value to all the processes of the group.