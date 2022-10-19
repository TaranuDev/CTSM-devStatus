---
title: "prep_rof_mod.F90"
tags:
- CPL7
enableToc: false # do not show a table of contents on this page
---

## Overview:



## Code:
```fortran
module prep_rof_mod

...

! define as parameters the field names and lists, for fields that need to be treated specially (including irrigation and sectoral water withdrawal and return flows for each sector)
character(len=*), parameter :: dom_withd_flux_field = 'Flrl_dom_withd'
character(len=*), parameter :: dom_rf_flux_field = 'Flrl_dom_rf'
...

! whether the model is being run with a separate irrigation/sector water usage field
logical :: have_dom_field
...

subroutine prep_rof_init(infodata, lnd_c2_rof, atm_c2_rof)

!---------------------------------------------------------------
! Description
! Initialize module attribute vectors and all other non-mapping
! module variables


...

! define integers to store index of specific sector water field 
! this index is used to extract the right fields from the arrays containing all fields passed through the coupler
integer                     :: index_dom_withd
integer                     :: index_dom_rf
...


if (rof_present) then
  x2r_rx => component_get_x2c_cx(rof(1)) ! coupler to rof array containing all fields
  ...
  ! extract indices for each field of interest
  index_dom_withd = mct_aVect_indexRA(x2r_rx, dom_withd_flux_field, perrWith='quiet')
  index_dom_rf = mct_aVect_indexRA(x2r_rx, dom_rf_flux_field, perrWith='quiet')
  ...
  
  ! assign values to the logical have_field variables for each sector
  ! if obtained index is different from 0, then field exist => .true. and otherwise .false.
  ...
end if



if (rof_present .and. lnd_present) then

  ! maping of land field to rof (here we change nothing)
  ...
  
  ! Irrigation and sectoral water usage are mapped separatly 
   call shr_string_listDiff( &
		list1 = seq_flds_l2x_fluxes_to_rof, &
		list2 = dom_withd_flux_field, &
		listout = lnd2rof_normal_fluxes)
   ...
  endif

end if


...
end subroutine prep_rof_init



...

subroutine prep_rof_merge(l2x_r, a2x_r, fractions_r, x2r_r, cime_model)

!-----------------------------------------------------------------------
! Description
! Merge land rof and ice forcing for rof input


...
! define indeces of all fields to be merged (including for irrigation and sectoral water usage)
integer, save :: index_l2x_Flrl_dom_withd ! l2x means land -> coupler
integer, save :: index_l2x_Flrl_dom_rf
...
integer, save :: index_x2r_Flrl_dom_withd ! x2r means coupler -> routing
integer, save :: index_x2r_Flrl_dom_rf
...

if (first_time) then
  ! allocate memory for the merging array (here we change nothing)
  ...
  ! For each sector get the index of the corresponding sector field from the land to coupler array
  if (have_dom_field) then
     index_l2x_Flrl_dom_withd  = mct_aVect_indexRA(l2x_r,'Flrl_dom_withd' )
     index_l2x_Flrl_dom_rf  = mct_aVect_indexRA(l2x_r,'Flrl_dom_rf' )
  end if
  ...
  ! and same for the coupler to routing array 
  if (have_dom_field) then
     index_x2r_Flrl_dom_withd  = mct_aVect_indexRA(x2r_r,'Flrl_dom_withd' )
     index_x2r_Flrl_dom_rf  = mct_aVect_indexRA(x2r_r,'Flrl_dom_rf' )
  end if
  ...
end if

do i = 1,lsize
	! Fill the array x2r based on fields already saved in l2x (including for sector water usage)
	...
end do

...
end subroutine prep_rof_merge


subroutine prep_rof_calc_l2r_rx(fractions_lx, timer)
! Here we added the remaping of sectoral water usage fluxes
! using map_lnd2rof_sectorwater() subroutine
end subroutine prep_rof_calc_l2r_rx

```