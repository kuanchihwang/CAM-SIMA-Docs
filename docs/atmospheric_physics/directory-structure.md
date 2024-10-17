# atmospheric_physics directory structure

This page lists out the directory structure for the atmospheric_physics repo, and what the general purpose of each directory and subdirectory is.

## Top-level directories

These directories represent code and tools that is solely contained within the atmospheric_physics repository.  If there is a sub-directory that is also an external submodule or repository it will be marked with the "(external)" label.

### **doc/**

Contains files used to document the current state of the atmospheric_physics repository, such as a `ChangeLog` and the `NamesNotInDictionary.txt` file which lists all standard names that are currently  not in the official [CCPP Standard Names repo](https://github.com/ESCOMP/CCPPStandardNames).

### **schemes/**

Contains subdirectories for all of the CCPP-ized physics schemes, with the subdirectories containing the core physics scheme source code,
the associated CCPP metadata files, namelist XML files, and any relevant depdendency files.

Note that there is a special `to_be_ccppized` subdirectory which contains source code that is needed by certain CCPP physics schemes, but which is not
yet fully CCPP compliant (e.g. may have code that is host-model specific).

### **suites/**

Contains CCPP Suite Definition Files (SDFs) for official scientific and production configurations of the CAM-SIMA host model.

### **test/**

Contains code and tools used to run tests on the CCPP-ized physics schemes.

**Subdirectories**:

- cmake   - Contains cmake files needed to configure and build  unit tests.
- docker  - Contains dockerfiles needed to configure, butild, and run unit tests.
- include - Contains utility source code needed to build and run unit tests.
- musica  - Contains test code for the CCPP-ized [Multie-Scale Infrastructure for Chemistry Modeling (MUSICA)](https://github.com/NCAR/musica) components.
- test_suites - Contains test SDFs for use in CAM-SIMA snapshot regression testing.