---
title: "mosart_cpl_indices.F90"
tags:
- MOSART
- Coupler
enableToc: false # do not show a table of contents on this page
---


## Overview:
This module simply extract the indices for the fields passed between MOSART and the driver.

To support sectoral water usage we add the indices corresponding to sectoral water usage. Later these indices are used to extract the corresponding field from the array `coupler -> routing`.