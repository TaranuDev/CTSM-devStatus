---
title: "Input Data"
tags:
- InputData
enableToc: false # do not show a table of contents on this page
---
## Overview:
To estimate daily withdrawal and consumption fluxes we rely on existing datasets.

For historical period, we use the sectoral water usage reconstruction by [Huang et al, 2018](https://hess.copernicus.org/articles/22/2117/2018/). This dataset provide monthly withdrawal and consumption data for the period 1971-2010 for 6 sectors (irrigation, domestic, livestock, thermoelectric, manufacturing and mining).

In the case of CESM model, irrigation is already implemented, so the plan for this project is to account for the other 5 sectors to completely capture the human related water usage.

For future applications, we intend to use spatially and temporally downscaled GCAM data. This is a new dataset, covering a combination of 4RCMx5SSPx5GCM matrix. More information about the methodology can be found from [Khan et al, 2022](https://jgcri.github.io/khan-etal_2022_tethysSSPRCP/metarepo.html#Citation).