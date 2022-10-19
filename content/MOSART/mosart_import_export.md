---
title: "mosart_import_export.F90"
tags:
- MOSART
- Coupler
enableToc: false # do not show a table of contents on this page
---


## Overview:
Using the import fields indices for sectoral water usage fluxes defined in [MOSART/src/cpl/mct/mosart_cpl_indices.F90](Documentation/MOSART/mosart_cpl_indices.md) we extract the relevant fields (withdrawal and return flow for each sector) and store them in the `rtmCTL` object which is an instance of `runoff_flow` object.

