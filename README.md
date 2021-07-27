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

An optional step in the workflow for quantification of FISH puncta within invidiual neurites. We use the ImageJ SNT plugin (current version of the simple neurite tracer) to conduct supervised segmentation of individual neurites using the immunostaining channel. Individual paths within each image are segmented in 3D, filled using the *Fill Out* feature, and the filled models are converted to binary masks. A binary mask for each segmented dendrite should be saved as a TIFF file for downstream analysis.

See https://imagej.net/plugins/snt/ for details.

<br />

## 3) Quantification of FISH puncta within neurites

