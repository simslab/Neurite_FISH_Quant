# Neurite_FISH_Quant
Python code for quantification of FISH puncta in neurites
<br />

This repository contains Jupyter Notebooks with Python 3 code for quantification of FISH puncta within neuronal dendrites or axons. However, prior identification of FISH puncta in the images (and optional neurite segmentation) is required. We use ImageJ plugins for this purpose, as specified below. The workflow assumes Z-stack, multi-channel fluorescence images, with one channel used for identification of neurites and other channels for FISH.

<br />

The overall analytical workflow can be summarized in three major steps:
<br />

**1) Identification of FISH puncta (ImageJ / TrackMate plugin)**<br /><br />
**2) Segmentation of neurites (optional) (ImageJ / SNT plugin)**<br /><br />
**3) Quantification of FISH puncta within neurites**
   - 3D analysis within an entire field (thresholding to identify neurite pixels, does not require segmentation)
   - 2D/3D analysis of individual segmented neurites (requires binary mask for each segmented neurite)

<br />Each step is addressed in detail below.

### Software Requirements

1. ImageJ: https://imagej.net/software/fiji/
   - TrackMate plugin: https://imagej.net/plugins/trackmate/
   - SNT plugin: https://imagej.net/plugins/snt/

2. Python 3 (and Jupyter Notebook)
   - *Pandas, Numpy, Matplotlib, SciPy* modules
   - *aicsimageio* module: https://github.com/AllenCellModeling/aicsimageio

<br />

## 1) Identification of FISH puncta (ImageJ / TrackMate plugin)
    
The first step in the analysis is identification of FISH puncta within the image files, which we conduct in ImageJ using TrackMate (Tivenez et al., 2017). The downstream analysis is general and requires (X,Y,Z) coordinates of each punctum, and could be used with any spot detection algorithm.

When initializing TrackMate, note the X, Y, and Z pixel size in microns: these are required to convert the micron coordinates to pixel coordinates during downstream analysis. Within TrackMate, we use the Laplacian of Gaussian detector. For RNA Scope images, the estimated spot diameter should typically be between 0.5 - 1.0 microns. The optimal detection settings often vary depending on the tissue, probes, and image settings. Further filtering on spot quality, total intensity, etc. is often necessary to suppress detection of non-specific background. These settings should be optimized alongside images of negative control samples that were acquired under identical processing and imaging conditions.

After detection and filtering of spots, the tracking features within TrackMate can be skipped. The final output for each image is generated using the 'Export Spot Statistics' function. Save each set of Spot Statistics as a CSV file: the key features for downstream analysis are the (X,Y,Z) coordinates of each spot.

See https://imagej.net/plugins/trackmate/ for details.

References:

Tinevez, J.-Y., Perry, N., Schindelin, J., Hoopes, G. M., Reynolds, G. D., Laplantine, E., … Eliceiri, K. W. (2017). TrackMate: An open and extensible platform for single-particle tracking. Methods, 115, 80–90. doi:10.1016/j.ymeth.2016.09.016

<br />

## 2) Segmentation of neurites (optional) (ImageJ / TrackMate plugin)

An optional step in the workflow is quantification of FISH puncta within invidiual neurites. For this, we use the ImageJ SNT plugin (current version of the simple neurite tracer) to conduct supervised segmentation of individual neurites using the immunostaining channel. Individual paths within each image are segmented in 3D, filled using the *Fill Out* feature, and the filled models are converted to binary masks. A binary mask for each segmented neurite should be saved as a TIFF file for downstream analysis. The linear path length for each segmented neurite should also be saved for downstream analysis (puncta per micron of neurite).

See https://imagej.net/plugins/snt/ for details.

<br />

## 3) Quantification of FISH puncta within neurites

As noted above, the quantification of puncta within neurites can be performed on entire fields of neurites (option a) or on segmented neurites (option b). Details for each workflow and the associated Jupyter Notebooks are found below:


#### a) 3D analysis of FISH puncta per volume of neurite within an entire field

Jupyter Notebook: *TrackMateQuant_perField.ipynb*

**Input Requirements:** <br />
1. TIFF image file containing the immunofluorescence channel used for neurite identification. Must be the exact same image used for FISH spot identification with TrackMate
2. CSV file containing the Spot Statistics output from TrackMate for each RNA to be analyzed.
3. (X,Y,Z) pixel size from the TrackMate input menu

<br />



**Key Parameters:** 
1. *Z_um, Y_um, X_um*: pixel sizes in microns, can be obtained from the TrackMate input menu or from the metadata within your image files
2. *Threshold*: Intensity threshold for the neurite immunofluorescence channel, set to determine which pixels are neurites.
3. *Pct_Threshold*: Percentage of pixels required to be overlapping between FISH puncta and neurites (as decimal between 0-1) to determine whether a FISH puncta is co-localized within a neurite.

<br />

**Workflow:**

The notebook is designed to iterate through all files within a given directory. The directory should contain a TIFF image file and an accompanying TrackMate CSV files for up to two RNAs. For example, a set for a single image containing one neurite channel and two RNA channels would be: (Image1.tiff, Image1_TrackMate_RNA1.csv, Image2_TrackMate_RNA2.csv)

A broad overview of the analytical workflow is summarized here:

1. Pixels of neurites are identified using a threshold on the immunofluorescence channel. The *Threshold* parameter is manually entered, and is typically in the range of 2 standard deviations above the background. The desired threshold to identify neurites should be manually examined for satisfactory inclusion of neurite pixels within the images (i.e., in ImageJ).
2. Some basic features about the neurite pixels are calculated, such as the fractional volume of the image occupied by neurite pixels, etc.
3. The coordinates of the centroid of each punctum are converted into pixel coordinates. Each punctum is treated as a 23-pixel spheroid-like object: essentially a 3x3x3 cube surrounding the centroid pixel and excluding the four corner pixels. FISH puncta with centroid coordinates at the edge pixels of the image cannot be analyzed in this way and are removed.
4. Spot statistics are calculated, including: Total # of Puncta (SpotCount), # of Puncta within neurites (SpotCount_Pos), % of Puncta within neurites (PosSpots_Percent), # of Puncta per image volume (SpotCount_Vol), and Puncta per neurite volume (SpotCount_Neurite_Vol)
5. Merging of spot statistics for each RNA within each image and data export.

<br />

#### b) 3D analysis of puncta per volume or path length for individual segmented neurites

Jupyter Notebook: *TrackMateQuant_perNeurite.ipynb*

**Input Requirements:** <br />
1. CSV file containing the Spot Statistics output from TrackMate for each RNA to be analyzed.
2. (X,Y,Z) pixel size from the TrackMate input menu
3. A binary mask (TIFF image file, 0 = background / 255 = neurite) from ImageJ / SNT plugin for each individual neurite. Masks must be derived from the exact same image used for FISH spot identification with TrackMate.
4. CSV file containing path information (Path Length, number of branches, anything else from SNT plugin...) for each binary mask.


<br />


**Key Parameters:** 
1. *Z_um, Y_um, X_um*: pixel sizes in microns, can be obtained from the TrackMate input menu or from the metadata within your image files
2. *Pct_Threshold*: Percentage of pixels required to be overlapping between FISH puncta and neurites (as decimal between 0-1) to determine whether a FISH puncta is co-localized within a neurite.

<br />

**Workflow:**

The notebook is designed to iterate through all sets of binary masks for each image. In contrast to the *TrackMateQuant_perField.ipynb* workflow, this means a single set of TrackMate CSV files for up to two RNAs is used to analyze multiple segmented neurites (binary masks) derived from each original image. Files should be stored in subdirectories within a single parent directory as follows:

--> ParentDirectory <br />
------> *TiffMasks*: BinaryMasks for segmented neurites from each original image, i.e. (Image1_Neurite1.tif, Image1_Neurite2.tif, etc...) <br />
------> *TrackMate*: TrackMate RNA output files from each original image, i.e. (Image1_Spots_RNA1.csv, Image1_Spots_RNA2.csv, etc...)

A broad overview of the analytical workflow is summarized here:

1. Pixels of the binary mask for the segmented neurites are identified using a simple threshold (neurite pixels from the *SNT Fill* function must be set to 255 in the mask).
2. The data processing steps within each iteration are essentially the same as in the *TrackMateQuant_perField.ipynb* workflow, with quantification of the neurite pixel features, pixel coordinate overlap analysis for each punctum, summary and export of Spot statistics. The difference in this workflow is that each of these measurements is made per neurite (per binary mask file). 
3. After storing the Spot statistics for each neurite, these data can be combined with the path length and other information from the ImageJ SNT plugin (an example is provided with a file 'NeuritePathInfo.csv'). Thus, in addition to 3D quantification per volume of neurite (as in the *perField* workflow), it is also to quantify the # of puncta per micron length of neurite with this workflow.



