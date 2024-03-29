[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.1474301.svg)](https://doi.org/10.5281/zenodo.1474301)

# MODFLOW-MAXDRN
Adjustments to the DRAIN package for MODFLOW-2005 and MODFLOW-NWT to allow for an upper limit on simulated drain fluxes.

## About
MODFLOW-MAXDRN is a version of MODFLOW-2005 [(Harbaugh, 2005; Harbaugh et al., 2017)](https://water.usgs.gov/ogw/modflow/mf2005.html) and MODFLOW-NWT [(Niswonger et al., 2011)](https://water.usgs.gov/ogw/modflow-nwt/) which allows maximum drain discharges to be specified by the user. These maximum discharges are used to place an upper limit on the simulated drain discharge. This is of particular interest when dimensioning dewatering systems with MODFLOW where drains are often used to find the discharge needed to obtain a targeted drawdown. Simulated discharge rates might be higher than what can be supplied by the physical well, particularly in early-stage drawdown. This results in simulated dewatering durations that are too optimistic in addition to cumulative extracted volumes that are larger than those supplied by the well. In MODFLOW-MAXDRN if, for a specific drain, the model calculates a discharge higher than the maximum discharge, this discharge will be set equal to the maximum and will become independent of the head difference between drain elevation and hydraulic head. In doing so, the drain switches from a head-dependent (Cauchy) boundary type to a constant-flux (Neumann) boundary. As such, an upper limit is imposed on the calculated drain discharge. 

The maximum-flux constraint was incorporated in the code by adjusting the source code of the DRAIN packages in MODFLOW-2005 v1.12 as well as in MODFLOW-NWT v1.1.4. Full backward compatibility was retained. For more information, see the the [MODFLOW-MAXDRN readme file](./doc/mf_maxdrn_readme.txt). The source code can be found in the [src folder](./src). Executables compiled with GNU Fortran are included as binaries in the release.

## To download
Under [releases](https://github.com/cneyens/MODFLOW-MAXDRN/releases), you will find a compressed file containing the repository which includes the source code (src), test and documentation (doc) folders. Compiled executables for MODFLOW-2005 (mf2005_maxdrn.exe) and MODFLOW-NWT (mf_nwt_maxdrn.exe) and test output files are also listed.

## Versions
Version 1.0.3 - October 2018

## License
This project is licensed under the MIT License. See also the [LICENSE](./LICENSE.md) file.

## References
[Harbaugh, A.W., 2005, MODFLOW-2005, the U.S. Geological Survey modular ground-water model -- the Ground-Water Flow Process: U.S. Geological Survey Techniques and Methods 6-A16.](https://pubs.usgs.gov/tm/2005/tm6A16/)

[Harbaugh, A.W., Langevin, C.D., Hughes, J.D., Niswonger, R.N., and Konikow, L. F., 2017, MODFLOW-2005 version 1.12.00, the U.S. Geological Survey modular groundwater model: U.S. Geological Survey Software Release, 03 February 2017, http://dx.doi.org/10.5066/F7RF5S7G](http://dx.doi.org/10.5066/F7RF5S7G)

[Niswonger, R.G., Panday, Sorab, and Ibaraki, Motomu, 2011, MODFLOW-NWT, A Newton formulation for MODFLOW-2005: U.S. Geological Survey Techniques and Methods 6–A37, 44 p.](https://pubs.usgs.gov/tm/tm6a37/)
