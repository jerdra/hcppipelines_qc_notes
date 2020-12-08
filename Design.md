# HCP Pipelines Design

## Concepts

### Operators
Operators function on SVG/Image canvases, essentially with no information about the underlying data file

- _Overlay Operator_ - `over(bg, *fgs) -> imageNodeClass`
- _Transition Operator_ - `transition(*bgs) -> imageNodeClass`
- _Generic Operator_ - `func(*iterable<imageNodeClass>) -> imageNodeClass`

These could theoretically be implemented in a JavaScript image composer? With Django/Flask to serve images generate the static template?

### Image Classes
Image classes generate SVG views for `operator` functions to deal with, these contain over-rideable function implementations and may host either NIFTI base-class data or CIFTI/GIFTI surface-based data:

*Volume Based*
- _VolumeImage (ABSTRACT)_ - NIFTI base class providing methods for working with `.nii.gz` based files
- _EPIImage_ - extract the reference (1st vol) from a 4D image
- _TSNRImage_ - compute TSNR from a 4D image
- _MaskImage_ - generate a binary mask from a given NIFTI image
- _ContourImage_ - Generate a contour image from a binary mask
- _MeanImage_ - Generate a Mean image fom a 4D image
...Alternatively we could simplify this by converting our object based identities to a first-class function based identity, i.e use generic volume image that requires a function class to use

*Surface Based*
_SurfaceImage_ (ABSTRACT) - GIFTI/CIFTI based class providing methods for working with `CIFTI` and `GIFTI` formats given a base Mesh image
- LabelSurface - Handle dlabel-type images with full display capabilities
- ContourSurface - Handle Border ROI image representation
- MaskSurface - Handle mask display on image
...Again these representations of the data can be done by defining a class of functions with strict input/output representations

^ Maybe functions should inherit a specific abstract base class to enforce their structure so they can be used as plug and play objects?

### Design Strategies

Under each base class which will provide functions and routines for dealing with NIFTI/GIFTI/CIFTI data, the main differences we're going to be dealing with is how the data is pre-processed. Ultimately each class/pseudo-class will send back a file path/handle to a type of image file/data to canvas into a layout of sorts.

The simplest implementation strategy here is to define a base class for SurfaceProc and VolumeProc which will provide a single function called extract_data() with enforced image input and output types. This can be done by implementing typing, i.e:

1. Define a base class `VolumeImage`, `SurfaceImage`
2. VolumeImage and SurfaceImage will implement a function called extract_data() which will take in a `Callable` and self. It will perform extract_data using `Callable` and passing in self args.
3. The problem with using `Callable` is that we can't encode additional internal properties in the function as easily (i.e color for ROIs, Contours, Masks). Using an object-based definition can ease this since we're more flexible to implement additional properties rather than passing around a dictionary...:
	1. A way around this is to implement a **func_args dictionary to be passed into the callable?

