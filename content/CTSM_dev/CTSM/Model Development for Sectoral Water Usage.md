## General description:
The objective of this model development is to extend CTSM functionality to support sectoral water usage.

The supported sectors are: domestic, livestock, thermoelectric, manufacturing and mining.

Instead of deriving the sectoral demand and consumption from predictors, we will rely instead on avaialable datasets. But the development will be done in such a way, that it will be relatively simple to build on it, for example by computing the demand of a sector by using a set of predictors.

The main features of this development are:
1. Daily withdrawal and consumption fluxes are computed using the input data. `land component`
2. Withdrawal is satisfied from both surface (river water) and groundwater sources (unconfined gw). The withdrawal from groundwater is done only if there is no sufficient water in river storage. `land component`
3. The consumed water is applied on surface soil over natural vegetation (and potentially pervious roads). `land component`
4. The withdrawal and consumption fluxes are transfered to the routing component (MOSART). `coupler`
5. The return flow is computed as the difference between withdrawal and consumption (after adjusting the fluxes taking into account river storage). `routing`
6. The updated river volume is sent back to the land model (at the coupling frequency). `coupler`

Following this approach we assure conservation of water, and at the same time account for the sectoral water usage impact on surface energy budget over land since consumption is directly applied to the soil surface.

### The development include changes to the following model components:
[[Changes Tracker CTSM]]
[[Changes Tracker CMEPS]]
[[Changes Tracker CPL7]]
[[Changes Tracker MOSART]]



