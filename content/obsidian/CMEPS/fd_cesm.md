---
title: "fd_cesm.yaml"
tags:
- CMEPS
enableToc: false # do not show a table of contents on this page
---

## Overview:
Here we simply add to the mediator field dictionary new fields corresponding to the withdrawal and return flow fluxes.

## Code:
```perl
# example of added field:
- standard_name: Flrl_dom_withd
  canonical_units: kg m-2 s-1
  description: land export to river
```