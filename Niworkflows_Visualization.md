## How Niworkflows does Visualization

Niworkflows generates svgs based on the visualization utilities found in `niworkflows.viz.utils`. 

### General Plotting Mechanism

#### `plot_segs`

1. First set colorbar limits based on percentile range, otherwise use default settings which may be expressed through the `plot_params` kwargs
2. Next use a mask image in order to determine how to set the cut coordinates. My assumption is that we use equally spaced cuts within the bbox determined by the mask 
3. Then for each slicing dimension, generate segmentation plots using the `_plot_anat_with_contours` mechanism

#### `_plot_anat_with_contours`

1. Set colors
2. Plot the anatomical image using `nilearn.plotting.plot_anat`
3. store the object generated above, and use the `add_contours` method to draw segmentation lines
4. `extract_svg` from the plot and apply compression. Looks for SVG tag in nilearn image and extracts the data after changing some display-related components.
5. close the image plotting `object` and return the path to the SVG file


#### SVG composition

1. Add svg strings together
2. Get root node of each SVG then query width, height and store
3. Scale width/height as needed



