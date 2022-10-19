---
title: "SectorWaterMod.F90"
tags:
- CTSM
- Biogeophys
enableToc: false # do not show a table of contents on this page
---

## Overview:
This new module is largely inspired from existing module `src/biogeophys/IrrigationMod.F90`.

## Code:
```fortran
module SectorWaterMod

#include "shr_assert.h"

! specify all module/subroutines used
! here I will need to update the list by taking out things I didn't really used
...

  

!PUBLIC TYPES:
! Define sector water usage module parameters
type, public :: sectorwater_params_type

	! Time of day to start domestic and livestock water use, seconds (0 = midnight).
	! We start satisfying the demand in the time step FOLLOWING this time,
	integer :: dom_and_liv_start_time

	! Time of day to start industrial (thermoelectric, manufacturing and mining) water use, seconds (0 = midnight).
	! We start satisfying the demand in the time step FOLLOWING this time,
	integer :: ind_start_time

	! Legth in seconds other which domestic and livestock water demand is satisfied.
	! Actual time may differ if this is not a multiple of dtime.
	! SectorWater module won't work properly if dtime > secsperday.
	integer :: dom_and_liv_length

	! Legth in seconds other which industrial water demand is satisfied.
	integer :: ind_length

	! Threshold for river water volume below which sectorwater usage is 
	! shut off, if limit_sectorwater is .true. (fraction of available river water). 
	! A threshold of 0 means all river water can be used; a threshold of 0.1 means 90% of the river volume can be used;
	real(r8) :: sectorwater_river_volume_threshold

	! Whether sectorwater usage is limited based on river storage.
	! This only applies if ROF is enabled 
	! (i.e., rof_prognostic is .true.) - otherwise we don't limit 
	! sectorwater usage, regardless of the value of this flag.
	logical :: limit_sectorwater_if_rof_enabled

	! use groundwater supply for sectorwater usage 
	! (in addition to surface water)
	logical :: use_groundwater_sectorwater
end type sectorwater_params_type

  
  
! Define the sectorwater_type to store dynamical quantities related to sectoral
! water usage, as well as the required procedures  
type, public :: sectorwater_type

	type(sectorwater_params_type) :: params

	integer :: dtime ! land model time step (sec)

	integer :: dom_and_liv_nsteps_per_day ! number of time steps per day in which we satisfy domestic and livestock demand

	integer :: ind_nsteps_per_day ! number of time steps per day in which we satisfy industrial demand (thermoelectric, manufacturing and mining)

	! Private data members; time-varying:
	! naming: dom = domestic, liv = livestock, elec = thermoelectric, 
	! mfc = manufacturing, min = mining
	! naming: withd = withdrawal, cons = consumption, rf = return flow



	real(r8), pointer :: input_mon_dom_withd_grc (:) ! input expected withdrawal for current month

	real(r8), pointer :: input_mon_dom_cons_grc (:) ! input expected consumption for current month

	real(r8), pointer :: dom_withd_grc (:) ! expected withdrawal flux for the day [mm/s]

	real(r8), pointer :: dom_cons_grc (:) ! expected consumption flux for the day [mm/s]

	real(r8), pointer :: dom_withd_actual_grc (:) ! actual withdrawal flux for the day [mm/s]

	real(r8), pointer :: dom_cons_actual_grc (:) ! actual consumption flux for the day [mm/s]

	real(r8), pointer :: dom_rf_actual_grc (:) ! actual return flow flux for the day [mm/s]

	! define same quantities for other sectors
	...

	integer , pointer :: n_dom_and_liv_steps_left_grc (:) ! number of time steps for which we still need to satisfy domestic and livestock demand (if 0, ignore)

	integer , pointer :: n_ind_steps_left_grc (:) ! number of time steps for which we still need to satisfy industrial demand (if 0, ignore)

	contains

	! Public routines
	procedure, public :: Init => SectorWaterInit
	! procedure, public :: Restart
	procedure, public :: ReadSectorWaterData
	! procedure, public :: CalcSectorWaterFluxes
	procedure, public :: CalcSectorWaterNeeded
	procedure, public :: Clean => SectorWaterClean ! deallocate memory


	! Private routines
	procedure, private :: ReadNamelist
	procedure, private :: CheckNamelistValidity ! Check for validity of input parameters
	procedure, private :: InitAllocate => SectorWaterInitAllocate
	procedure, private :: InitHistory => SectorWaterInitHistory
	procedure, private :: InitCold => SectorWaterInitCold
	procedure, private :: CalcSectorDemandVolrLimited ! calculate demand limited by river volume for each patch

end type sectorwater_type

  

interface sectorwater_params_type

module procedure sectorwater_params_constructor

end interface sectorwater_params_type

  

real(r8), parameter :: m3_over_km2_to_mm = 1.e-3_r8

character(len=*), parameter, private :: sourcefile = &

__FILE__


contains

  

! ========================================================================

! Constructors

! ========================================================================
function sectorwater_params_constructor(dom_and_liv_start_time, ind_start_time, &
dom_and_liv_length, ind_length, sectorwater_river_volume_threshold, &
limit_sectorwater_if_rof_enabled, use_groundwater_sectorwater) &
result(this)
	! !DESCRIPTION:

	! Create an sectorwater_params instance
	
	! !ARGUMENTS:
	type(sectorwater_params_type) :: this ! function result

	integer , intent(in) :: dom_and_liv_start_time

	integer , intent(in) :: ind_start_time

	integer , intent(in) :: dom_and_liv_length

	integer , intent(in) :: ind_length

	real(r8), intent(in) :: sectorwater_river_volume_threshold

	logical , intent(in) :: limit_sectorwater_if_rof_enabled

	logical , intent(in) :: use_groundwater_sectorwater

	! !LOCAL VARIABLES:

	character(len=*), parameter :: subname = 'sectorwater_params_constructor'

	!-----------------------------------------------------------------------

	this%dom_and_liv_start_time = dom_and_liv_start_time

	this%ind_start_time = ind_start_time

	this%dom_and_liv_length = dom_and_liv_length

	this%ind_length = ind_length

	this%sectorwater_river_volume_threshold = sectorwater_river_volume_threshold

	this%limit_sectorwater_if_rof_enabled = limit_sectorwater_if_rof_enabled

	this%use_groundwater_sectorwater = use_groundwater_sectorwater
	
end function sectorwater_params_constructor

  

! ========================================================================

! Infrastructure routines (initialization, restart, etc.)

! ========================================================================
subroutine SectorWaterInit(this, bounds, NLFilename, use_aquifer_layer)

	class(sectorwater_type) , intent(inout) :: this

	type(bounds_type) , intent(in) :: bounds

	character(len=*) , intent(in) :: NLFilename ! Namelist filename

	logical , intent(in) :: use_aquifer_layer

	call this%ReadNamelist(NLFilename, use_aquifer_layer)

	call this%InitAllocate(bounds) ! whether an aquifer layer is used in this run

	call this%InitHistory(bounds)

	call this%InitCold(bounds)

end subroutine SectorWaterInit

  

!-----------------------------------------------------------------------
subroutine ReadNamelist(this, NLFilename, use_aquifer_layer)
	! !DESCRIPTION:
	! Read the sectorwater namelist
	! !USES:
	...
	! !ARGUMENTS:

	class(sectorwater_type) , intent(inout) :: this
	character(len=*), intent(in) :: NLFilename ! Namelist filename
	logical, intent(in) :: use_aquifer_layer ! whether an aquifer layer is used in this run

	! !LOCAL VARIABLES:
	! variables from sectorwater_params_type
	integer :: dom_and_liv_start_time
	integer :: ind_start_time
	integer :: dom_and_liv_length
	integer :: ind_length
	real(r8) :: sectorwater_river_volume_threshold
	logical :: limit_sectorwater_if_rof_enabled
	logical :: use_groundwater_sectorwater



	integer :: ierr ! error code
	integer :: unitn ! unit for namelist file
	character(len=*), parameter :: nmlname = 'sectorwater_inparm'
	character(len=*), parameter :: subname = 'ReadNamelist'

	!-----------------------------------------------------------------------



	namelist /sectorwater_inparm/ dom_and_liv_start_time, ind_start_time, dom_and_liv_length, &
	ind_length, sectorwater_river_volume_threshold, limit_sectorwater_if_rof_enabled, &
	use_groundwater_sectorwater

	! Initialize options to garbage defaults, forcing all to be specified
	! explicitly in order to get reasonable results
	dom_and_liv_start_time = 0
	ind_start_time = 0
	dom_and_liv_length = 0
	ind_length = 0
	sectorwater_river_volume_threshold = nan
	limit_sectorwater_if_rof_enabled = .false.
	use_groundwater_sectorwater = .false.


	! Read the Namelist and fill the sectorwater_inparm namelist variables
	...


	! MPI_BCAST all the sectorwater_params_type variables values to 
	! all processes
	...
	
	! Update the sectorwater_type params with the values from namelist file
	this%params = sectorwater_params_type( &

	dom_and_liv_start_time = dom_and_liv_start_time, &

	ind_start_time = ind_start_time, &

	dom_and_liv_length = dom_and_liv_length, &

	ind_length = ind_length, &

	sectorwater_river_volume_threshold = sectorwater_river_volume_threshold, &

	limit_sectorwater_if_rof_enabled = limit_sectorwater_if_rof_enabled, &

	use_groundwater_sectorwater = use_groundwater_sectorwater)


	! Write to the log file the parameters values obtained from namelist file
	! Later log files can be checked to confirm that the right values were used
	! Here we also call the CheckNamelistValidity(use_aquifer_layer) subroutine
	...
	
end subroutine ReadNamelist

!-----------------------------------------------------------------------

subroutine CheckNamelistValidity(this, use_aquifer_layer)
	! !DESCRIPTION:

	! Check for validity of input parameters.
	! Assumes that the inputs have already been packed into 'this%params'.
	! Only needs to be called by the master task, since parameters are the same 
	! for all tasks.

	! !ARGUMENTS:
	class(sectorwater_type), intent(in) :: this
	logical, intent(in) :: use_aquifer_layer ! whether an aquifer layer is used in this run

	! !LOCAL VARIABLES:
	character(len=*), parameter :: subname = 'CheckNamelistValidity'

	!-----------------------------------------------------------------------
	! Use associate to access the this%params% variables with shortcuts
	...

	! Use if statements to check if params values are in admissible values range
	...  

end subroutine CheckNamelistValidity

!-----------------------------------------------------------------------

subroutine SectorWaterInitAllocate(this, bounds)
	! !DESCRIPTION:
	! Initialize sector water data structure

	! !USES:
	...

	! !ARGUMENTS:
	class(sectorwater_type) , intent(inout) :: this
	type(bounds_type) , intent(in) :: bounds

	! !LOCAL VARIABLES:
	integer :: begg, endg ! bounds limits

	character(len=*), parameter :: subname = 'InitAllocate'

	!-----------------------------------------------------------------------
	begg = bounds%begg; endg= bounds%endg


	! Allocate memory for variables of sectorwater_type in the limits of the 
	! clumps bounds (+ give nan as default values)
	allocate(this%input_mon_dom_withd_grc (begg:endg)) ; this%input_mon_dom_withd_grc (:) = nan
	allocate(this%input_mon_dom_cons_grc (begg:endg)) ; this%input_mon_dom_cons_grc (:) = nan
	allocate(this%dom_withd_grc (begg:endg)) ; this%dom_withd_grc (:) = nan
	allocate(this%dom_cons_grc (begg:endg)) ; this%dom_cons_grc (:) = nan
	allocate(this%dom_withd_actual_grc (begg:endg)) ; this%dom_withd_actual_grc (:) = nan
	allocate(this%dom_cons_actual_grc (begg:endg)) ; this%dom_cons_actual_grc (:) = nan
	allocate(this%dom_rf_actual_grc (begg:endg)) ; this%dom_rf_actual_grc (:) = nan

	! Do the same for other sectors
	...
	
end subroutine SectorWaterInitAllocate

!-----------------------------------------------------------------------

subroutine SectorWaterInitHistory(this, bounds)
	! !DESCRIPTION:
	! Initialize sectoral water use history fields

	! !USES:
	...

	! !ARGUMENTS:
	class(sectorWater_type) , intent(inout) :: this
	type(bounds_type) , intent(in) :: bounds

	! !LOCAL VARIABLES:
	integer :: begg, endg
	character(len=*), parameter :: subname = 'InitHistory'
	!-----------------------------------------------------------------------
	begg = bounds%begg; endg= bounds%endg


	! Add actual withdrawal as history fields for outputs
	! Maybe later we can add other fields too
	! But for the first tests, this is enough
	this%dom_withd_actual_grc(begg:endg) = spval
	call hist_addfld1d (fname='DOM_ACTUAL_WITHD', units='mm/s', &
	avgflag='A', long_name='domestic actual withdrawal flux', &
	ptr_patch=this%dom_withd_actual_grc, default='inactive')

	! Do the same for other sectors
	...

end subroutine SectorWaterInitHistory

  

!-----------------------------------------------------------------------

subroutine SectorWaterInitCold(this, bounds)
	! !DESCRIPTION:
	! Do cold-start initialization for sector water data structure

	! !ARGUMENTS:
	class(sectorwater_type) , intent(inout) :: this
	type(bounds_type) , intent(in) :: bounds
	character(len=*), parameter :: subname = 'InitCold'
	!-----------------------------------------------------------------------

	! Initialize the sectorwater_type variables 
	this%dtime = get_step_size()
	this%ind_nsteps_per_day = 0._r8
	this%dom_and_liv_nsteps_per_day = 0._r8

	! actual withdrawl
	this%dom_withd_actual_grc(bounds%begg:bounds%endg) = 0._r8
	... ! same for other sectors

	! actual consumption
	this%dom_cons_actual_grc(bounds%begg:bounds%endg) = 0._r8
	... ! same for other sectors

	! actual return flow
	this%dom_rf_actual_grc(bounds%begg:bounds%endg) = 0._r8
	... ! same for other sectors
end subroutine SectorWaterInitCold

!-----------------------------------------------------------------------

pure function Calc_dom_and_liv_NstepsPerDay(this, dtime) result(dom_and_liv_nsteps_per_day)
	! !DESCRIPTION:
	! Given dtime (sec), determine number of steps per day to satisfy demand for domestic and livestock sectors
	...
end function Calc_dom_and_liv_NstepsPerDay

!-----------------------------------------------------------------------

pure function Calc_ind_NstepsPerDay(this, dtime) result(ind_nsteps_per_day)
	! !DESCRIPTION:
	! Given dtime (sec), determine number of steps per day to satisfy demand for industrial sectors
	...
end function Calc_ind_NstepsPerDay

!-----------------------------------------------------------------------

subroutine SectorWaterClean(this)
	! !DESCRIPTION:

	! Deallocate memory for all sectorwater_type variables
	...
end subroutine sectorWaterClean

! ========================================================================

! Science routines

! ========================================================================  

subroutine ReadSectorWaterData (this, bounds, mon)
	! !DESCRIPTION:

	! read the input data from fsurfdata (withdrawal and consumption)
	! at the moment fsurfdata for sector water module test have the data 
	! for year 2000 in monthly format for all sectors
	! when we will finish development we will not rely on surfdata, but on 
	! alternative methods more convenient for transient data

	! !USES:
	...

	! !ARGUMENTS:
	class(sectorwater_type), intent(inout) :: this
	type(bounds_type) , intent(in) :: bounds
	integer , intent(in) :: mon ! month (1, ..., 12) for nstep+1

	! !LOCAL VARIABLES:

	type(file_desc_t) :: ncid ! netcdf id
	integer :: ier ! error code
	integer :: g ! gridcell index
	integer :: ni,nj,ns ! indices
	integer :: dimid,varid ! input netCDF id's
	integer :: ntim ! number of input data time samples
	integer :: nlon_i ! number of input data longitudes
	integer :: nlat_i ! number of input data latitudes
	logical :: isgrid2d ! true => file is 2d
	
	real(r8), pointer :: mon_dom_withd(:) ! monthly domestic withdrawal read from input files
	real(r8), pointer :: mon_dom_cons(:) ! monthly domestic consumption read from input files
	... ! Same for other sectors


	character(len=256) :: locfn ! local file name
	character(len=32) :: subname = 'ReadSectorWaterData'

	!-----------------------------------------------------------------------

	if (masterproc) then

		write (iulog,*) 'Attempting to read annual sectoral water usage data .....'
	end if


	allocate(&
	mon_dom_withd(bounds%begg:bounds%endg), &
	mon_dom_cons(bounds%begg:bounds%endg), &
	..., & ! same for other sectors 
	stat=ier)

	! Determine necessary indices
	call getfil(fsurdat, locfn, 0)
	call ncd_pio_openfile (ncid, trim(locfn), 0)
	call ncd_inqfdims (ncid, isgrid2d, ni, nj, ns)

	! Check if ldomain and input file have matching dimensions (ni, nj and ns)
	! if not then abort run
	...

	! read the data for current month
	call ncd_io(ncid=ncid, varname='withd_dom', flag='read', data=mon_dom_withd, &
	dim1name=grlnd, nt=mon)
	call ncd_io(ncid=ncid, varname='cons_dom', flag='read', data=mon_dom_cons, &
	dim1name=grlnd, nt=mon)

	... ! do the same for other sectors

	call ncd_pio_closefile(ncid) ! close input file

	! fill the sectorwater_type input fields with current month 
	! withdrawal and consumption (loop over each gridcell)
	do g = bounds%begg,bounds%endg
		this%input_mon_dom_withd_grc(g) = mon_dom_withd(g)
		this%input_mon_dom_cons_grc(g) = mon_dom_cons(g)

		... ! do the same for other sectors
	end do

end subroutine ReadSectorWaterData

!-----------------------------------------------------------------------

subroutine CalcSectorWaterNeeded(this, bounds, volr, rof_prognostic)
	! !DESCRIPTION:
	! Calculate sector water fluxes (withdrawal, consumption)

	! !USES:
	use shr_const_mod , only : SHR_CONST_TKFRZ
	use clm_time_manager , only : get_curr_date, is_end_curr_month, get_curr_days_per_year

	! !ARGUMENTS:
	class(sectorwater_type) , intent(inout) :: this
	type(bounds_type) , intent(in) :: bounds
	! river water volume (m3) (ignored if rof_prognostic is .false.)
	real(r8), intent(in) :: volr(bounds%begg:bounds%endg)
	! whether we're running with a prognostic ROF component; 
	! this is needed to determine whether we can limit demand based on river volume.
	logical, intent(in) :: rof_prognostic

	! !LOCAL VARIABLES:
	integer :: g ! gridcell index
	integer :: year ! year (0, ...) for nstep+1
	integer :: mon ! month (1, ..., 12) for nstep+1
	integer :: day ! day of month (1, ..., 31) for nstep+1
	integer :: sec ! seconds into current date for nstep+1
	real(r8) :: dayspyr ! days per year
	real(r8) :: dayspm ! days per month
	real(r8) :: secs_per_day ! seconds per day
	real(r8) :: dom_and_liv_flux_factor ! factor to transform the demand from mm per month to mm/s for the given day
	real(r8) :: ind_flux_factor ! factor to transform the demand from mm per month to mm/s for the given day



	real(r8) :: dom_demand(bounds%begg:bounds%endg)
	real(r8) :: dom_demand_volr_limited(bounds%begg:bounds%endg)


	real(r8) :: dom_consumption(bounds%begg:bounds%endg)
	real(r8) :: dom_consumption_volr_limited(bounds%begg:bounds%endg)

	... ! Same for other sectors

	! Whether we should limit deficits by available volr
	logical :: limit_sectorwater

	character(len=*), parameter :: subname = 'CalcSectorWaterNeeded'

	!-----------------------------------------------------------------------

	! Get current date
	call get_curr_date(year, mon, day, sec)
	! Get number of days in current year
	dayspyr = get_curr_days_per_year()
	! Compute average number of days per month for the current year
	dayspm = dayspyr/12_r8
	! Compute the flux factors to transform from mm/month to mm/s
	dom_and_liv_flux_factor = ((1_r8/dayspm)/this%params%dom_and_liv_length)
	ind_flux_factor = ((1_r8/dayspm)/this%params%ind_length)



	! Read input for new month if end of month
	! This will update the input fields of withdrawal and consumption
	! of sectorwater_type
	if (is_end_curr_month()) then
		call this%ReadSectorWaterData(bounds, mon)
	endif

	! Compute demand [mm]
	! First initialize demand to 0 everywhere;
	dom_demand(bounds%begg:bounds%endg) = 0._r8
	dom_consumption(bounds%begg:bounds%endg) = 0._r8

	... ! same for other sectors

	! Update this day fluxes based on input data
	do g = bounds%begg,bounds%endg
		dom_demand(g) = this%input_mon_dom_withd_grc(g)
		dom_consumption(g) = this%input_mon_dom_cons_grc(g)

		... ! same for other sectors
	end do ! end loop over gridcels



	! Limit sectoral withdrawal based on volr (river volume)
	! Note that we cannot do this limiting if running without a prognostic
	! river model, since we need river volume for this limiting.
	limit_sectorwater = (this%params%limit_sectorwater_if_rof_enabled .and. rof_prognostic)

	if (limit_sectorwater) then

		! Compute the allowed withdrawal if water limitation is taken into account
		call this%CalcSectorDemandVolrLimited( &
		bounds = bounds, &
		dom_demand = dom_demand(bounds%begg:bounds%endg), &
		dom_consumption = dom_consumption(bounds%begg:bounds%endg), &
		liv_demand = liv_demand(bounds%begg:bounds%endg), &
		liv_consumption = liv_consumption(bounds%begg:bounds%endg), &
		elec_demand = elec_demand(bounds%begg:bounds%endg), &
		elec_consumption = elec_consumption(bounds%begg:bounds%endg), &
		mfc_demand = mfc_demand(bounds%begg:bounds%endg), &
		mfc_consumption = mfc_consumption(bounds%begg:bounds%endg), &
		min_demand = min_demand(bounds%begg:bounds%endg), &
		min_consumption = min_consumption(bounds%begg:bounds%endg), &
		volr = volr(bounds%begg:bounds%endg), &
		dom_demand_volr_limited = dom_demand_volr_limited(bounds%begg:bounds%endg), &
		dom_consumption_volr_limited = dom_consumption_volr_limited(bounds%begg:bounds%endg), &
		liv_demand_volr_limited = liv_demand_volr_limited(bounds%begg:bounds%endg), &
		liv_consumption_volr_limited = liv_consumption_volr_limited(bounds%begg:bounds%endg), &
		elec_demand_volr_limited = elec_demand_volr_limited(bounds%begg:bounds%endg), &
		elec_consumption_volr_limited = elec_consumption_volr_limited(bounds%begg:bounds%endg), &
		mfc_demand_volr_limited = mfc_demand_volr_limited(bounds%begg:bounds%endg), &
		mfc_consumption_volr_limited = mfc_consumption_volr_limited(bounds%begg:bounds%endg), &
		min_demand_volr_limited = min_demand_volr_limited(bounds%begg:bounds%endg), &
		min_consumption_volr_limited = min_consumption_volr_limited(bounds%begg:bounds%endg))

	else
		! If water limitation is not considered
		! the consider the volr_limited withdrawals to be the same
		! as input based withdrawl
		dom_demand_volr_limited(bounds%begg:bounds%endg) = dom_demand(bounds%begg:bounds%endg)
		dom_consumption_volr_limited(bounds%begg:bounds%endg) = dom_consumption(bounds%begg:bounds%endg)
		... ! do the same for the other sectors
	end if



	! Convert demand to withdrawal rates [mm/s]

	do g = bounds%begg,bounds%endg
		! compute expected and actual withdrawal and consumption
		this%dom_withd_grc(g) = dom_demand(g)*dom_and_liv_flux_factor
		this%dom_withd_actual_grc(g) = dom_demand_volr_limited(g)*dom_and_liv_flux_factor

		this%dom_cons_grc(g) = dom_consumption(g)*dom_and_liv_flux_factor
		this%dom_cons_actual_grc(g) = dom_consumption_volr_limited(g)*dom_and_liv_flux_factor

		! computed actual return flow
		this%dom_rf_actual_grc(g) = this%dom_withd_actual_grc(g) - this%dom_cons_actual_grc(g)

		! Do the same for other sectors
		...
	end do

end subroutine CalcSectorWaterNeeded

!-----------------------------------------------------------------------

subroutine CalcSectorDemandVolrLimited(this, bounds, dom_demand, dom_consumption, liv_demand, liv_consumption, elec_demand, elec_consumption, &
mfc_demand, mfc_consumption, min_demand, min_consumption, volr, dom_demand_volr_limited, dom_consumption_volr_limited, liv_demand_volr_limited, &
liv_consumption_volr_limited, elec_demand_volr_limited, elec_consumption_volr_limited, mfc_demand_volr_limited, &
mfc_consumption_volr_limited, min_demand_volr_limited, min_consumption_volr_limited)

	! !ARGUMENTS:
	class(sectorwater_type) , intent(in) :: this
	type(bounds_type) , intent(in) :: bounds

	real(r8), intent(in) :: dom_demand( bounds%begg:bounds%endg)
	real(r8), intent(in) :: dom_consumption( bounds%begg:bounds%endg)

	... ! same for other sectors


	! river water volume [m3]
	real(r8), intent(in) :: volr( bounds%begg:bounds%endg)

	real(r8), intent(out) :: dom_demand_volr_limited( bounds%begg:bounds%endg)
	real(r8), intent(out) :: dom_consumption_volr_limited( bounds%begg:bounds%endg)

	... ! same for other sectors

	! !LOCAL VARIABLES:
	integer :: g ! gridcell index
	real(r8) :: available_volr ! volr available for withdrawal [m3]
	real(r8) :: max_demand_supported_by_volr ! [kg/m2] [i.e., mm]

	! ratio of demand_volr_limited to demand for each grid cell
	real(r8) :: dom_demand_limited_ratio_grc(bounds%begg:bounds%endg)
	... ! same for other sectors

	character(len=*), parameter :: subname = 'CalcSectorDemandVolrLimited'

	!-----------------------------------------------------------------------


	do g = bounds%begg, bounds%endg

		if (volr(g) > 0._r8) then
			available_volr = volr(g) * (1._r8 - this%params%sectorwater_river_volume_threshold)
			max_demand_supported_by_volr = available_volr/grc%area(g) * m3_over_km2_to_mm
		else
			! Ensure that negative volr is treated the same as 0 volr
			max_demand_supported_by_volr = 0._r8
		end if



		if (dom_demand(g) > max_demand_supported_by_volr) then
			dom_demand_limited_ratio_grc(g) = max_demand_supported_by_volr / dom_demand(g)
			liv_demand_limited_ratio_grc(g) = 0._r8
			elec_demand_limited_ratio_grc(g) = 0._r8
			mfc_demand_limited_ratio_grc(g) = 0._r8
			min_demand_limited_ratio_grc(g) = 0._r8
		else if (liv_demand(g) > (max_demand_supported_by_volr - dom_demand(g))) then
			dom_demand_limited_ratio_grc(g) = 1._r8
			liv_demand_limited_ratio_grc(g) = max_demand_supported_by_volr / liv_demand(g)
			elec_demand_limited_ratio_grc(g) = 0._r8
			mfc_demand_limited_ratio_grc(g) = 0._r8
			min_demand_limited_ratio_grc(g) = 0._r8
		else if (elec_demand(g) > (max_demand_supported_by_volr - dom_demand(g) - liv_demand(g))) then
			dom_demand_limited_ratio_grc(g) = 1._r8
			liv_demand_limited_ratio_grc(g) = 1._r8
			elec_demand_limited_ratio_grc(g) = max_demand_supported_by_volr/ elec_demand(g)
			mfc_demand_limited_ratio_grc(g) = 0._r8
			min_demand_limited_ratio_grc(g) = 0._r8
		else if (mfc_demand(g) > (max_demand_supported_by_volr - dom_demand(g) - liv_demand(g) - elec_demand(g))) then
			dom_demand_limited_ratio_grc(g) = 1._r8
			liv_demand_limited_ratio_grc(g) = 1._r8
			elec_demand_limited_ratio_grc(g) = 1._r8
			mfc_demand_limited_ratio_grc(g) = max_demand_supported_by_volr / mfc_demand(g)
			min_demand_limited_ratio_grc(g) = 0._r8
		else if (min_demand(g) > (max_demand_supported_by_volr - dom_demand(g) - liv_demand(g) - elec_demand(g) - mfc_demand(g))) then
			dom_demand_limited_ratio_grc(g) = 1._r8
			liv_demand_limited_ratio_grc(g) = 1._r8
			elec_demand_limited_ratio_grc(g) = 1._r8
			mfc_demand_limited_ratio_grc(g) = 1._r8
			min_demand_limited_ratio_grc(g) = max_demand_supported_by_volr / min_demand(g)
		else
			dom_demand_limited_ratio_grc(g) = 1._r8
			liv_demand_limited_ratio_grc(g) = 1._r8
			elec_demand_limited_ratio_grc(g) = 1._r8
			mfc_demand_limited_ratio_grc(g) = 1._r8
			min_demand_limited_ratio_grc(g) = 1._r8
		end if
	end do



	dom_demand_volr_limited(bounds%begg:bounds%endg) = 0._r8
	dom_consumption_volr_limited(bounds%begg:bounds%endg) = 0._r8
	... ! same for other sectors

	do g = bounds%begg, bounds%endg

		dom_demand_volr_limited(g) = dom_demand(g) * dom_demand_limited_ratio_grc(g)
		dom_consumption_volr_limited(g) = dom_consumption(g) * dom_demand_limited_ratio_grc(g)

		... ! same for other sectors
	end do
end subroutine CalcSectorDemandVolrLimited
end module SectorWaterMod

```

## Things to add later:
- Add a restart procedure
- For the moment we do not really make use of different start time for domestic/livestock and industrial fluxes (despite adding to params, and some specific procedures). The same goes for the duration of the withdrawls. In the future, we want to make domestic and livestock fluxes to be satisfied between 6AM-12PM (local time), and for industrial fluxes we assume the fluxes to be constant throughout the entire day.
- I also did an error in the `CalcSectorDemandVolrLimited` subroutine, when I compute the demand_limited_ratio_grc, I use as numerator `max_demand_supported_by_volr`. This is incorect, I need to update this depending on the withdrawal for previous sectors. Since irrigation is satisfied after sectoral, it means that I also need to do some update to the `IrrigationMod.F90` module. This can be done quite easily, we just need to add the `sectorwater_inst` as another input to the `CalcIrrigationNeeded()` subroutine and then update `max_demand_supported_by_volr` by taking into account all previous withdrawals and return flows.
- Also there are other things which I may need to consider in the future (e.g. when we will use the full input data which is 30 years for `hist` and another 90 years for `ssp-scenarios` experiments I cannot anymore rely on surfdata to read input)
- Another issue: I give as input to the subroutine checking water limitations depending on river volume the demand for the month. Instead I should give the demand for the day. 