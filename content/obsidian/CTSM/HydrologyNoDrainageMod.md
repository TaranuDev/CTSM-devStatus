---
title: "HydrologyNoDrainageMod.F90"
tags:
- CTSM
- Biogeophys
enableToc: false # do not show a table of contents on this page
---

## Overview:
This module contains the main subroutine to execute the calculation of soil and snow hydrology.

But in addition to this here is also defined the `CalcAndWithdrawIrrigationFluxes()` subroutine which do the calculation of irrigation withdrawal fluxes and perform the withdrawal (including from groundwater).

This is why it is here that we defined the `CalcAndWithdrawSectorWaterFluxes()` subroutine, with a similar objective:
- compute current day withdrawal, consumption and return flow;
- apply these fluxes;
- update the status of `water_inst` variables which should be transferred to the `routing component`;


## Code:
```fortran
subroutine CalcAndWithdrawSectorWaterFluxes(bounds, soilhydrology_inst, sectorwater_inst, water_inst, volr, rof_prognostic)
!
! !DESCRIPTION:
! Calculates sector water withdrawal fluxes and withdraws from groundwater
!
! !USES:
! use SoilHydrologyMod       , only : WithdrawGroundwaterSectorWater
use clm_time_manager       , only : is_beg_curr_day
!
! !ARGUMENTS:
integer  :: g  ! gridcell index
type(bounds_type)              , intent(in)    :: bounds
type(soilhydrology_type)       , intent(in)    :: soilhydrology_inst
type(sectorwater_type)         , intent(inout) :: sectorwater_inst
type(water_type)               , intent(inout) :: water_inst

! river water volume (m3) (ignored if rof_prognostic is .false.)
real(r8), intent(in) :: volr( bounds%begg: )

! whether we're running with a prognostic ROF component; this is needed to determine
! whether we can limit irrigation based on river volume.
logical, intent(in) :: rof_prognostic

! !LOCAL VARIABLES:
integer :: i  ! tracer index

character(len=*), parameter :: subname = 'CalcAndWithdrawSectorWaterFluxes'
!-----------------------------------------------------------------------

! Read withdrawal and consumption data from input surfdata
! Compute the withdrawal, consumption and return flow (expected and actual)
! To limit computation time, call this subroutine only once a day
if (is_beg_curr_day()) then
   call sectorwater_inst%CalcSectorWaterNeeded(bounds, volr, rof_prognostic)
endif

! This part is not elegant at all and will be replaced later
do g = bounds%begg, bounds%endg
   water_inst%waterlnd2atmbulk_inst%qdom_withd_grc(g) = sectorwater_inst%dom_withd_actual_grc(g)
   water_inst%waterlnd2atmbulk_inst%qdom_rf_grc(g) = sectorwater_inst%dom_rf_actual_grc(g)

   water_inst%waterlnd2atmbulk_inst%qliv_withd_grc(g) = sectorwater_inst%liv_withd_actual_grc(g)
   water_inst%waterlnd2atmbulk_inst%qliv_rf_grc(g) = sectorwater_inst%liv_rf_actual_grc(g)

   water_inst%waterlnd2atmbulk_inst%qelec_withd_grc(g) = sectorwater_inst%elec_withd_actual_grc(g)
   water_inst%waterlnd2atmbulk_inst%qelec_rf_grc(g) = sectorwater_inst%elec_rf_actual_grc(g)

   water_inst%waterlnd2atmbulk_inst%qmfc_withd_grc(g) = sectorwater_inst%mfc_withd_actual_grc(g)
   water_inst%waterlnd2atmbulk_inst%qmfc_rf_grc(g) = sectorwater_inst%mfc_rf_actual_grc(g)

   water_inst%waterlnd2atmbulk_inst%qmin_withd_grc(g) = sectorwater_inst%min_withd_actual_grc(g)
   water_inst%waterlnd2atmbulk_inst%qmin_rf_grc(g) = sectorwater_inst%min_rf_actual_grc(g)

end do

end subroutine CalcAndWithdrawSectorWaterFluxes
```


## Things which may require change:
At the moment, the consumed water for each sector is not used in any way. Which means that the system is constantly loosing water as the withdrawal > return flow. In the future we plan to add the consumed water to the surface soil (only for natural vegetation and pervious roads columns). See more details [CTSM/src/main/clm_driver.F90](Documentation/CTSM/clm_driver.md).

In addition to this when we will apply the consumption water to the surface soil, we will need to account this in the `column` and `gridcell` balance check [CTSM/src/biogeophys/BalanceCheckMod.F90](Documentation/CTSM/BalanceCheckMod.md).
