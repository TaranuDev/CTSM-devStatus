Tags: #CMEPS

## Overview:
In this module we have defined the `esmFldsExchange_cesm` subroutine.

To support sectoral water usage we add the import fields for each sector withdrawal and return flow which are to be transferred from the `land` to `routing` component.

## Code:
```fortran
...
! example of code for domestic withdrawal:

    ! ---------------------------------------------------------------------
    ! to rof: domestic withdrawal flux from land (withdrawal from rivers)
    ! ---------------------------------------------------------------------
    if (phase == 'advertise') then
       call addfld(fldListFr(complnd)%flds, 'Flrl_dom_withd')
       call addfld(fldListTo(comprof)%flds, 'Flrl_dom_withd')
    else
       if ( fldchk(is_local%wrap%FBImp(complnd, complnd), 'Flrl_dom_withd', rc=rc) .and. &
            fldchk(is_local%wrap%FBExp(comprof)         , 'Flrl_dom_withd', rc=rc)) then
          call addmap(fldListFr(complnd)%flds, 'Flrl_dom_withd', comprof, mapconsf, 'lfrac', lnd2rof_map)
          call addmrg(fldListTo(comprof)%flds, 'Flrl_dom_withd', &
               mrg_from=complnd, mrg_fld='Flrl_dom_withd', mrg_type='copy_with_weights', mrg_fracname='lfrac')
       end if
    end if
...
```