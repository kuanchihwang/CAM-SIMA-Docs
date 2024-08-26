# 8 - Run CAM-SIMA

It's time to try to run CAM-SIMA!

## Configure CAM-SIMA

- Either navigate to the case you created when you were validating your metadata or [create a new case](../usage/creating-a-case.md)
- Your `user_nl_cam` needs to contain the following (to bring in the cam_snapshot for input and to activate the validation tool):
```
ncdata='/FullPathName/your_before_snapshot_file.nc'
ncdata_check='/FullPathName/your_output_filename.nc'
debug_output = 0  !Turns off debug messages which interfere with the validation tool output
```
- If the number of vertical levels in your snapshot file is something other than 30, then you will also need to add the following to your user_nl_cam:
   
    ```
    pver = N
    ```

    where “N” is the number of vertical levels in your snapshot file (likely 32 levels for CAM6)

## Build and run

- Run `./case.build` and `./case.submit`
    - Consult the table below for common errors. Also see [debugging tips](../development/debugging.md) for help if you're getting errors

| Error text  |  Description   | Possible Fix |
|:------------|----------------|--------------|
|`ERROR: cam_get_file: FAILED to get UNSET_PATH` | No initial data file specified (or defaulted to) | Specify `ncdata = '<file>'` in your user_nl_cam file |
|`ERROR: read_field_3d: No variable foun din (/XXX, /)` | Variable not found when we're reading in the initial data or snapshot file | <ul><li>Double-check your standard name</li><li>Determine if CAM-SIMA needs to handle/initialize the variable. If it does, follow the procedure for the error below</li></ul> |
|`ERROR: Cannot read XXX from file, XXX has no horizontal dimension` | Variable is a scalar; we don't have an interface to read it in from a file (and it's not likely to be present on the file) | <ul><li>If "XXX" is static during the run, add `access="protected"` to the variable's XML entry in the registry</li><li>if the variable is not static, either:<ul><li>Initialize the variable in the init phase of your scheme (`intent=out`), or</li><li>initialize "XXX" in CAM-SIMA (NOT in the physics code) and add a call to `mark_as_initialized(<standard_name>)` so CAM-SIMA doesn't try to read in the variable</li></li></ul></ul> |

- Once the model completes without error, the results from the validation tool will appear in the `atm_log` file for each timestep (under the header ` ********** Physics Check Data Results **********`)
    - The message `No differences found!` should be logged for each timestep.
    - If there are differences found, that indicates there are issues with your ported code
    - See the section on [Tips for uncovering unexpected answer changes](#tips-for-uncovering-unexpected-answer-changes) for debugging strategies

!!! Note "Additional step for NAG compiler"
    If we have a working NAG compiler with CAM-SIMA (not working as of 8/26/2024), perform this extra step:

    - Make at least one run using the NAG compiler on izumi in debug mode and specifying “-nan” as an additional compilation option.  This option initializes all variables to NaN so that if something is uninitialized and is used, it will be trapped.  To enable this flag:
        - After executing the ./create_newcase and ./case.setup commands and BEFORE executing the ./case.build, edit the file in your case directory `cmake_macros/nag.cmake` (- If you want to change the file so that it runs every time you execute a `./create_newcase`, you can find the file at `ccs_config/machines/cmake_macros/nag.cmake`). 
        - Change the line Append line so that it contains ‘-nan’ and it should look like this:
    ```
    if (DEBUG)
       string(APPEND FFLAGS " -nan -C=all -g -time -f2003 -ieee=stop")
    endif()
    ```



Once you have successfully run CAM-SIMA and all answer changes have either accepted or fixed, proceed to [9 - Bring back into CAM](back-to-cam.md)

## Tips for uncovering unexpected answer changes

- Check all of your standard_names.  CAM-SIMA will run successfully, but if a standard_name is incorrect, you may be running with an incorrect variation of the variable that you intended
- If comparing printed values on the physics grid between CAM and CAM-SIMA, then it might be good to turn-off chunking in CAM, which can be done by setting “-pcols” in “CAM_CONFIG_OPTS” to be a large number (e.g. >500 for the ne3 grid).
    - You can also utilize the `tools/find_max_nonzero_index.F90` tool to (more) easily compare values between CAM and CAM-SIMA at a specific rank and index that is known to be non-zero
- Note that the way latitude in radians is calculated for the null dycore in CAM-SIMA is different (at a round-off level) to the way it is calculated in the SE dycore in CAM.  This means if you are validating a physics scheme that has latitude (in radians) as an input, then in CAM you’ll need to change the scheme interface to read in latitude in degrees, and then convert to radians by multiplying by pi/180 (which you’ll only want to do when creating the snapshot file, not when validating CAM itself).
- Sometimes CAM-SIMA will set a state variable directly, while CAM will only set a tendency variable and then use the “physics_update” routine to update the state using the tendency.  This can result in round-off level differences.  If this happens then simply try setting the state variable directly in CAM when creating the snapshot files for CAM-SIMA testing.
- If you run into a "tip" that should be added here, please confer with the other CAM SEs to get it added to the documentation!!