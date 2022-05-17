Tags: #CPL7

## Overview:
New module created to support sectoral water usage. It is inspired from the existing module `driver/main/map_lnd2rof_irrig_mod.F90`.

This module contains routines for mapping the sectoral water fields (both withdrawal and return flow) from the LND grid onto the ROF grid.

The mapping is done for all the fluxes at the same time.

The basic idea is that we want to pull/add sectoral water fluxes out of/in ROF cells proportionally to the river volume (volr) in each cell. This is important in cases where the various ROF cells overlapping a CLM cell have very different volr.

If we didn't do this volr-normalized remapping, we'd try to extract the same amount of water from each of the ROF cells, which would be more likely to have withdrawals exceeding available volr.

 MOSART model have code to handle excess withdrawals, by pulling the excess directly out of the ocean, but we'd like to avoid resorting to this as much as possible.
 
 This mapping works by:
	 1. Normalizing the land's sector water flux by volr
	 2. Mapping this volr-normalized flux to the rof grid
	 3. Converting the mapped, volr-normalized flux back to a normal (non-volr-normalized) flux on the rof grid.
	 
## Code:
In principle it is the same code as in the [[med_phases_prep_rof_mod.F90]] so we will not go in much  details here.

For the moment, it is a little bit unclear why we need two instances of same code and why [[med_phases_prep_rof_mod.F90]] is not sufficient.

I will need to check the calls in more details to clear this out.
