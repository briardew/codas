# codas
This document covers the basics of downloading, compiling, running, and analyzing
results from the Constituent Data Assimilation (CoDAS) applied to the Goddard Earth Observing
System (GEOS) on the NASA Center for Climate Simulation (NCCS) Discover supercomputer. You’ll
need executables for the GEOS General Circulation Model (GCM) and Gridpoint Statistical
Interpolation (GSI), usually compilable from a GEOS Atmospheric Data Assimilation (ADAS)
tag [link here], the CoDAS setup utility, a ship in a bottle, protractor, and a picture
of Edith Piaf.

## Setting up a CoDAS run/experiment directory
The environment variable $NOBACKUP is pre-defined on Discover to point to your nobackup directory.

1. Define your GEOS directory (```$GEOSDIR```). If you checked out and compiled the model
    following the instructions below, you can skip this step. Otherwise, for StratChem
    experiments, you can use
    ```
    setenv GEOSDIR /discover/nobackup/bweir/GEOS/bw_Icarus-3_2_p9_MEM_20-SLES12
    ```
    
    For carbon experiments, you can use
    ```
    setenv GEOSDIR /discover/nobackup/bweir/GEOS/bw_Heracles-5_4_p3_SLES12
    ``` or any other existing code directory you have (must have the appropriate CoDAS hooks for applying increments).
    
2. Define the **reference** run directory (```$CLONEDIR```). Your run will be a clone of the
    run in this directory. For example, for StratChem experiments, use
    ```
    setenv CLONEDIR /discover/nobackup/bweir/GEOS/runs/sage_ana
    ```
    or for carbon experiments, use
    ```
    setenv CLONEDIR /discover/nobackup/bweir/GEOS/runs/m2cc-v1_ana
    ```
3. Define your personal run root directory (```$EXPROOT```). This can be any writable directory, e.g.,
    ```
    setenv EXPROOT $NOBACKUP/GEOS/runs
    ```
4. Define your experiment (```$EXPID```). This can be anything, e.g.,
    ```
    setenv EXPID codas001
    ```
5. Run the CoDAS setup command:
    ```
    cd $GEOSDIR/GEOSagcm/src/Applications/GEOSgcm_App
    ./codas_setup --clone $CLONEDIR --root $EXPROOT --expid $EXPID --nocvs
    ```

Notes: This can have issues if ```$CLONEDIR``` points to a symbolic link instead of the actual directory,
or if ```$CLONEDIR``` is not the same run directory referenced in the gcm_run.j script. Still looking into this.

The ```codas_setup``` utility places several hidden files in your home directory. This will hopefully be fixed,
but, in the meantime, you can change the GID by editing the ```~/.GROUProot``` file.

## Running CoDAS
1. Go to your run directory:
    ```
    cd $EXPROOT/$EXPID
    ```
    
2. Copy restarts into your run directory. For example
    ```
    tar xf /archive/u/bweir/GEOS5.0/m2cc-v1_tags/restarts/Y2009/restarts.e20090827_21z.tar
    /home/bweir/codas/utils/striprst.sh
    ```
    You should have restart files with only underscores, e.g., ```gocart_internal_rst```, and a ```cap_restart```
    file that is 20090827 210000, corresponding to our restart time.
        
    These restarts may have a ```codas_background_rst``` file that is used to remember the background files at the beginning of the analysis window needed by the GSI. These files often don’t copy well from one experiment to another. If you’re having problems, deleting the ```codas_background_rst``` file and restarting is usually a good test.
        
3. Run CoDAS:
    ```
    sbatch ./codas_run.j
    ```
    It’s good practice to check the group ID in the run script because the ```codas_setup``` utility simply copies what’s in the ```~/.GROUProot``` file.
