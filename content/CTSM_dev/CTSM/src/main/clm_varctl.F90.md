Tags: #CTSM #Main 

## Overview:
This module contains the run control variables.

To support sectoral water usage we add the `sectorwater` logic variable.
The default value is `.false.` meaning that by default sectoral water usage is not done (this can be changed in clm user namelist after case setup)
## Code:
```fortran

!----------------------------------------------------------
  ! Sector water usage logic
  !----------------------------------------------------------
  
  ! do not satisfy sector water demand by default
  logical, public :: sectorwater = .false.        

```