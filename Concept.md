
# Conceptual Structure of Program 

Want to develop a QC dashboarding software which is capable of displaying summaries, filtering, flagging, commenting. The idea is that this QC dashboard is specifically tailored towards preparation for analysis. We want to flow from outputs of a pipeline directly into a formal QC stage for analysis.

The basic underlying structure of this is plotting NIFTI/CIFTI data in a composable manner. Unfortunately this is not well supported by Nilearn/Nibabel/Niworkflows since they take in all inputs at once. This isn't necessarily something that we want. We want to be able to continue adding to plotting entities dynamically. 


*The plotting mechanism: Viewers*
This generates a plot given an image using the associated plotting function:

- `plot_surf` for surfaces w/overlays
- `plot_anat`/`plot_epi` for volume

_What about modifications?_
The question is whether we have equivalent modifications on each object.

1. *ROIs/Contours*
Contours and ROIs have analogs as borders and patches in both volume and surface space

2. *Seed connectivity*
Seed connectivity has analogs in volume and surface space

_Basic Implementation_
Viewers will work on subclassing for now, can think about other utilities later on...

With surfaces `plot_surf` expects overlays in the call itself, but does not add to the object that is created.
With volumes `plot_epi` you can fk around with the object that is created and returned.

