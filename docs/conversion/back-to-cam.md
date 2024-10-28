# 9 - Bring back into CAM

## Code modifications
Bring the CCPP version back into a new CAM clone or branch

- Make a new clone or branch of CAM6 (personal preference)
- Remove the non-CCPP version from your CAM source directory
- Make a new directory in the atmosperhic_physics submodule of CAM (make sure the directory name is the one you used in step one of the conversion to CCPP):
```
mkdir src/atmos_phys/<parameterization>
```
- Copy your new CCPP version files from CAM-SIMA into this new directory
- Update the routine names to match the CCPP-ized subroutine names where the subroutines are called
- Update the calling lists if they were changed when the CCPP-ized subroutines were made
- If the CCPP-ized module is in a new subdirectory, you will need to add it to `bld/configure`:
```
print $fh "$camsrcdir/src/atmos_phys/<parameterization>\n";

## Testing
```
- To quickly test that your code is working during development, you can create two new model cases with a compset that involves your physics scheme, and the following XML changes:

```
./xmlchange --force ROF_NCPL=\$ATM_NCPL
./xmlchange DOUT_S=FALSE,STOP_OPTION=nsteps,STOP_N=9,DEBUG=TRUE
```

!!! Note - test cases
    The first case will be CAM without your changes, and the second will be CAM with your new physics scheme code changes. Both simulations should produce a `<case_name>.cam.rh0.XXX` file, which you can compare by using the cprnc tool, which can be found (and used) on derecho below. Your new physics code is working correctly if the printed output from cprnc states that there are no differences in the files.

```
/glade/campaign/cesm/cesmdata/tools/cprnc/cprnc
cprnc <file1> <file2>
```
- Optional step: if want to have additional testing... Once you believe it is working, make a [snapshot](create-snapshots.md) to run CAM with this modified clone.
    - Use the cprnc tool to compare the resulting CAM6 “after” snapshot file with your original CAM6 “after” snapshot file (from before the port). If they differ and you changed the internal calculations during your port, you will need to determine if these changes are expected and correct.  
        - If they are different (and the changes are correct), then you will need to repeat the steps listed in the [Run CAM-SIMA](run-cam-sima.md) section with the newly generated snapshot files from the modified CAM clone.

## Committing your changes

- Open a PR in [ESCOMP/atmospheric_physics](https://github.com/ESCOMP/atmospheric_physics) for your parameterization and any interstitial code (including diagnostics) using the code in your sandbox
    - Make sure you include a ChangeLog and NamesNotInDictionary.txt file as specified [here](https://github.com/ESCOMP/atmospheric_physics/wiki/Development-workflow#changelog-and-namesnotindictionarytxt)
- Open a PR in [ESCOMP/CAM](https://github.com/ESCOMP/CAM) for your changes in CAM (if calling lists changed) or it may as simple as a PR to update the atmospheric_physics tag in `.gitmodules`. Don't forget to make a ChangeLog.
    - Be sure to include the `ccpp-conversion` label on your PR
- If any changes were made to CAM-SIMA (more than likely, at least an update to the atmospheric_physics submodule will probably be needed), open a PR in [ESCOMP/CAM-SIMA](https://github.com/ESCOMP/CAM-SIMA).
- Once all PRs are merged and tagged, check off the scheme as "Converted" in the [spreadsheet](https://docs.google.com/spreadsheets/d/1_1TTpnejam5jfrDqAORCCZtfkNhMRcu7cul37YTr_WM/edit?gid=0#gid=0)

[Details on opening PR’s](https://github.com/ESCOMP/CAM/wiki/CAM-Development-Workflow-in-GitHub#how-to-submit-code-changes-to-be-included-in-escompcam)