src/cpl/nuopc/lnd_import_export.F90 (done)

src/cpl/lilac/lnd_comp_esmf.F90 (done)

src/cpl/lilac/lnd_import_export.F90 (done)

src/cpl/mct/clm_cpl_indices.F90 (done)

src/cpl/mct/lnd_import_export.F90 (done)

components/cpl7/driver/main/CMakeLists.txt (done)

cime/CIME/non_py/src/components/xcpl_comps_nuopc/xrof/src/rof_comp_nuopc.F90 (done)

cime/CIME/non_py/src/components/xcpl_comps_nuopc/xlnd/src/lnd_comp_nuopc.F90 (done)

Things I tried to correct the bug with NaNs from CLM to the coupler:
1. Instead of initializing with NaNs or spval, I initialized all fields to 0 in the SectorWater module (didn't work).