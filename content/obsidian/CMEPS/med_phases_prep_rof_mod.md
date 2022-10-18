---
title: "med_phases_prep_rof_mod.F90"
tags:
- CMEPS
enableToc: false # do not show a table of contents on this page
---

## Overview:
It is here that we accumulate the import land fields which are later distributed to other model components.

To support sectoral water usage several new fields are added and a new subroutine `med_phases_prep_rof_sectorwater` is created inspired from `med_phases_prep_rof_irrig`. Its role is to remap, in a safe way (proofing against negative `volr`), the fields from `land component` to the `routing component`. 

## Code:
```fortran
module med_phases_prep_rof_mod

  !-----------------------------------------------------------------------------
  ! Create rof export fields
  ! - accumulate import lnd fields on the land grid that are sent to rof
  !   - done in med_phases_prep_rof_accum
  ! - time avergage accumulated import lnd fields on lnd grid when necessary and
  !   then map the time averaged accumulated lnd fields to the rof grid
  !   and then merge the mapped lnd fields to create FBExp(comprof)
  !   - done in med_phases_prep_rof_avg
  !-----------------------------------------------------------------------------

...
  ! new subroutine
  private :: med_phases_prep_rof_sectorwater


  subroutine med_phases_prep_rof_sectorwater(gcomp, rc)

   !---------------------------------------------------------------
   ! Description
   ! Do custom mapping for the sectoral water fluxes, from land -> rof.
   !
   ! The basic idea is that we want to pull/add the fluxes out of ROF cells proportionally to
   ! the river volume (volr) in each cell. This is important in cases where the various
   ! ROF cells overlapping a CTSM cell have very different volr: If we didn't do this
   ! volr-normalized remapping, we'd try to extract the same amount of water from each
   ! of the ROF cells, which would be more likely to have withdrawals exceeding
   ! available volr.
   !
   ! (Both RTM and MOSART have code to handle excess withdrawals by pulling the excess
   ! directly out of the ocean. We'd like to avoid resorting to this if possible.
   !
   ! This mapping works by:
   ! (1) Normalizing the land's sectoral water fluxes by volr
   ! (2) Mapping this volr-normalized flux to the rof grid
   ! (3) Converting the mapped, volr-normalized flux back to a normal
   !     (non-volr-normalized) flux on the rof grid.
   !---------------------------------------------------------------

   ! input/output variables
   ... ! here we change nothing 
   
   ! local variables
   ... ! here we add ESMF_Field variables correponding to sectoral withdrawal and return flow fluxes
   ! as well as pointers to the ESMF_Field for each sector corresponding 
   ! to the land and routing normalized withdrawals and return flows,
   ! as well as flux and river volume 

   !---------------------------------------
   ! Get the internal state
   !---------------------------------------
	... ! change nothing

   ! ------------------------------------------------------------------------
   ! Initialize module field bundles if not already initialized
   ! ------------------------------------------------------------------------

   ! Check if the required fields are created and if not
   ... 
   ! Create and get the fields in source and destination field bundles
   ...
   
  
   ! ------------------------------------------------------------------------
   ! 1) Create volr_l: Adjust volr_r, and map it to the land grid
   ! ------------------------------------------------------------------------

   ! Treat any rof point with volr < 0 as if it had volr = 0. Negative volr values can
   ! arise in RTM. This fix is needed to avoid mapping negative sector waterfluxes to those
   ! cells: while conservative, this would be unphysical (it would mean that sector water fluxes
   ! actually adds water to those cells).

   ! Create volr_r 
   ... ! change nothing here

   ! Map volr_r to volr_l (rof->lnd) using conservative mapping without any fractional weighting
   ... ! change nothing here
   
   ! Get volr_l
   ... ! change nothing here

   ! ------------------------------------------------------------------------
   ! (2) Determine sector water usage from land on land grid normalized by volr_l
   ! ------------------------------------------------------------------------

   ! In order to avoid possible divide by 0, as well as to handle non-sensical negative
   ! volr on the land grid, we divide the land's sector water flux into two separate flux
   ! components:
   ! - a component where we have positive volr on the land grid (put in
   !   sectorX_withd/rf_normalized_l, which is mapped using volr-normalization)
   ! - a component where we have zero or negative volr on the land
   !   grid (put in sectorX_withd/rf_volr0_l, which is mapped as a standard flux).
   ! We then remap both of these components to the rof grid, and then
   ! finally add the two components to determine the total sector water
   ! flux on the rof grid.

   ! First extract accumulated sector water flux from land (both withdrawal and return flow)
  ...

   ! Fill in values for sectorX_withd/rf_normalized_l and sectorX_withd/rf_volr0_l
 ...
 
 ! Loop over each volr gridcell from the land component
 	  ! if volr in specified grid > 0 (positive) then
      		! directly normalize withdrawal and return flow by the volr_l(l)
      ! else
         	! treat withdrawal and return flow as normal fluxes 
			! (no normalization)
 ! Finish loop over volr gridcell from the land component
 
 ! ------------------------------------------------------------------------
 ! (3) Map normalized sector water fluxes from land to rof grid and
 !     convert to a total sector fluxes on the ROF grid
 ! ------------------------------------------------------------------------

 ! maping is done using the med_map_field_normalized() subroutine
 ! we are remaping the two components (one which result when volr_l(l) <= 0 
 ! which is not normalized and the one when volr_l(l) > 0, which is normalized)
 ...
 
 ! Convert to a total sector water flux on the ROF grid, and put this in the pre-merge FBlndAccum2rof_r
 ...

 ! loop over the all the routing gridcells 'r' over which we do the mapping
 	  ! the total withdrawl and return flows are then reconstructed as 
	  ! the sum of the normalizable and non-normalizable component
 ! end loop
```