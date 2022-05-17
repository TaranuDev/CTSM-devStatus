Tags: #CTSM

## Overview:
To support sectoral water usage, changes were made to several components (see [[Model Development for Sectoral Water Usage]]).

Therefore to be able to run simulations supporting sectoral water usage it is required to use the right version of `CMEPS`, `CPL7` and `MOSART`.

We change the Externals.cfg, so that it point to the TaranuDev github where the updated forks of the before-mentioned components are available. This way the `./manage_externals/checkout_externals` command will install the components which support sectoral water usage.  