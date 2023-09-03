# CoDAS 
This document covers the basics of downloading, compiling, running, and
analyzing results from the Constituent Data Assimilation (CoDAS) applied to the
Goddard Earth Observing System (GEOS) on the NASA Center for Climate Simulation
(NCCS) Discover supercomputer. You’ll need executables for the GEOS General
Circulation Model (GCM) and Gridpoint Statistical Interpolation (GSI), which
are provided for Discover in [the Quickstart](#quickstart-on-discover), but can be
compiled from the GEOS Atmospheric Data Assimilation (ADAS) fixture
([available here](https://github.com/GEOS-ESM/GEOSadas)).

## Contents
1. [Quickstart on Discover](#quickstart-on-discover)
2. [Configuring CoDAS](#configuring-codas)
3. [Data types](#data-types)
4. [Adding new observations](#adding-new-observations)
5. [Outputs](#outputs)
6. [Downloading and compiling CoDAS](#downloading-and-compiling-codas)

## Quickstart on Discover
The environment variable `$NOBACKUP` is pre-defined on Discover to point to your
nobackup directory. Right now, this document assumes you are using a C shell
variant, which is the default on Discover. It will be transitioned to Bash-like
shells in the near future.

### Cloning an existing run

1. Define your GEOS directory (`$GEOSDIR`). If you checked out and compiled the
    model following the instructions below (we need to add that section!), you can skip this step.
    ```
    setenv GEOSDIR /discover/nobackup/bweir/GEOS/GEOSadas-5_30_0/GEOSadas
    ```

2. Define the **reference** run directory (`$CLONEDIR`). Your run will be a
    copy (i.e., clone) of the run in this directory. For example, for carbon experiments, use
    ```
    setenv CLONEDIR /discover/nobackup/bweir/GEOS/runs/carbon-ng_ana
    ```
    Note: The variable `$CLONEDIR` must point to the directory referenced in the `codas_run.j`
    script, not a symbolic link.

3. Define your personal run root directory (`$EXPROOT`). This can be any
    writable directory, e.g.,
    ```
    setenv EXPROOT $NOBACKUP/GEOS/runs
    ```

4. Define your experiment (`$EXPID`). This can be anything, e.g.,
    ```
    setenv EXPID codas001
    ```

6. Run the CoDAS setup command (this command will copy CoDAS into the above-defined path):
    ```
    cd $GEOSDIR/install/bin
    ./codas_setup --clone $CLONEDIR --root $EXPROOT --expid $EXPID
    ```

The `codas_setup` utility places several hidden files in your home directory.
This will hopefully be fixed, but, in the meantime, you can change the GID (GID is what?) by
editing the `~/.GROUProot` file.

**The cloning utility really doesn't like variables used in the definitions of `$HOMDIR` and
`$EXPDIR` in `codas_run.j`. Thinking about solutions to this, but there's only so much
inference the cloning utility can do.**

### Running CoDAS
The following assumes you've defined the environment variables `$EXPROOT`, `$EXPID`, and
`$CLONEDIR` as in [the cloning guide](#cloning-an-existing-run).

1. Go to your run directory:
    ```
    cd $EXPROOT/$EXPID
    ```

2. Copy restarts into your run directory. For example,
    ```
    tar xf $CLONEDIR/restarts/restarts.e20180505_21z.tar
    ./analyze/bin/striprst.sh
    ```
    You should have restart files with only underscores, e.g.,
    `gocart_internal_rst`, and a `cap_restart` file that is 20180505 210000,
    corresponding to our restart time.
        
    These restarts may have a `codas_background_rst` file that is used to
    remember the background files at the beginning of the analysis window
    needed by the GSI. These files sometimes don’t copy well from one experiment to
    another. If you’re having problems, this can be checked by temporarily renaming
    the `codas_background_rst` file and restarting. This restart is necessary
    for zero-diff results.
        
4. Run CoDAS:
    ```
    sbatch ./codas_run.j
    ```
    It’s good practice to check the group ID in the run script because the
    `codas_setup` utility simply copies what’s in the `~/.GROUProot` file.

## Configuring CoDAS
The only difference between a CoDAS run directory and a GEOS GCM run directory
is the `codas_run.j` job script, the `GSIsa.x` executable, and a directory
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
| tgez   | Point sample on an altitude grid |
| tgev   | Point sample on a pressure grid |
| tgaz   | Averaging kernel on an altitude grid |
| tgav   | Averaging kernel on a pressure grid |

Very many constituent retrievals can be downloaded and transformed into these
formats with the xtralite utilitiy
([available here](https://github.com/briardew/xtralite)).

### Averaging kernels
Most remote-sensing instruments are sensitive to an integrated path in the
atmosphere (e.g., from the sun to the surface to the satellite). Skipping over
several important steps, we can view the observation opeator as
$$H(x) = y_0 + A(x - x_0),$$
where $x_0$ and $y_0$ are the a priori profile and observation values.

Add Rodgers and Connor and other citations.

## Adding new observations
To add a new dataset, you’ll need to add entries in `obsys.rc`, `tgasinfo`,
`gsiparm.anl`, and `gsidiags.rc`. The easiest approach is to use existing
entries as a guide.

## Outputs
Most fields are typical GCM outputs but with assimilation. Assimilation-specific collections are

| Collection | Description |
| :--------- | :--- |
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

## Downloading and compiling CoDAS

**This is optional on Discover as compiled executables are already available.**

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

## Any questions?

Future improvements
* Auto-generate `gsidiags.rc` (should be “easy”)
* Auto-generate `gsiparm.anl` (“medium”?)
* Convert run script to Python (cf. other efforts)?
* Convert `codas.rc` to YAML?
* Generalize `codas.rc` to auto-generate remainder of etc files
