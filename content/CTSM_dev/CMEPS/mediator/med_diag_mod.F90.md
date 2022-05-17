Tags: #CMEPS


Here are defined the `med_phases_diag_lnd()` and `med_phases_diag_rof()` subroutines which perform global diagnostics of input/output fluxes.

To support sectoral water usage, the withdrawal and return fluxes for each sector are added to the diagnostics from both land and rof side.

The sign of fluxes should be carefully accounted for. Negative sign mean that the system is losing water, while positive sign means it is gaining water.

## Code:
```fortran
! Example of code for the land diagnostic:

subroutine med_phases_diag_lnd( gcomp, rc)
...
call diag_lnd(is_local%wrap%FBImp(complnd,complnd), 'Flrl_dom_withd' , f_watr_roff, ic,&
         areas, lfrac, budget_local, minus=.true., rc=rc)
		 ! for withdrawal minus sign (as system lose water)
if (ChkErr(rc,__LINE__,u_FILE_u)) return

call diag_lnd(is_local%wrap%FBImp(complnd,complnd), 'Flrl_dom_rf' , f_watr_roff, ic,&
         areas, lfrac, budget_local, minus=.false., rc=rc)
if (ChkErr(rc,__LINE__,u_FILE_u)) return
		  ! for return flow plus sign (as system gains water)
...
end subroutine med_phases_diag_lnd( gcomp, rc)
```