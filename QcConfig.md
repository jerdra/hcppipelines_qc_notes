# VizSource Class

This class assembles the inputs by reading off a YAML configuration file which specifies how to join objects.

The following information must be captured when specifying the YAML syntax.

1. The input files to be used to generate the report image
2. The output name in a BIDS formatted output
3. The method to use to display the image

The trickiest part is to have an appropriate output name mapping. Maybe this needs to be explicitly encoded into the metadata i.e:

1. Map from regex --> BIDS key-words
2. Be able to collapse across sessions if needed (i.e preprocessed T1s) - if missing spec then assume its not used
3. Use of a surface/volumetric visualization should be done through specifying which method to use

## YAML Config

The YAML file should follow the structure given below:


### Method 1 - Implicit BIDS structuring
```yaml

- Item1:
	method: (Registration|Segmentation|...)
	args:
		bg_nii:
			file: <FILE EXPRESSION>
			bids_map:
				ses:<REGEX TO EXTRACT SES>
				sub:<REGEX TO EXTRACT SUB>
				desc:<REGEX TO EXTRACT DESC OR NAME>
				task:<REGEX TO EXTRACT TASK>
				anat: <WHATEVER>
			out_path: <sub>/<ses>/anat/...
			match_on: [ses, sub,...]...
```

**Problem:** `out_path` shouldn't be defined for each argument...

### Method 2

```yaml

- Item1:
	method: (Registration|Segmentation|...)
	args:
		bg_nii: <FILE GLOB OR PATH>
		fg_nii: <FILE GLOB OR PATH>
		segs: [<FILE GLOB OR PATH>, <FILE GLOB OR PATH>, ...]
		...
	bids_map:
		ses: {<INTERFACE_ARG>, <REGEX OR CONSTANT>}
		sub: {<INTERFACE_ARG>, <REGEX OR CONSTANT>}
		desc: {<INTERFACE_ARG>, <REGEX OR CONSTANT>}
		task: {<INTERFACE_ARG>, <REGEX OR CONSTANT>}
		modality: {<INTERFACE_ARG>, <REGEX OR CONSTANT>}
	out_path: <sub>/<ses>/anat/...
	match_on: [<BIDS_ENTITIES>]
```

Here, any listed item such as segs will match each listed regex w/1 item per regex. Then `bg_nii`, `fg_nii`, and `segs` will be paired up using `match_on` which is derived from `bids_map`

**Problems**:
- You may need more than 1 item per glob pattern (i.e a numbered output that can be captured in a single glob expression). In that case, this would fail since you need 1 item per glob. 
- One saving grace is that we may be able to use `match_on` to group items to use instead of relying on enumerating each expected item

### Method 3

```yaml

- Item1:
	method: (Registration|Segmentation|...)
	args:
		- field: bg_nii
		  values: <FILE GLOB OR PATH>
		- field: fg_nii
		  values: <FILE GLOB OR PATH>
		- field: segs
		  match_on: <REGEX>
		  values: [<FILE GLOB OR PATH>, <FILE GLOB OR PATH>, ...]
		  order_by: <ORDER_KEY_REGEX>
	bids_map:
		ses: {<INTERFACE_ARG>, <REGEX OR CONSTANT>}
		sub: {<INTERFACE_ARG>, <REGEX OR CONSTANT>}
		desc: {<INTERFACE_ARG>, <REGEX OR CONSTANT>}
		task: {<INTERFACE_ARG>, <REGEX OR CONSTANT>}
		modality: {<INTERFACE_ARG>, <REGEX OR CONSTANT>}
	out_path: <sub>/<ses>/anat/...
	match_on: [<BIDS_ENTITIES>]
```

**Problems**:
- We could use the approach of relying on `match_on` for collating the inputs we need. 
- Or can define it on the `args` level

Specs:
- `method`: Refers to the type of QC image that will be generated
- `bids_map`: Use information from `<INTERFACE_ARG>` to extract information to map to a BIDS entity
- `out_path`: Use `bids_map` generated entities to create a file path (could enforce a BIDS-derivatives to simplify user configuration by default)
- `args`: Contains a list of dictionaries with a key value of the field to use, and a file expression or path as its value. We can specify additional properties if using `Method 3` such as:
	- `field` - field name to map to
	- `values` - values to input
	- `order_by` - how to order outputs with multiple items are expected as values, use a regex key
