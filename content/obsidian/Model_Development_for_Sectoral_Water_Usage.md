---
title: "Model Development for Sectoral Water Usage"
tags:
enableToc: false # do not show a table of contents on this page
---

## General description:
The objective of this model development is to extend CTSM functionality to support sectoral water usage.

The supported sectors are: domestic, livestock, thermoelectric, manufacturing and mining.

Instead of deriving the sectoral demand and consumption from predictors, we will rely instead on available [datasets](./Input_Data.md). But the development will be done in such a way, that it will be relatively simple to build on it in the future by adding new functionalities (e.g. demand based on predictors).

The main features of this development are:
1. Daily withdrawal and consumption fluxes are computed using the input data. `land component`
2. Withdrawal is satisfied from both surface (river water) and groundwater sources (unconfined gw). The withdrawal from groundwater is done only if there is no sufficient water in river storage. `land component`
3. The consumed water is applied on surface soil over natural vegetation (and potentially pervious roads). `land component`
4. The withdrawal and return flow fluxes are transferred to the routing component (MOSART). `coupler`
5. The return flow is computed as the difference between actual withdrawal and consumption. `routing`
6. The updated river volume is sent back to the land model (at the coupling frequency). `coupler`

Following this approach we assure conservation of water, and at the same time account for the sectoral water usage impact on surface energy budget over land since consumption is directly applied to the soil surface.

### The development include changes to the following model components:
- [Changes Tracker CTSM](./CTSM/Changes_Tracker_CTSM.md)
- [Changes Tracker CMEPS](./CMEPS/Changes_Tracker_CMEPS.md)
- [Changes Tracker CPL7](./CPL7/Changes_Tracker_CPL7.md)
- [Changes Tracker MOSART](./MOSART/Changes_Tracker_MOSART.md)



