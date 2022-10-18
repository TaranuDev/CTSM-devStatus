---
title: "seq_diag_mct.F90"
tags:
- CPL7
enableToc: false # do not show a table of contents on this page
---
## Overview:

## Code:
```fortran
! Add to the seq_diag_lnd_mct() and seq_diag_rof_mct() diagnostics subroutines the fields corresponding to the withdrawal and return flow of sectoral water usage.

! This is required to make sure that no error is comitted during the exchange of information between the land and routing model.

! It is important to be careful with the sign
! For example for land to coupler budget:
if (index_l2x_Flrl_dom_withd /= 0) then
nf = f_wroff ; budg_dataL(nf,ic,ip) = budg_dataL(nf,ic,ip) - ca_l*l2x_l%rAttr(index_l2x_Flrl_dom_withd,n) + ca_l*l2x_l%rAttr(index_l2x_Flrl_dom_rf,n)
end if

! and for the coupler to routing budget:
if (index_x2r_Flrl_dom_withd /= 0) then
nf = f_wroff; budg_dataL(nf,ic,ip) = budg_dataL(nf,ic,ip) + ca_r*x2r_r%rAttr(index_x2r_Flrl_dom_withd,n) - ca_r*x2r_r%rAttr(index_x2r_Flrl_dom_rf,n)
end if

```