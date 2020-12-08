# hcppipelines Report Objects

## Volumetric T1

### Bias-field Correction
Bias field correction can be pulled from the following images:

*T1w*
`T1w/T1w_acpc_dc.nii.gz`
`T1w/T1w_acpc_dc_restore.nii.gz`

*T2w*
`T2w/T2w_acpc_dc_restore.nii.gz`
`T2w/T2w_acpc_dc_restore.nii.gz`


### Gradient Distortion Correction
May not be necessary for PRISMA or GE premier scanners due to lower fields. Check images anyway. Images have practically no difference, mostly because distortion maps weren't collected sooooo don't need this.


### BrainMask
HCP pipelines derives the Brain Mask from the post-freesurfer pipeline by using the wmparc and running a few basic morphological operations on it. The associated files to convert are:

*T1w*
`T1w/T1w_acpc.nii.gz`
`T1w/T1w_acpc_dc_brainmask.nii.gz`

*T2w*
`T2w/T2w_acpc_brain_mask.nii.gz`
`T2w/T2w_acpc.nii.gz`

### T1w/T2w Co-Registration

The registration of T1 --> T2 can be pulled from the following files:

`T1w/T1w_acpc_dc.nii.gz`
`T1w/T2w_acpc_dc.nii.gz`

### MNI Transformation

Computed in PreFreesurfer pipeline as:

`MNINonLinear/T1w.nii.gz`

Which can be directly compared to the FSL 1mm MNI template with an overlay after applying a brain mask.


## Volumetric EPI

### Gradient Distortion Correction
Not applicable

### Susceptibility Distortion Correction

Will give a view of *SDC* correction step transitioning using the first volumes

`<fMRI>/<ses>_<task>_<run>_mc.nii.gz`
`<fMRI>/GistortionCorrectionAndEPIToT1wReg_FLIRTBBRandFreesurferBBRbased/Scout_gdc_undistorted`

### T1 to EPI Co-Registration

BBRegister registration mechanism. Need surface overlays here w/T1w in Native

`<fMRI>/DistortionCorrectionAndEPITo1wReg_FLIRT.../Scout_gdc_undistorted2T1w.nii.gz`
`<fMRI>/DistortionCorrectionAndEPI.../T1w_acpc_dc_restore_brain`

### EPI Masking

Masking is performed using the same mask as the brainmask_fs, but they don't have an unmasked version U R G G G G. Maybe just use Co-registration here...

`<fMRI>/DistortionCorrection.../Scout_gdc_undistorted2T1w.nii.gz`
`T1w/brainmask_fs.2.nii.gz`

### Surface Workflows

*Structure*
### T2-Refined Surface Placement

Files to use:

`T1w/fsaverage_LR32k/<sub>.<hemi>.pial.32k_fs_LR.surf.gii`
`T1w/fsaverage_LR32k/<sub>.<hemi>.white.32k_fs_LR.surf.gii`
`T1w/fsaverage_LR32k/<sub>.<hemi>.midthickness.32k_fs_LR.surf.gii`
`T1w/T1w.nii.gz`
`T1w/brainmask_fs.nii.gz`

*Would it be useful to include a T1 --> T2 co-registration component here?*

### MNI Space Surface Placement

### Structural Images w/Sulcal Depth?

### Myelin Map

*Function*

### Default Mode Network connectivity

### Ribbon Image?
