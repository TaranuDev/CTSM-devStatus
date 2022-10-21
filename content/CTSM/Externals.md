---
title: "Externals.cfg"
tags:
- CTSM
enableToc: false # do not show a table of contents on this page
---

## General discussion:
To add the sectoral water abstractions, changes were made to several components of the CESM model (see [CTSM/Model Development for Sectoral Water Usage](Model_Development_for_Sectoral_Water_Usage.md)).

In order to run the model using the new functionality, one will have to use the right version of `CTSM`, `CMEPS`, `CPL7` and `MOSART`.

At the moment, we didn't requested a pull request with the main model repository. Therefore, we update the Externals.cfg, so that it point to the [TaranuDev github](https://github.com/TaranuDev) to access the right versions of the forked model repositories.

This way the `./manage_externals/checkout_externals` command will install the components which support sectoral water abstractions.

The user may wish to read the [Tutorial: how to run CESM with active human water abstractions](Tutorials/tutorial_run_new_model.md) to see an example of model installation and run.