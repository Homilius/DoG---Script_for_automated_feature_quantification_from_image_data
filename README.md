# Automated DoG - Script for Automated Feature Quantification from Image Data

## Introduction

"Automated DoG" is an automated FIJI script leveraging the "Difference of Gaussians" (DoG) method for unbiased identification and quantification of high-signal features within images (e.g. immunofluorescence stainings of stress-granules or P-bodies). Gaussian blur replaces pixel values with the weighted average of neighboring pixels. This process, applied with two sigma values, highlights regions of steep intensity-change gradients, aiding in the efficient identification of high-signal cellular features like stress granules through straigth forward thresholding.

## How It Works

1. **Background Subtraction:** Utilizes the Rolling ball and sliding paraboloid approach to subtract background locally.
2. **DoG Application:** Performs the difference of gaussians to identify regions of interest (ROIs).
3. **ROI Measurement:** Measures ROIs (e.g., Area, Mean, etc.) on the original image and exports the data to a .csv file for each image.

## Usage Instructions

To use the script, follow these steps:

1. **Prepare Your Environment:** Open FIJI and press `cmd+shift+n` to access the script executor.
2. **Set Directory Path:** Provide the path to your directory containing .czi (or .tif) files. Ensure there are no spaces in `dir_path` or file names (e.g., use "/Users/yourname/Desktop/U2OS_images" NOT "/Users/yourname/Desktop/U2OS images").
3. **Adjust Settings:** Modify threshold, particle size, and circularity settings as needed. Disable `run("Close All");` during adjustment to sanity check the output.

## Script Snippet

```javascript

// Provide path to image dir:
dir_path = "/Full_path_to_dir_containing_images";
dir_path = dir_path+"/" 

// List all files in the directory:
list = getFileList(dir_path);

// Process each .tif file:
for (i=0; i<list.length; i++) {
    if (endsWith(list[i], ".tif")) {
        // Get prefix from file name:
        prefix = replace(list[i], ".tif", "");
        suffix = ".tif";
        file_name = prefix + suffix;
        full_path = dir_path + file_name;

        // Open file:
        open(full_path);
        run("Grays");
        run("Enhance Contrast...", "saturated=0.20");
        run("Duplicate...", "title=Original");
        selectWindow("Original");
        run("Duplicate...", "title=clone");
        
        // Duplicate and adjust:
        selectWindow("clone");
        run("Subtract Background...", "rolling=10 sliding");
        run("Duplicate...", "title=1");
        selectWindow("clone");
        run("Duplicate...", "title=2");

        // Perform DoG (difference of gaussians):
        selectWindow(1); 
        run("Gaussian Blur...", "sigma=1");
        selectWindow(2); 
        run("Gaussian Blur...", "sigma=2");
        imageCalculator("Subtract create", "1", "2");

        // Define and measure particles: 
        selectWindow("Result of 1");
        run("Gaussian Blur...", "sigma=1");
        run("Enhance Contrast...", "saturated=0.20");
        setAutoThreshold("Otsu dark");
        //setThreshold(25, 100000, "raw");
        run("Set Measurements...", "area mean perimeter fit shape feret's integrated median display" + " redirect=" + "Original" + " decimal=2");
        run("Analyze Particles...", "size=0.3-20 circularity=0.3-1.0 show=Outlines display clear add");

        // Save data:
        saveAs("Results", dir_path + prefix + "_RESULTS.csv");
        // Close all windows:
        run("Close All");
    }
}
