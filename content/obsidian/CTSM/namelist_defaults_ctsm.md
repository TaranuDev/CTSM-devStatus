---
title: "namelist_defaults_ctsm.xml"
tags:
- CTSM
- Namelist
enableToc: false # do not show a table of contents on this page
---

## Overview:
Here we add the default values for the namelist entries supporting sectoral water usage (see groups 1-2 in [CTSM/bld/namelist_files/namelist_definition_ctsm.xml](Documentation/CTSM/namelist_definition_ctsm.md)).

## Code:
```php
<!-- Sector Water Usage default -->
<sectorwater phys="clm5_1" sim_year_range="1850-2100">.false.</sectorwater>
<sectorwater phys="clm5_0" sim_year_range="1850-2100">.false.</sectorwater>
<sectorwater phys="clm4_5" sim_year_range="1850-2100">.false.</sectorwater>

<sectorwater >.false.</sectorwater>
	

<!-- Sector water usage namelist defaults -->
<dom_and_liv_start_time>0</dom_and_liv_start_time>
<ind_start_time>0</ind_start_time>
<dom_and_liv_length>86400</dom_and_liv_length>
<ind_length>86400</ind_length>
<sectorwater_river_volume_threshold>0.1</sectorwater_river_volume_threshold>
	
<!--  River storage derived lake evaporation and sector water usage limitation  -->
<limit_sectorwater_if_rof_enabled phys="clm5_1" >.false.</limit_sectorwater_if_rof_enabled>
<limit_sectorwater_if_rof_enabled phys="clm5_0" >.false.</limit_sectorwater_if_rof_enabled>
<limit_sectorwater_if_rof_enabled               >.false.</limit_sectorwater_if_rof_enabled>

<use_groundwater_sectorwater>.false.</use_groundwater_sectorwater>

```