# Requirements 
In order to validate the processing of HCPPipelines we need internal validation of a number of processes. A mix of Ciftify and fMRIPrep are needed

## Structural Component 
* T1w masks
* T1/T2w co-registration - surface placement
* Myelin Map surface plot
* Native Surface placement
* MNI Registration
* MNI Space surface placement
	* midthickness/pial
* 3D model of midthickness/pial fast check if brain looks like a raisin
* Segmentation/How are confounds extracted? (separate from report generation)

## Functional Component 
* EPI --> T1 co-registration
	* Use fMRIPrep-based visualization here
* HCP Fieldmap correction (TOPUP-only?)
	* Do we have access to these images?
* BOLD masks
* Vertices --> dyconn (erin arbitrarily selected vertices)
	* Dorsal Attention Network seed in Parietal
	* Default Mode Network
	* Salience Network
* Add standard cleaning
* Confound ROIs?
* Motion stats?
