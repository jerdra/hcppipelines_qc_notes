# Working with Niworkflows reporting objects 

`Reportlets` object references HTML or SVG files and tacks on additional descriptions as well as section headings to the image. It takes the following arguments:
- `BIDSLayout` - reference to a BIDS layout object
- `config` - containing:
	- `title`
	- `description`
	- `bids` - which is a BIDS spec to be used by the associated `BIDSLayout` object
	- `config` - a formattable string object that is capable of taking on BIDS entities to fill in after the fact
Upon initialization the `Reportlets` object finds all BIDSFile under the description provided by `config` and attaches the `title` and `description` components onto them. Note that HTML snippets are injected straight into the final report.


`SubReport` is a container object which stores multiple `Reportlets` as well as associated `name`, `title`, and `description` parameters.

`Report` maintains a BIDSLayout object to index all `Reportlet` objects. The following parameter specifications are required:
- `reportlets_dir` - directory in which to store `Reportlets`
- `out_dir` - output directory in which to store the knitted HTML file
- `run_uuid` - store runtime information for nipype
- `config` - path object to a `YAML` file which contains information about how to put together the report.. This will actually instantiate reportlets and reports together for you using information in the `YAML` file.

However... no information is found on how to construct a proper `YAML` file... 
From the `Report.index` method and `fmriprep.yml` found in the niworkflows repo, we can probably determine the following information:
- `sections` which corresponds to `SubReport` objects contains a list of `YAML` keys of the following form. 
  
Starting with the simplest form (1 object across all sessions/tasks/runs)


	- name: <NAME>
	- ordering: how to sort svgs found within the reportlet descriptions
	- reportlets:
		- bids: <BIDS_CONFIG_DICT>
		 caption : <CAPTION>
		 subtitle: <SUBTITLE>
		 
		- bids: <BIDS_CONFIG_DICT>
		 caption : <CAPTION>
		 subtitle: <SUBTITLE>
		 description: <DESCRIPTION>
		 
		- bids: <BIDS_CONFIG_DICT>
		 caption : <CAPTION>
		 subtitle: <SUBTITLE>
		 static: (true|false)
		- ....
	- - name: <NAME>
	...
	
Where:
- `bids`: Points to BIDS configuration dictionary to be fed into `BIDSLayout`
	- Note that fields present in here are *regex enabled*
- `caption`: caption below embedded image, with substituteable keys
- `description`: presumably right after the reportlet header
- `static`: whether the SVG is dynamc or static
- `ordering`: in which order should `Entities` should be sorted for the particular section
