---
title: "namelist_defaults_ctsm_tools.xml"
tags:
- CTSM
- Namelist
enableToc: false # do not show a table of contents on this page
---

## General discussion:
This script builds the namelists for land component of the CESM model (CTSM/CLM5).

To support sectoral water usage we added two new subroutines: `setup_logic_sectorwater()` and `setup_logic_sectorwater_parameters()`.