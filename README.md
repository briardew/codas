# codas
Constituent Data Assimilation System (CoDAS)

For the moment, see the [CoDAS Quick Start Guide](https://docs.google.com/document/d/1sJCxe_Z5JTAU28Yfen4U7j75EIMQnNhmLModc4ZN1ek/edit?usp=sharing). This will be copied over here eventually.

This document covers the basics of downloading, compiling, running, and analyzing results from the Constituent Data Assimilation (CoDAS) applied to the Goddard Earth Observing System (GEOS) on the NASA Center for Climate Simulation (NCCS) Discover supercomputer. You’ll need executables for the GEOS General Circulation Model (GCM) and Gridpoint Statistical Interpolation (GSI), usually compilable from a GEOS Atmospheric Data Assimilation (ADAS) tag [link here], the CoDAS setup utility, a ship in a bottle, protractor, and a picture of Edith Piaf.

## Setting up a CoDAS run/experiment directory
The environment variable $NOBACKUP is pre-defined on Discover to point to your nobackup directory.

1. Define your GEOS directory (```$GEOSDIR```). If you checked out and compiled the model following the instructions below, you can skip this step. Otherwise, for StratChem experiments, you can use the build in
    ```
    setenv GEOSDIR /discover/nobackup/bweir/GEOS/bw_Icarus-3_2_p9_MEM_20-SLES12
    ```
For carbon experiments, you can use the build in
    ```
    setenv GEOSDIR /discover/nobackup/bweir/GEOS/bw_Heracles-5_4_p3_SLES12
    ```
or any other existing code directory you have (must have the appropriate CoDAS hooks for applying increments).
