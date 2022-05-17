Tags: #MOSART #Routing 

## Overview:
Small changes to the `runoff_flow` type for support of sectoral water fluxes.
Also allocate the memory and initialize variables with default values (zeros).

## Code:
```fortran
! no need to show the code as it is very simple
! it is here that the runoff_flow type is defined (rtmCTR is the instance of
! this type)

! so we just add: qsectorX_withd(:), qsectorX_rf(:) and qsectorX_actual(:)
! variables to the object as well as allocate the memory and initialize with
! default values
```