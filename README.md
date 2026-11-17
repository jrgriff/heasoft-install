# HEASoft Installation (macOS)

This document provides instructions on how to install [HEASoft](https://heasarc.gsfc.nasa.gov/docs/software/heasoft/), NASA's software package for high-energy astrophysics, on a Mac.  HEASoft includes the FTOOLS, __Xspec__, __XSTAR__, __fv__, and the analysis tools for missions such as Swift, NICER, NuSTAR and XRISM.  It is installed with conda, following [HEASARC's own conda instructions](https://heasarc.gsfc.nasa.gov/docs/software/conda.html).

The steps are the same for Apple Silicon (M-series) and Intel Macs.  You will need __macOS 11 (Big Sur) or newer__ and about __4 GB__ of free disk space.

## Table of contents
1. [conda/mamba](#condamamba)
2. [HEASoft](#heasoft)
3. [Checking your installation](#checking-your-installation)
4. [XQuartz (plot windows)](#xquartz-plot-windows)
5. [Xspec model data files](#xspec-model-data-files)
6. [XSTAR](#xstar)
7. [Calibration database (CALDB)](#calibration-database-caldb)
8. [Updating and removing HEASoft](#updating-and-removing-heasoft)
9. [Troubleshooting](#troubleshooting)
10. [Additional documentation](#additional-documentation)


## conda/mamba
This installation is done in the __Terminal__ app (in __Applications__ → __Utilities__).  *`$` indicates a terminal prompt; you do not need to type the `$`, only the command that follows. The __Copy__ button on each box copies just the commands.*

For the purposes of this guide, you will be installing [Miniforge](https://github.com/conda-forge/miniforge), a lightweight version of conda that also provides [`mamba`](https://mamba.readthedocs.io/en/latest/), a faster version of conda.

*If you already have conda (for example, from another course), you can skip to [HEASoft](#heasoft).  To check, run `conda --version`; if it prints a version number rather than `command not found`, you have it.*

**To install [Miniforge](https://github.com/conda-forge/miniforge#unix-like-platforms-macos-linux--wsl):**

```console
$ curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
$ bash Miniforge3-$(uname)-$(uname -m).sh
```

The installer asks a few questions:

1. __Please, press ENTER to continue__: press __Return__, then press the __space bar__ to page through the license.
2. __Do you accept the license terms?__: type `yes`.
3. __Press ENTER to confirm the location__: press __Return__ to accept the default.
4. __Proceed with initialization?__: type `yes`.  *The default here is __no__, so be sure to type `yes`; this is what makes the `conda` command work in new Terminal windows.*

Close and re-open Terminal afterwards.  Your prompt should now begin with `(base)`.

## HEASoft
**To create the `heasoft` environment** (this is one long command split across three lines; copy and paste it as a whole):

```console
$ mamba create --name heasoft heasoft \
    --channel https://heasarc.gsfc.nasa.gov/FTP/software/conda/ \
    --channel conda-forge
```

mamba will list the packages it is going to install and then download about 500 MB, which can take several minutes.  HEASoft comes from HEASARC's own conda channel, and everything else it needs (Python, astropy, matplotlib, etc.) comes from conda-forge.

**To activate the `heasoft` environment:**

```console
$ conda activate heasoft
```

Your prompt will change to start with `(heasoft)`, and HEASoft is set up automatically; there is no separate `heainit` step.  To use HEASoft in the future (for example, if you have just opened a new Terminal window), type `conda activate heasoft`

*The environment also includes Python with astropy, numpy, scipy, matplotlib and `heasoftpy` (a Python interface to HEASoft), so you can use HEASoft from Python scripts as well.*

## Checking your installation
With the `heasoft` environment active, check the HEASoft version:

```console
$ fversion
```

This should print something like `02Sep2026_V6.37.1`.  Then start __Xspec__:

```console
$ xspec
```

Xspec prints a banner and leaves you at an `XSPEC>` prompt.  Type `exit` to leave it.

## XQuartz (plot windows)
Xspec and several other HEASoft tools draw their plot windows using X11, which macOS does not include.  **To see plot windows**, install [XQuartz](https://www.xquartz.org): download the `.pkg` file from <https://www.xquartz.org>, open it and follow the installer.

*If you use [Homebrew](https://brew.sh), you can instead run `brew install --cask xquartz`.*

*After installing XQuartz for the first time you must log out and back in (or restart your Mac).*

**To test it**, start `xspec` and type these commands at the `XSPEC>` prompt.  After `model powerlaw`, press __Return__ twice to accept the default parameter values:

```text
cpd /xw
model powerlaw
dummyrsp 0.3 10
plot model
```

A window with a plot of a power-law model should open.  Type `exit` to leave Xspec.

## Xspec model data files
Some Xspec models (for example `apec`, `mekal` and `kerrbb`) need data files that are not included with HEASoft.  These are downloaded with `ftgetmodeldata`, and you only need to download files for the models you actually use.

**To download the files for a model**, here `apec`:

```console
$ conda activate heasoft
$ ftgetmodeldata apec latest
```

The files are stored inside the `heasoft` environment, where Xspec finds them automatically.

*Some of these files are large (the `apec` files alone are about 1.3 GB).  To see what would be downloaded without downloading anything, add `dryrun=yes` to the end of the command.  Downloading files for every model (`ftgetmodeldata all latest`) takes over 20 GB, so avoid it unless you need it.  The list of models that need data files is at <https://heasarc.gsfc.nasa.gov/FTP/software/xspec/spectral/modelDataFiles.csv>.*

## XSTAR
[XSTAR](https://heasarc.gsfc.nasa.gov/docs/software/xstar/xstar.html) needs its own data files, which come in a separate package (about 450 MB).  You only need this if you plan to use XSTAR.

**To add the XSTAR data** to the `heasoft` environment:

```console
$ mamba install --name heasoft xstar-data \
    --channel https://heasarc.gsfc.nasa.gov/FTP/software/conda/ \
    --channel conda-forge
```

## Calibration database (CALDB)
Mission analysis tools (for example, those for Swift, NICER or NuSTAR) look up instrument calibration files in the [CALDB](https://heasarc.gsfc.nasa.gov/docs/heasarc/caldb/caldb_intro.html).  The CALDB is very large, so instead of downloading it you can let HEASoft fetch just the files it needs from HEASARC as it goes.  *You can skip this section if you only use Xspec with spectra that have already been prepared for you.*

**To set up remote CALDB access**, first download two small configuration files into a `caldb` folder in your home folder:

```console
$ mkdir -p ~/caldb
$ cd ~/caldb
$ curl -O https://heasarc.gsfc.nasa.gov/FTP/caldb/software/tools/caldb.config
$ curl -O https://heasarc.gsfc.nasa.gov/FTP/caldb/software/tools/alias_config.fits
$ cd ~
```

Then tell the `heasoft` environment where to find the CALDB.  These settings are saved with the environment and take effect every time you activate it:

```console
$ conda activate heasoft
$ conda env config vars set CALDB=https://heasarc.gsfc.nasa.gov/FTP/caldb CALDBCONFIG=$HOME/caldb/caldb.config CALDBALIAS=$HOME/caldb/alias_config.fits
$ conda deactivate
$ conda activate heasoft
```

*`conda` may print a warning about "overwriting environment variables" when you re-activate; this is expected the first time.*

**To check it**, ask the CALDB about one instrument (Swift XRT is used here as an example):

```console
$ caldbinfo INST SWIFT XRT
```

It should finish with `CALDB appears to be set-up & accessible`.  This needs an internet connection, and so does any tool that reads calibration files this way.

## Updating and removing HEASoft
**To update HEASoft** to the newest release:

```console
$ mamba update --name heasoft heasoft \
    --channel https://heasarc.gsfc.nasa.gov/FTP/software/conda/ \
    --channel conda-forge
```

If you downloaded Xspec model data files, run `ftgetmodeldata` again after updating to pick up any newer versions.

**To remove HEASoft completely**, deactivate the environment and remove it:

```console
$ conda deactivate
$ conda env remove --name heasoft
```

This removes HEASoft and any model data files stored inside the environment.  It does not remove Miniforge or the `~/caldb` folder.

## Troubleshooting
**`conda: command not found`**: conda was not set up for your shell during installation (the __Proceed with initialization?__ question above).  Run the following, then close and re-open Terminal:

```console
$ ~/miniforge3/bin/conda init zsh
```

**HEASoft commands such as `xspec` or `fversion` are not found**: make sure the prompt starts with `(heasoft)`; if not, run `conda activate heasoft`.  If it does and the commands are still missing, deactivate and re-activate the environment:

```console
$ conda deactivate
$ conda activate heasoft
```

**Xspec says it cannot open the plot device, or no window appears**: install [XQuartz](#xquartz-plot-windows), and make sure you logged out and back in afterwards.

**HEASoft runs slowly on an Apple Silicon Mac**: you may be using an older Intel version of conda (for example, an old Anaconda installation) that runs under Rosetta.  Run `conda info` and look at the __platform__ line; on an M-series Mac it should say `osx-arm64`.  If it says `osx-64`, install Miniforge as described [above](#condamamba) and create the `heasoft` environment again with it.

**Using HEASoft together with Fermitools**: installing both in the same environment can cause conflicts.  Install Fermitools first and then HEASoft, or keep them in separate environments.

*HEASARC describes its conda packages as experimental.  If you run into an installation problem not covered here, the [HEASARC Help Desk](https://heasarc.gsfc.nasa.gov/cgi-bin/ftoolshelp) can help.*

## Additional documentation

- [HEASARC: Installing HEASoft using Conda](https://heasarc.gsfc.nasa.gov/docs/software/conda.html)
- [HEASoft documentation](https://heasarc.gsfc.nasa.gov/docs/software/heasoft/)
- [Xspec manual](https://heasarc.gsfc.nasa.gov/docs/software/xspec/manual/XspecManual.html)
- [Accessing the CALDB remotely](https://heasarc.gsfc.nasa.gov/docs/heasarc/caldb/caldb_remote_access.html)
- [Getting started with the Mac Terminal](https://support.apple.com/guide/terminal/welcome/mac)
- [Managing conda environments](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html)
