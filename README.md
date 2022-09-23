# codas
This document covers the basics of downloading, compiling, running, and
analyzing results from the Constituent Data Assimilation (CoDAS) applied to the
Goddard Earth Observing System (GEOS) on the NASA Center for Climate Simulation
(NCCS) Discover supercomputer. You’ll need executables for the GEOS General
Circulation Model (GCM) and Gridpoint Statistical Interpolation (GSI), which
are provided for Discover here, but can be compiled from the GEOS Atmospheric Data
Assimilation (ADAS) fixture
([available here](https://github.com/GEOS-ESM/GEOSadas)),
a ship in a bottle, protractor, and a picture of Edith Piaf.

## Setting up a CoDAS run/experiment directory
The environment variable `$NOBACKUP` is pre-defined on Discover to point to your
nobackup directory.

1. Define your GEOS directory (`$GEOSDIR`). If you checked out and compiled the
    model following the instructions below, you can skip this step. Otherwise,
    for StratChem experiments, you can use
    ```
    setenv GEOSDIR /discover/nobackup/bweir/GEOS/bw_Icarus-3_2_p9_MEM_20-SLES12
    ```
 
    For carbon experiments, you can use
    ```
    setenv GEOSDIR /discover/nobackup/bweir/GEOS/bw_Heracles-5_4_p3_SLES12
    ```
    or any other existing code directory you have (must have the appropriate
    CoDAS hooks for applying increments).

2. Define the **reference** run directory (`$CLONEDIR`). Your run will be a
    clone of the run in this directory. For example, for StratChem experiments,
    use
    ```
    setenv CLONEDIR /discover/nobackup/bweir/GEOS/runs/sage_ana
    ```
    
    For carbon experiments, use
    ```
    setenv CLONEDIR /discover/nobackup/bweir/GEOS/runs/m2cc-v1_ana
    ```

3. Define your personal run root directory (`$EXPROOT`). This can be any
    writable directory, e.g.,
    ```
    setenv EXPROOT $NOBACKUP/GEOS/runs
    ```

4. Define your experiment (`$EXPID`). This can be anything, e.g.,
    ```
    setenv EXPID codas001
    ```

5. Run the CoDAS setup command:
    ```
    cd $GEOSDIR/GEOSagcm/src/Applications/GEOSgcm_App
    ./codas_setup --clone $CLONEDIR --root $EXPROOT --expid $EXPID --nocvs
    ```

Notes: This can have issues if `$CLONEDIR` points to a symbolic link instead of
the actual directory, or if `$CLONEDIR` is not the same run directory
referenced in the `gcm_run.j` script. Still looking into this.

The `codas_setup` utility places several hidden files in your home directory.
This will hopefully be fixed, but, in the meantime, you can change the GID by
editing the `~/.GROUProot` file.

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
    You should have restart files with only underscores, e.g.,
    `gocart_internal_rst`, and a `cap_restart` file that is 20090827 210000,
    corresponding to our restart time.
        
    These restarts may have a `codas_background_rst` file that is used to
    remember the background files at the beginning of the analysis window
    needed by the GSI. These files often don’t copy well from one experiment to
    another. If you’re having problems, deleting the `codas_background_rst`
    file and restarting is usually a good test.
        
3. Run CoDAS:
    ```
    sbatch ./codas_run.j
    ```
    It’s good practice to check the group ID in the run script because the
    `codas_setup` utility simply copies what’s in the `~/.GROUProot` file.

## Configuring CoDAS
The only difference between a CoDAS run directory and a GCM run directory is
the `codas_run.j` job script, the `GSIsa.x` executable, and a directory
`analyze` containing configuration files and utilities.

The basic configuration of CoDAS is laid out in `analyze/codas.rc`, which
defines the gasses analyzed, the GEOS chemistry components that provide them
(and receive the analysis increment), and various tuning parameters for the
background errors.

Most of the files in `analyze/etc` are either unused and needed by the GSI, or
generated automatically by the CoDAS utilities and have an extension `.tmpl`.
You can change the datasets assimilated by editing the `gsiparm.anl` file.
Tuning parameters for the observation errors are defined in the `tgasinfo`
file, which must have a line corresponding to every level of every observation
system defined in `gsiparm.anl`.

The background and observation error parameters are set at nominal values to
start. In science-grade products, these numbers are typically tuned based on a
posteriori statistics, viz., the Desroziers et al. (2005;
doi:[10.1256/qj.05.108](https://doi.org/10.1256/qj.05.108)) diagnostics,
three-cornered hat method (need link), and comparisons to (mostly) independent
data.

You can replace the call to the `GSIsa.x` executable with a call to the
`GSIsa.dbg.x` executable to turn on debugging (this is a separate compilation
that I try to keep up to date, but may not be). This will output a bunch of
diagnostic information and only process the first three observations of each
observation stream.

| File                       | Description |
| :------------------------- | :---------- |
| `GEOSgcm.x`                | GEOS executable |
| `GSIsa.x`                  | GSI executable |
| `GSIsa.dbg.x`              | Debugging GSI executable |
| `codas_run.j`              | CoDAS run script |
| `analyze`                  | Directory for CoDAS settings and utilities |
| `analyze/codas.rc`         | Main CoDAS configuration file: species, chemistry component, background error covariance |
| `analyze/ana2inc.x`        | Subtracts background from analysis to compute increment |
| `analyze/codas_acqobs.j`   | Template job for acquiring obs |
| `analyze/codas_setup1.csh` | Does the first part of generating the etc files |
| `analyze/codas_setup2.csh` | Does the second part of generating the etc files |
| `analyze/etc/obsys.rc`     | Tells codas_acqobs.j where to find obs |
| `analyze/etc/tgasinfo`     | Additional observational settings for GSI: obs error inflation, bias, gross check |
| `analyze/etc/gsiparm.anl`  | Additional observational settings for GSI: obs type (e.g., tgav, tgaz, tgez, tgev) |
| `analyze/etc/gsidiags.rc`  | Additional observational settings for GSI (should be able to auto-generate) |

## Data types
CoDAS, with some exceptions, processes data in a generic data format. This format
puts data into four categories based on if the observation is a point sample or
uses an averaging kernel and if it is on an altitude grid or a pressure grid. A
two letter code after `tg` specifies these options (see table below).

| Abbrev | Data type |
| :----- | :-------- |
| tgav   | Averaging kernel on a pressure grid |
| tgaz   | Averaging kernel on an altitude grid |
| tgez   | Sample observation on an altitude grid |
| tgev   | Sample observation on a pressure grid |

Very many constituent retrievals can be downloaded and transformed into these
formats with the xtralite utilitiy
([available here](https://github.com/briardew/xtralite)).

### Averaging kernels
Most remote-sensing instruments are sensitive to an integrated path in the
atmosphere (e.g., from the sun to the surface to the satellite). Skipping over
several important steps, we can view the observation opeator as
$$H(x^m) = y^a + A(x^m - x^a)$$
Add Connor and Rodgers citations.

## Adding new observations
To add a new dataset, you’ll need to add entries in `obsys.rc`, `tgasinfo`,
`gsiparm.anl`, and `gsidiags.rc`. The easiest approach is to use existing
entries as a guide.

## Outputs
Most fields are typical GCM outputs but with assimilation. Assimilation-specific collections are

| `bkg.sfc`  | 2D background fields for the surface (met only) |
| `bkg.eta`  | 3D background fields for the atmosphere (met only) |
| `cbkg.eta` | 3D background fields for the atmosphere (constituents) |
| `ana.eta`  | 3D analysis fields (all analyzed variables from met + tracers) |

The `diag_*.bin` files are temporary and used to construct ODS files.

### ODS files
ODS files are a special output of GSI showing per-sounding values and
metadata, and are thus not gridded. These files are netCDF and named ODS due
to their extension (if this makes no sense, you're on the right track).
Soundings are laid out in two-dimensional blocks that can be reshaped into
column-major? vectors. Make sure to use the `qcexcl` flag: `0` means obs was
assimilated, `2` means it was rejected for assimilation but still deemed good
by the data provider, and `-127` means it was not considered for assimilation.

## Downloading and compiling CoDAS (optional)
**This is outdated, keeping here as a reminder to update to git.**

If you wish to modify the CoDAS code, you’ll need to download (check out) and
compile a given version (tag) of the GSI and optionally a separate GCM.
**You’ll only need to do this once**. If you’ve never used git/CVS before,
you’ll need to follow the steps in the GEOS Quick Start Guide (follow this link
for Heracles 5.4). The directions below should work, but use an outdated CVS
repository instead of the newer GitHub repositories.

You can download and compile the GCM anywhere, but here we’ll put it in the
directory `$GEOSDIR`, which we’ll come back to in the next section. Recall from
before that we defined the `$GCMTAG` and `$GEOSDIR` environment variables. For
example, for StratChem experiments,
```
setenv GCMTAG bw_Icarus-3_2_p9_MEM_20-SLES12
setenv GEOSDIR $NOBACKUP/GEOS/$GCMTAG
```
For carbon experiments, do the same, but with
```
setenv GCMTAG bw_Heracles-5_4_p3_SLES12
setenv GEOSDIR $NOBACKUP/GEOS/$GCMTAG
```
Then run
```
mkdir -p $GEOSDIR
cd $GEOSDIR
cvs co -r $GCMTAG GEOSagcm
cd GEOSagcm/src
./parallel_build.csh
```

# Any questions?

Future improvements
* Auto-generate `gsidiags.rc` (should be “easy”)
* Auto-generate `gsiparm.anl` (“medium”?)
* Convert run script to Python (cf. other efforts)?
* Convert `codas.rc` to YAML?
* Generalize `codas.rc` to auto-generate remainder of etc files
