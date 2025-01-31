.. _AddNewSchemes:
  
****************************************
Tips for Adding a New Scheme
****************************************

This chapter contains a brief description on how to add a new scheme to the *CCPP-Physics* pool.

* Identify the variables required for the new scheme and check if they are already available for use in the CCPP by checking the metadata tables in ``GFS_typedefs.F90`` or by perusing file ``ccpp-framework/doc/DevelopersGuide/CCPP_VARIABLES_XYZ.pdf`` generated by ``ccpp_prebuild.py``.

    * If the variables are already available, they can be invoked in the scheme’s metadata table and one can skip the rest of this subsection. If the variable required is not available, consider if it can be calculated from the existing variables in the CCPP. If so, an interstitial scheme (such as ``scheme_pre``; see more in :numref:`Chapter %s <CompliantPhysParams>`) can be created to calculate the variable. However, the variable must be defined but not initialized in the host model as the memory for this variable must be allocated on the host model side.  Instructions for how to add variables to the host model side is described in :numref:`Chapter %s <Host-side Coding>`.

    * It is important to note that not all data types are persistent in memory. The interstitial data type is erased every time step and does not persist from one set to another or from one group to another. The diagnostic data type is periodically erased because it is used to accumulate variables for given time intervals.

    * If information from the previous timestep is needed, it is important to identify if the host model readily provides this information. For example, in the Model for Prediction Across Scales (MPAS), variables containing the values of several quantities in the preceding timesteps are available. When that is not the case, as in the UFS Atmosphere, interstitial schemes are needed to compute these variables. As an example, the reader is referred to the GF convective scheme, which makes use of interstitials to obtain the previous timestep information.

* Examine scheme-specific and suite interstitials to see what needs to be replaced/changed; then check existing scheme interstitial and determine what needs to replicated. Identify if your new scheme requires additional interstitial code that must be run before or after the scheme and that cannot be part of the scheme itself, for example because of dependencies on other schemes and/or the order the scheme is run in the SDF.

* Follow the guidelines outlined in :numref:`Chapter %s <CompliantPhysParams>` to make your scheme CCPP-compliant. Make sure to use an uppercase suffix ``.F90`` to enable C preprocessing.

* Locate the CCPP *prebuild* configuration files for the target host model, for example:

    * ``NEMSfv3gfs/ccpp/config/ccpp_prebuild_config.py`` for the UFS Atmosphere
    * ``gmtb-scm/ccpp/config/ccpp_prebuild_config.py`` for the SCM

* Add the new scheme to the list of schemes in ``ccpp_prebuild_config.py`` using the same path as the existing schemes:

  .. code-block:: console

    SCHEME_FILES = [ ...
    ’../some_relative_path/existing_scheme.F90’,
    ’../some_relative_path/new_scheme.F90’,
    ...]

* If the new scheme uses optional arguments, add information on which ones to use further down in the configuration file. See existing entries and documentation in the configuration file for the possible options:

  .. code-block:: console

    OPTIONAL_ARGUMENTS = {
            ’SCHEME_NAME’ : {
            ’SCHEME_NAME_run’ : [
            # list of all optional arguments in use for this
            # model, by standard_name ],
            # instead of list [...], can also say ’all’ or ’none’
            },
        }

* Place new scheme in the same location as existing schemes in the CCPP directory structure, e.g., ``../some_relative_path/new_scheme.F90``.

* Edit the SDF and add the new scheme at the place it should be run. SDFs are located in

    * ``NEMSfv3gfs/ccpp/suites`` for the UFS Atmosphere
    * ``gmtb-scm/ccpp/suites`` for the SCM

* Before running, check for consistency between the namelist and the SDF. There is no default consistency check between the SDF and the namelist unless the developer adds one. Errors may result in segment faults in running something you did not intend to run if the arrays are not allocated.

* Test and debug the new scheme:

    * Typical problems include segment faults related to variables and array allocation.
    * Make sure SDF and namelist are compatible. Inconsistencies may result in segmentation faults because arrays are not allocated or in unintended scheme(s) being executed.
    * A scheme called GFS_debug (``GFS_debug.F90``) may be added to the SDF where needed to print state variables and interstitial variables. If needed, edit the scheme beforehand to add new variables that need to be printed.
    * Check *prebuild* script.
    * Compile code in DEBUG mode, run through debugger if necessary (gdb, Allinea DDT, totalview, ...).
    * Use memory check utilities such as valgrind.
    * Double-check the metadata table in your scheme to make sure that the standard names correspond to the correct local variables.

* Done. Note that no further modifications of the build system are required, since the *CCPP-Framework* will autogenerate the necessary makefiles that allow the host model to compile the scheme.


