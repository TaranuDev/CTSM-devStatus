---
title: "RtmHistFlds.F90"
tags:
- MOSART
- Routing
enableToc: false # do not show a table of contents on this page
---

## Overview:
For each sector we add 3 new outputs from the routing model.
These are: 
- QsectorX_WITHD_FROM_COUPLER
- QsectorX_RF_FROM_COUPLER
- QsectorX_ACTUAL

In general QsectorX_WITHD_FROM_COUPLER already corresponds to an actual withdrawal, since river volume is also accounted in the `land component`. But since the coupling period between the `land` and `routing` component do not happen at each step, there is the possibility that the land model will use more water than actually available in the grid river network. So, an additional volume check is done in the routing model and if indeed the withdrawal from the coupler is higher than available river water, than new actual withdrawal is computed as seen by the routing model (while the difference with coupler is sent to the ocean).

These 3 variables are enough to deduce over variables of interest. For example we can compute:
QDOM_ACTUAL_RF = (QDOM_ACTUAL/QDOM_WITHD_FROM_COUPLER) x QDOM_RF_FROM_COUPLER

## Code:
```fortran
subroutine RtmHistFldsInit()
...
call RtmHistAddfld (fname='QDOM_WITHD_FROM_COUPLER', units='m3/s',  &
                 avgflag='A', long_name='Amount of water withdrew for domestic usage', &
              ptr_rof=rtmCTL%qdom_withd, default='inactive')
     
call RtmHistAddfld (fname='QDOM_RF_FROM_COUPLER', units='m3/s',  &
                 avgflag='A', long_name='Amount of water returned from domestic usage', &
              ptr_rof=rtmCTL%qdom_rf, default='inactive')
			  
call RtmHistAddfld (fname='QDOM_ACTUAL', units='m3/s',  &
                 avgflag='A', long_name='Actual domestic usage (if limited by river storage)', &
              ptr_rof=rtmCTL%qdom_actual, default='inactive')
...     
end subroutine RtmHistFldsInit
```

