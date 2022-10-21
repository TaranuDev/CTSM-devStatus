---
title: "namelist_defaults_ctsm_tools.xml"
tags:
- CTSM
- Namelist
enableToc: false # do not show a table of contents on this page
---


## General discussion:
In order to use the sectoral abstractions [input data](Input_Data.md), the dataset should be pre-processed using a special [toolset](https://github.com/ESCOMP/CTSM/tree/master/tools) to prepare the surface dataset which will be used by the land component CTSM/CLM5 during the run.

## What changed:
In order to run the toolset and prepare the `surfdata` based on the pre-processed `rawdata` we need to specify some details about the rawdata, such as its location, land mask and horizontal resolution.

**Attention:** In the near future the existing toolset will receive an important update, significantly changing the workflow to prepare the input data. But at the moment, this is the current standard.