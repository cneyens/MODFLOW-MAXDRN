Cas Neyens - Vrije Universiteit Brussel
June 2018
cas.neyens@vub.be

			 MODFLOW-MAXDRN
			 	v1.0.3
--------------------------------------------------------------------------------------------------

MODFLOW-MAXDRN (mf_maxdrn) is a version of MODFLOW-2005 (Harbaugh, 2005) and MODFLOW-NWT (Niswonger et al., 2011) which allows maximum drain discharges to be specified. If, for a specific drain, the model calculates a discharge higher than the maximum discharge, this discharge will be set equal to the maximum and will become independent of the head difference between drain elevation and hydraulic head. As such, an upper limit is imposed on the calculated drain discharge.

IMPLEMENTATION
Hereafter follows a brief overview of the conceptual implementation. For input instructions, see the corresponding 'mf_maxdrn_input.pdf' file.

During each iteration step, for a given drain L at cell IJK (where the head in the cell is greater than the drain elevation), the volumetric discharge from this drain is calculated by:
			QQ = C*(EL-HNEW)
where QQ is the volumetric discharge [L**3/T], C is the user-specified conductance value [L**2/T], EL is the user-specified drain elevation [L] and HNEW is the model-calculated hydraulic head [L] at cell IJK for the current iteration. Note that QQ is always negative, i.e. a drain can never produce inflow in the domain, conform the MODFLOW-2005 implementation. If the absolute value of QQ is greater than the user-specified maximum discharge for drain L, then:
			QQ = -QMAX
where QMAX is the user-specified maximum volumetric discharge for drain L [L**3/T] and a positive number. Because QMAX is specified as a positive number and because a drain flux is always negative, the negative sign is added. QQ is than subtracted from the right-hand side of the groundwater-flow equation in matrix form by:
			RHS(J,I,K) = RHS(J,I,K) - QQ
				or:
			RHS(J,I,K) = RHS(J,I,K) - (-QMAX)
The left-hand side of the matrix formulation remains as is. As such, the the drain flux becomes head-independent (i.e. a constant flux) for the current iteration step. If QMAX is specified as a negative number, the drain has an unlimited discharge and is therefore treated as a classic drain in MODFLOW. The calculations above are then skipped.
When QQ is close to QMAX, the iteration may proceed slowly because MODFLOW switches from a head-independent to a head-dependent boundary condition between iterations which introduces non-linearity in the equation.

PROGRAMMER DOCUMENTATION
MODFLOW-MAXDRN is implemented in MODFLOW-2005 v1.12 and MODFLOW-NWT v1.1.4. No changes have been made to the source code except for the file containing the DRAIN package (gwf2drn7.f & gwf2drn7_NWT.f, respectively), the Link-MT3DMS package LMT8 (lmt8.f & lmt8_NWT.f, respectively) and the Drain Observation Package DROB (obs2drn7.f). These changes are commented, starting with the flag '!CNe'. MODFLOW-MAXDRN is fully backward compatible and all functionalities of the base versions are retained. The exception to this is the GMG solver in MODFLOW-2005, which has not been compiled for this version. MODFLOW-MAXDRN has been compiled using the GNU FORTRAN Compiler in the Code::Blocks IDE, following the guidelines on the online guide to MODFLOW (https://water.usgs.gov/ogw/modflow/MODFLOW-2005-Guide/index.html?suggestions_on_compiling.htm).

The changes in the source code were made in such a way that 1) adjustments to existing code are minimal, 2) additions to existing code are minimal and 3) all changes are made only in subroutines of the DRAIN package and packages related to writing output of drains (LMT and OBS).

GWF2DRN7 PACKAGE
		GWFDRNMODULE
			- Added a logical variable IDRNQMAX which is a flag indicating if MODFLOW-DRAIN is active.

		GWF2DRN7AR
			- Call IDRNQMAX from GWFDRNMODULE
			- Allocate IDRNQMAX and set to FALSE
			- Added a statement which sets IDRNQMAX to TRUE if the 'QMAX' keyword is specified in the inputfile as well as writing a statement to the listing file. 'QMAX' is added to the DRNAUX array for writing labels to the listing file.
			- The first dimension of the DRAI array is incremented by one to account for QMAX values
			- Read QMAX values for drain parameter K in the loop and write this value to the listing file
			- Because DRAI has different dimensions when MODFLOW-DRAIN is active, the ULSTRD subroutine arguments (NDRNVL & IAL) are adjusted when reading the parameter drain records. 
			- The NDRNVL-th record of the first dimension in the DRAI array is filled with QMAX for parameter K.
			- Repeat the above two steps for time-varying parameters

		GWF2DRN7RP
			- Call IDRNQMAX from GWFDRNMODULE
			- Incremented ULSTRD subroutine arguments (NDRNVL & NAUX) by one when reading QMAX from non-parameter drain records. Copied the NDRNVL-th record of the DRAI array to the NDRNVL+1-th record. This is done because the NDRNVL-th record is filled with drain flows when writing a list-formatted cell-by-cell flow file during budget calculations.
			- Incremented UPARLSTSUB subroutine arguments (NDRNVL, NREAD & NAUX) when substituting drain parameters when MODFLOW-DRAIN is active. Copied the NDRNVL-th record of the DRAI array to the NDRNVL+1-th record.

		GWF2DRN7FM
			- Call IDRNQMAX & NDRNVL from GWFDRNMODULE
			- Added QQ & QMAX double precision variables
			- When MODFLOW-DRAIN is active (IDRNQMAX=TRUE), QMAX is set equal to the NDRNVL+1-th record of the DRAI array. If QMAX is negative, skip the following. QQ is calculated as the drain discharge. If the absolute value of QQ is greater than QMAX, replace QQ with -QMAX. Subtract QQ from RHS and go to the next drain in the loop.

		GWF2DRN7BD
			- Call IDRNQMAX from GWFDRNMODULE
			- Added QMAX double precision variable
			- When MODFLOW-DRAIN is active (IDRNQMAX=TRUE), QMAX is set equal to the NDRNVL+1-th record of the DRAI array. If QMAX is negative, skip the following. If the absolute value of QQ is greater than QMAX, replace QQ with -QMAX.

		GWF2DRN7DA
			- Deallocate IDRNQMAX

LMT8 PACKAGE
		LMT8DRN7
			- Call IDRNQMAX & NDRNVL from GWFDRNMODULE
			- Added QMAX double precision variable
			- When MODFLOW-DRAIN is active (IDRNQMAX=TRUE), QMAX is set equal to the NDRNVL+1-th record of the DRAI array. If QMAX is negative, skip the following. If the absolute value of QQ is greater than QMAX, replace QQ with -QMAX.

OBS2DRN7 PACKAGE
		OBS2DRN7SE
			- Call IDRNQMAX & NDRNVL from GWFDRNMODULE
			- Added QMAX double precision variable
			- When MODFLOW-DRAIN is active (IDRNQMAX=TRUE), QMAX is set equal to the NDRNVL+1-th record of the DRAI array. If QMAX is negative, skip the following. If the absolute value of HH is greater than QMAX, replace HH with -QMAX.


CHANGELOG:
	v1.0.0 (June 2018): Initial stable version.
	
	v1.0.1 (July 2018): Drains with a negative QMAX have no maximum limit on the discharge.

	v1.0.2 (August 2018): Fixed issue where QMAX was not read properly in transient simulations for KSTP > 1 (NDRNVL-th record of DRAI array was filled with drain fluxes overwriting QMAX values). Adjusted LMT and OBS subroutines to incorporate QMAX flows.

	v1.0.3 (October 2018): Changes to be more consistent with general MODFLOW coding procedure. IDRNQMAX was changed from a logical to an integer flag. IDRNQMAX was added to the GWFDRNTYPE and in the SGWF2DRN7PNT and SGWF2DRN7PSV subroutines. These changes have no affect whatsoever on the behaviour of the code with respect to v1.0.2. 

