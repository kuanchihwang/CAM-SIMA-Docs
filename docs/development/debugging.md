# Debugging techniques
Start with the [CAM wiki](https://github.com/ESCOMP/CAM/wiki/CAM-debugging-techniques).

## CAM-SIMA-specific debugging
### Build errors
Debugging tips if you get build errors:

- If the output indicates that the error message or failure is coming from somewhere within $CAM-SIMA/ccpp_framework:
    - If you're getting a clear error message, it's likely that you have something wrong with your metadata
    - If you're getting an error message that indicates that something is breaking in the framework code itself (something went uncaught) - consult the AMP SEs
- If the error happens during the atm build, you can see the full output of the atm build in the build log here: `bld/atm.bldlog.*`

### Run-time errors

- Start with the atm.log* - if the issue occurred during the execution of the CAM code, it will hopefully have a clear and concise error message
- Move to the cesm.log* - it will hopefully include a stack trace for the error in question; if the error did not occur in the CAM code (or CAM did not properly trap the error), it will help you identify the source of the issue.
- If neither log file contains helpful information, a few first steps:
    - Resubmit the case; it could be a machine hiccup
    - Turn on DEBUG mode (if it's not on already) and rebuild/rerun
    - Look in your run directory for any log files called `PETXXX` - if there was an issue on the [ESMF](https://earthsystemmodeling.org/docs/release/latest/ESMF_usrdoc/) side of things, it will show up in one of these (there will be one PET file per processor)
    - Try a different compiler - maybe it'll give you a more helpful error message
    - set NTASKS=1 (`./xmlchange NTASKS=1`), do a clean rebuild (as instructed), and run again; maybe running in serial will identify the error
    - Look for the `***************** HISTORY FIELD LIST ******************` in the atm.log* file; if it's not there, the error occurred at init time
        - If the error occurred during init time, try a new case with a different grid and/or dycore
        - If the model ran for a few timesteps before dying (look for the `CAM-SIMA time step advanced` message in the atm.log* file), it's likely that one or more variable that you introduced or modified has gone off the rails (value has become very large or very small or zero)
            - Update your user_nl_cam to output all possible suspected variables to a history file at some point shortly before the model dies, then inspect the output to see if any are obviously wrong
        - If the model completed all timesteps, try running a shorter case to see if the problem persists; if so, it's an error during the model finalization
    - Run the [TotalView](#totalview) debugger on izumi
    - Use the old standard - print statements - to narrow down where the code is stopping
    - Ask for help!

### Unexpected answer changes

- Two paths here:
    - You're getting unexpected DIFFs from the regression testing
        - Consult with a scientist about whether differences are expected and for which configurations (compsets, resolutions, namelists parameters, etc)
        - If the differences are very small (look like round-off), consult with the other AMP SEs on whether we're ok with this
        - If the differences are indeed unexpected and larger than round-off, create a case using the code from the head of `development` and:
            - place print statements in both code bases (your development branch and the head of `development`) to identify where the numbers are going awry OR
            - run the [TotalView](#totalview) debugger OR
            - use the comparison tool described below (`$CAM-SIMA/tools/find_max_nonzero_index.F90`)
    - You're getting unexpected answer changes compared with CAM
        - Consult with other AMP SEs about whether the differences appear to be due to round-off error
        - Use the comparison tool (LINK ONCE IT EXISTS): `$CAM-SIMA/tools/find_max_nonzero_index.F90`
            - This tool can help you narrow down where the issue begins by printing out values at a specific index and comparing those with the "truth" (from CAM)

### TotalView

- Grab an interactive node. You can do this by copying the following commands into a .csh script:

```
#! /bin/csh -f
#PBS -q long
# Number of nodes (CHANGE THIS if needed)
# #PBS -l walltime=6:00:00,nodes=1:ppn=16
# # output file base name
# #PBS -N test_dr
# # Put standard error and standard out in same file
# #PBS -j oe
# # Export all Environment variables
# #PBS -V
```

then run:

```
qsub -X -I <script>.csh
```
- Create and configure a new case (using gnu and only 1 task)
```
./create_newcase --pecount 1 --case <CASEDIR> --compset <COMPSET> --res <RESOLUTION> --compiler gnu --run-unsupported
```
- Turn on debug in the case `./xmlchange DEBUG=True`
- Build the case (`./case.build`)
- Run command `bash` to change to bash (if not already)
- Run the following commands:

```
np=1
nthreads=1

source .env_mach_specific.sh

RUNDIR=`./xmlquery RUNDIR -value`
EXEROOT=`./xmlquery EXEROOT -value`
LID=`date '+%y%m%d-%H%M%S'`

cd $RUNDIR
mkdir timing
mkdir timing/checkpoints
echo `pwd`
export OMP_NUM_THREADS=$nthreads
totalview ${EXEROOT}/cesm.exe
```

- `exit` to exit the totalview window and give up the node

### gdb

- `gdb` may be useful if you prefer a command-line interface for debugging.
- Similarly to TotalView, here are some basic set up tips for debugging:
    - Use `DEBUG=true`.
    - Build using `NTASKS=1` and `NTHRDS=1`, so you do not have to debug in parallel.
    - To more easily compare against CAM-SIMA, you can turn off chunking by specifying `-pcols <number>` in `CAM_CONFIG_OPTS` to be large enough to cover all columns.
    - Use an interactive node; the environment can be set up by `source .env_mach_specific.sh` from the case directory.
    - Run `gdb ../bld/cesm.exe` from the case run directory.
- If comparing against CAM-SIMA:
    - Generate the snapshots CAM-SIMA will run on by running CAM first; then, disable snapshot output in CAM when running CAM in the debugger. If CAM is writing the snapshots at the same time CAM-SIMA is attempting to read them, you will get cryptic `NaN` in file read errors in CAM-SIMA.
    - Remember that CAM snapshots start from the **second** timestep, so you have to skip the first set of whatever breakpoints you set in CAM to get to the first timestep CAM-SIMA runs.
- Tips for `gdb`:
    - Set breakpoints using `break file_name.F90:line`
        - Conditional breakpoints are useful if you want to break inside a loop; e.g., `break file_name.F90:123 if k == 26`.
        - Save breakpoints using `save breakpoints <file>` and load them using `source <file>` when restarting `gdb`.
        - Best locations for breakpoints for inspecting both CAM and CAM-SIMA are probably within the scheme itself; there you can inspect if the input variables coming into the scheme actually match each other (if not, then you may have to walk back to the previous scheme in the SDF to see where divergence started.)
        - While it may be easy to inspect physics state in CAM (in a `_tend` subroutine or directly in `tphysbc`/`tphysac`) finding the equivalent location in the CCPP cap may not be easy.
        - You can `info breakpoints` to see a list of breakpoints set, and show a count of how many # of times the breakpoints were triggered.
        - If your breakpoints are hitting more often than expected but the timestep is not advancing, maybe it was accidentally set in a loop, or chunking was not disabled (for CAM).
    - Inspect variables using the `print` command; if differences are isolated in some columns, you can consult the results of "Physics check data" from CAM-SIMA or `cprnc` output of history files to find where the values are different; then you can print the variable at these columns (Fortran syntax will work, i.e., `print zi(2:5,25:26)`)
    - A combination of breakpoints and watchpoints will help isolate where the CAM and CAM-SIMA runs diverge, then you can `backtrace` the exact call stack that lead to a specific variable diverging.
    - If you need to "go back in time", Totalview on Izumi will be helpful!
