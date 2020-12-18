# HCPQC Class Hierarchy

## Grabbing images from HCP pipelines

With HCPPipelines we've already defined most of our input image sources, we just need to source them and put it together. Furthermore we can generalize this to *any* pipeline through specify how we grab our datasources through a JSON configuration file which fully specifies how the images are gathered and combined. 

Using a JSON spec + class is useful because it means that for any new pipeline we want to generate QC images for, we can just write a JSON configuration file and the rest of the work will be handled by the `QcConfig` class as well as our `nipype` interface objects.

[QcConfig Class](./QcConfig.md)

## Reports to create

All reports that we need to create are outlined in:

- [Requirements](./Requirements.md)
- [Source Files](./Source_Files.md)

## Volumetric

Since most of the images that we need to build essentially replicate an identity operation, we can use the `IRegRPT` and an equivalent `ISegRPT` class to generate *all our visualizations*.


- [ ] Update input specification of `IRegRPT` class to allow for more flexible arguments to pass to the `RegistrationRC` superclass
- [ ] Create `ISegRPT` class in a similar manner to `IRegRPT`


## Surface

Niworkflows surface report capabilities are lacking. We'll need to create custom classes much like `niworkflows.interfaces.report_base` but will contain a `_post_run_hook(self, runtime)` routine that will generate the surface visualizations. Nilearn will work for this, but using other packages such as `meshplot` and `pythreejs` could be even more powerful.

The following `nipype.interfaces.mixins.reporting.ReportCapableInterface` subclasses are needed:

- [ ] `ISurfRPT` - display mesh surfaces with optional overlay, will be used for
	- [ ] Myelin Map
	- [ ] Default Mode connectivity map
	- [ ] Curvature map
	- [ ] Areal distortion map
- [ ] `ISurfSegRPT` - for ROI border display if needed (i.e show various parcellations with networks such as default mode (i.e Yeo + DMN seed))?
