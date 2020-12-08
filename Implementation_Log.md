# Implementation Log

## Constructing a YAML configuration spec

First we're gonna try out a rather simple specification of the report generation module. Just using dseg in order to understand how parts are going to be put together

The first error is that the root node is expected to already exist within the `report_dir` variable that is fed into the `Report` object. In addition, the `reportlet_dir` arg must contain all the SVGs organized into a BIDS-like format. Internally, by default, niworkflows uses the built-in `figures` configuration.

The following snippet successfully initializes a `Report` object:

{{{python

r # Report(reportlets_dir#Path(report_dir),
			out_dir#Path(out_dir),
			run_uuid#"anyuniquestring",
			config#Path(yaml_file))
}}}

In `niworkflows.core.reports` there is a utility called `generate_reports` which can take a subject list and generate reports for each one of them. Although all it does is wrap another function `run_reports` which initializes a `Report` instance for each subject and generates reports.

It works! The next step is to write functions to generate surface SVGs from HCP pipelines.

## Structuring the script ##

The volumetric component of the script will be mainly handled by niworkflows visualization tools in terms of displaying segmentation and overlays...

The responsibilities of the script will be to:
1. Generate the required files from HCPPipelines outputs
2. Output SVGs into a reports folder using a BIDS structure
3. Generate HTML reports

### Generating Required Files

*Basic Tasks Required*
1. Masking
2. Overlay of mask contours
3. Overlay of surface contours
4. Overlay of image files for transitioning

*Solutions to task*
`niworkflows.viz.utils` contains a number of function we can take advantage of:

- `extract_svg` will handle nilearn --> svg transformation
- `plot_segs` will handle masking + segmentation outlines (FAST)
- `plot_registration` will handle overlays w/ribbon if included
	- What does `contour` look like here? its ribbon.mgz from freesurfer
	- How does `ribbon.mgz` load? They use `contour.get_fdata()`
	- cuts needs to be defined, must be a dictionary w/each view (z,x,y)
	- `add_contours` using `pial` and `white` which are defined using `nilearn.image.new_img_like`
- `compose_view` sets CSS flickering animation on SVG

### Components Required

*Solved*
1. A way to pull files from the HCPPipelines outputs. Can use a YAML configuration file to spec how to map files to a more human-readable format. This can store an index for us at the single subject level. Don't really need the flexibility of a whole BIDSLayout sort of deal


2. A way to compose transformations onto files in a step-by-step fashion. Could also use a YAML configuration file here which could allow for easy additions and swapping. This is perhaps the more configurable way of settings things up for volume data. Idk how surface based data should be handled here - clearly polymorphisms are required. But for a quick and dirty solution you could just use shitty if-statements to deal with logic until we refactor this using objects and a more complete system later on


Stick this in a single YAML file
The first step is to define file paths with a YAML specification.


### Implementing indexing functionality

- [X] Write the YAML spec to translate templates --> named objects with properties (surface etc). The YAML requires the following major components:
	- space (base folder name)
	- mapping to name
	- whether we expect multiple files or not?
	- whether the file is surface-based or volume based
	- template path to look in within space directory

- [X] Pair files appropriately respecting the conditions we expect for HCPPipelines

### Defining cuts for visualization

For cuts we need a mask which will enable us to choose sensible end-points for visualization. For each space image we need to define a mask in the same space which can be used.
If we define space to point to masks uniquely then this would solve our problem. The question of course is - does a mask exist for each of our desired spaces?

- [X] Confirm whether mask exists for each of the required spaces:
	- [X] EPI: `Scout_gdc_mask.nii.gz`
	- [X] T1w: `brainmask_fs.nii.gz`: to confirm, we probably don't need this anymore
	- [X] T1w_acpc: `T1w_acpc_brain_mask.nii.gz`
	- [X] T2w_acpc: `T2w_acpc_brain_mask.nii.gz`
	- [X] MNINonLinear: `brainmask_fs.nii.gz`

Now that we've confirmed our masking requirements. The next step is to fetch masks. The assumption here is that we're using objects in the same space so either FG or BG images can have an associated mask.

- [X] Fetch mask

The final component is to re-direct the generated SVG to the final output directory which will be used for reportlets. This requires some information, specifically the following:

- [X] Datatype, if func then derive the session
- [X] Description, found in composition item
- [X] Suffix, found in composition item
- [X] Subject, not really included anywhere... except arg

The issue is that each rendered fg/bg contains the full path. Alternatively you could return hcp_objs for each glob item which will conveniently store the datatype with it

- [X] Modify functions to use the NamedTuple more consistently

Next we need to construct the full path spec from out_dir. For anatomical we exclude session, whereas for func we include session as per fMRIPrep's reportlet structure. It looks like `BIDSLayout.build_path` with only `entities` as the arg is useless:
- [X] Try using `path_patterns`


### Milestones ###

- [X] First test SVG works!
- [X] Figure out how to inject text
- [X] Figure out how to get the appropriate cuts - need mask, define mask for each space
- [ ] Do a full volume run
- [ ] Implement surface methods BUT LIKE HOW. will probs need a better way...


### Required Figures ###

- [ ] T1 bias field correction
- [ ] T2 bias field correction
- [ ] T1/T2 co-registration
- [ ] Brainmask overlay
- [ ] MNI Transformation
- [ ] SDC
- [ ] EPI to T1 co-registration
- [ ] EPI masking (ROI maps may require multiple inputs here... as part of the ROI)

### Refactoring to extensibility ###


#### Object Version ####

The issue w/the current method is that it only supports fg/bg views and transitions. It doesn't allow for the inclusion of contours etc because that would require additional dictionary keys which gets messy real quick.

We're at the point where we should construct object identities/decorators to deal with the multiple steps required. For example:

1. Organizing SVGViews to compose individual elements in the appropriate manner
2. A way of fetching files from the HCPPipelines output directory (layout)
3. Dealing with volumes and surfaces differentially

The part we've solved is probably the specification of input files using the YAML configuration file and partially the composition mechanics.

Here's the imagined workflow:
1. Each composition is an SVGView of type "Segmentation, Registration". They have a set of inputs that can be used for:
	1. fg image - forego this concept entirely?
	2. bg image
	3. entities
	4. description
	5. output specification

Some key details we need to take care of:
1. Whether or not you need a bg image relies on whether flickering is required
2. In addition contour specification should be able to handle surfaces or segmentation masks appropriately.

The base class `SVGRenderer` requires the following:
1. fg image (what is the makeup of this?)
2. description
3. output specification

The subclass for `VolumeRenderer` requires the following:
1. Inherit from base class
2. segmentations to use (what is the makeup of this?)

For volume rendering we're actually probably mostly there. We just need a factory that will give us a `VolumeRenderer` as needed... In addition depending on how segmentations are implemented we should be able to flexibly handle how contours are generated

The subclass `SurfaceRenderer` requires the following:
1. Inherit from base class
2. Modulation layer (sulcal typically - could be overloadable)

Here, rendering will be done using a different tool than niworkflows (custom nilearn/nibabel scripting)

The subclass `FunctionalSurfaceRenderer` requires the following:
1. Inherit from base class
2. A seed mask for calculating seed-based FC views
3. Modulation layer (sulcal typically - could be overloadable)


Each of the above "Renderer" classes will handle the generation of SVGs. Any composition that we'd like to do should be explictly separate from this functionality.

The next step is how to interface with the user. In this case composing objects with the YAML is actually probably fine. We just need to be careful about how we map this information into class instances

How will the fg/bg pieces work as well as named-based composition?


My ideal system under the hood would be the following:

Each object is fully captured by its "SVGRenderer" object which has the sole functionality of generating its own SVGs and exporting them. A factory class is used to generate these on the basis of the YAML configuration spec that is passed to it.

A sort of pairing functionality which can reference pairs of SVGs to compose them for export (compose_view) should work in most cases. You can probably organize the above classes into a dictionary to reference them w/unique keys. The Factory class can probably just output a dict with references to all the `SVGRenderer` instances.

Composition can be done w/a managing hook by an exporter of sorts. So in all `SVGRenderers` should not be able to handle their own export. But by an `Exporter` family of functions or class instead.

Structure:
1. Create a NiSVGFactory concrete factory class
2. Register each subclass of NiSVGPlotter to NiSVGFactory
Each subclass will hold essential information extracted from the YAML configuration file and handle its own SVG generation mechanism (surface vs volume based vs functional)

## Dealing with Overlay Patterns

When working in Volume space, Nilearn accomplishes overlay through using the Display object which is essentially a `nibabel.OrthoSlicer` class of objects. When working with this we have to take the output object and add modifications to it via passing it around/chaining commands.

There are some basic functionalities that we would like to perform:

*Composition*
Can be done by compositor classes/functions
- Composit overlays, transitions, patches onto an image or surface

*Preprocessing*
Sometimes the data needs to be preprocessed on the fly before images are generated. For example a seed-based connecivity may be run on the raw data before generating an image. This can be problematic in some sense because we need a consistent input/output specification

_Implementation_

*Define some basic operations to be performed on the data*
1. Reduction operations to combine multiple data pieces into one (reduce)
	- Masking
	- Sum
	- Product
	- Division
	- Multiply
_Note:_ These are dependent on whether the object is surface or volume. These functions can just call _extract_data on the container objects then perform the associated operation using a simple numpy operation. In this case these interfaces don't need to know about the underlying functionality of these containers. Let's set this up first!

*Data Models*
With the NIFTI images it is required that we have a shape and affine matrix. However with CIFTI/GIFTI data it is often required that there are associated surf.gii files that should be used. Meaning that additional information needs to be stored. The question is how we can construct these objects

_Question to solve_
- How to construct objects with differing dependencies (builder)... use kwargs w/ignored param?

