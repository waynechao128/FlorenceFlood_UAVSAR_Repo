This is a github repository used to hold the scripts for the manuscript named: 
**"Flood extent mapping during Hurricane Florence with repeat-pass L-band UAVSAR images"** submitted to ***Water Resources Research*** 
</br>This research was financially supported by the **Dynamics of Extreme Events, People, and Places (DEEPP)** project (https://deepp.cpc.unc.edu/).

Chao Wang<sup>1</sup>, Tamlin M. Pavelsky<sup>1</sup>, Fangfang Yao<sup>2</sup>, Xiao Yang<sup>1</sup>, Shuai Zhang<sup>3</sup>, Bruce Chapman<sup>4</sup>, Conghe Song<sup>5</sup>, Antonia Sebastian<sup>1</sup>, Brian Frizzelle<sup>6</sup>, Elizabeth Frankenberg<sup>7</sup>, Nicholas Clinton<sup>8</sup>

</br><sup>1</sup>Department of Earth, Marine and Environmental Sciences, University of North Carolina, Chapel Hill, NC, USA
</br><sup>2</sup>CIRES University of Colorado Boulder, Boulder, CO, USA
</br><sup>3</sup>College of Marine Science, University of South Florida, St. Petersburg, FL, USA 
</br><sup>4</sup>Jet Propulsion Laboratory, California Institute of Technology, Pasadena, CA, USA
</br><sup>5</sup>Department of Geography, University of North Carolina, Chapel Hill, NC, USA
</br><sup>6</sup>Carolina Population Center, University of North Carolina, Chapel Hill, NC, USA
</br><sup>7</sup>Department of Sociology and Carolina Population Center, University of North Carolina, Chapel Hill, NC, USA
</br><sup>8</sup>Google, Inc., Mountain View, CA, USA


![UAVSAR_Workflow](./Figures/UAVSAR_processing_flowchar.jpg)


### Introduction
This is a flood detection algorithm framework (Figure above). The workflow includes three major components.
</br>The processing steps in the pink box were carried out using the European Space Agency (ESA) PolSARpro v6.2 software package (Pottier et al, 2009) through custom python batch scripts. The detailed steps include extraction of T3 coherency matrix elements, “Refined Lee Filter” speckle filtering, polarization orientation angle correction, and polarimetric decomposition.
</br>The steps in the light yellow and light blue boxes were implemented in the Google Earth Engine (GEE) platform using the python (v3.7.3) API (v0.1.200), which include radiometric terrain correction, radiometric normalization, and supervised classification modules.

The proposed framework for flood inundation mapping from UAVSAR imagery has two components:
1) **Local Processing**:
</br>A) UAVSAR fully polarimetric extraction 
</br>B) Polarietric de-speckling, de-orientation 
</br>C) Polarimetric decomposition

2) **Cloud-based Processing**:
</br>A) Polarimetric terrain correction and normalization
</br>B) Supervised Classification

# Data
The UAVSAR data can be downloaded from this website:
https://uavsar.jpl.nasa.gov/

# How to start and Requirements
1) Download **PolSARpro v6.0 (Biomass Edition)** Software:
</br>We have tried to download and install the Linux version of PolSARpro (https://github.com/EO-College/polsarpro) in UNC longleaf HPC, however, this HPC does not support this software.
</br>So we turn to download Windows 64 bits Version follow the instruction as this website: https://ietr-lab.univ-rennes1.fr/polsarpro-bio/
</br>After installed the PolSARpro v6.0, we tested it with manully running a test by extracting and decompositing a PolSAR data.

2) **Batch** mode with Python:
</br>Find the right version for your setup of **Anaconda3** platform for running python scripts. In this study, we used **Spyder** IDE. Since we are performing analysis on four different flightlines and each flightline has 4-5 observations, it will be fast to have a batch mode. Thus, we developed a python script use **'subprocess'** function to Exclude Exe Program, which is the similar idea as how PolSARpro v6.0 windows version tcl GUI works.

```For instance, extract UAVSAR data as T3 matrix: Select Polarimetric Matrix Generation
#uavsar_convert_MLC
#-hf input annotation file
#-mem Allocated memory for blocksize determination(in Mb)
try:
    ExcludeProgram=softDir+'data_import\\'+'uavsar_convert_MLC.exe'
    subprocess.call(ExcludeProgram + " -hf \"" + Parameter_hf + \
                                     "\" -if1 \"" + Parameter_if1 + \
                                     "\" -if2 \"" + Parameter_if2 + \
                                     "\" -if3 \"" + Parameter_if3 + \
                                     "\" -if4 \"" + Parameter_if4 + \
                                     "\" -if5 \"" + Parameter_if5 + \
                                     "\" -if6 \"" + Parameter_if6 + \
                                     "\" -od \"" + Parameter_od_T3 + \
                                     "\" -odf T3 -inr "+str(grd_rows)+" -inc "+str(grd_cols)+\
                                     " -ofr 0 -ofc 0 -fnr "+str(grd_rows)+" -fnc "+str(grd_cols)+\
                                     "  -nlr 1 -nlc 1 -ssr 1 -ssc 1 -mem 4000 -errf \""+ Parameter_errf+"\"")
except subprocess.CalledProcessError as e:
    raise RuntimeError("uavsar_convert_MLC command '{}' "+\
                       "return with error (code {}): {}".format(e.cmd, e.returncode, e.output))
```

## Other Python Packages Installation
Check the file of 'requirements.txt'. For install GEE python API, please check 'GEE Python Instructions'.

# GEE Python Instructions
</br>In addition, for this pipeline to work you will need to have a GEE configured python installation ready to go.
</br>Explaining exactly how to do this is beyond the scope of this package but Google provides detailed installation instructions [here](https://developers.google.com/earth-engine/python_install).

## Usage
The main start function is the script named **'MainFunction.py'**.

**Step1**:
</br>**Local processing**:
</br>1) 'ExtractPolarimetricSAR_1.py'
</br>2) 'ConvertHH_HV_VV2Geotiff_1.py'
</br>3) 'PolarimetricDecomposition_1.py'

**Step2**:
</br>**Upload input metrics raster files** (here, we used Cloud Optimized GeoTIFF, please see detailed at https://www.cogeo.org/), because we processed lots of raster data so that we prefer to use Google Cloud Storage(GCS, https://developers.google.com/earth-engine/Earth_Engine_asset_from_cloud_geotiff?hl=en). 
'UploadUAVSARGeoTiff2CloudStorage.ipynb'
</br>Of course, you can upload to GEE assets instead of GCS. 


**Step3**:
</br>**Conduct normalization and classification**:
</br>Before conducting normalization procedure, it requires the incidence angle raster file instead of provided local incidence angle file (which has taken into account the local topography), because our study sites are relatively flat and also we have already conducted terrain correction.
</br>The script named **'DownloadIncidenceAngle_4.py'** was used to generate incidence angle used later.
</br>The script named **'GetNormalizationCorrectionParameters_6.py'** was then used for extracted the parameters for correcting side-look gradient. specifically, we adopted a simple log-scaled linear regression.

## Citation
Wang, C., Pavelsky, T. M., Yao, F., Yang, X., Zhang, S., Chapman, B., et al. (2022). Flood extent mapping during Hurricane Florence with repeat-pass L-band UAVSAR images. Water Resources Research, 58, e2021WR030606. https://doi.org/10.1029/2021WR030606

If you find our work useful for your research, please consider citing the following paper.
```
@article{https://doi.org/10.1029/2021WR030606,
author = {Wang, Chao and Pavelsky, Tamlin M. and Yao, Fangfang and Yang, Xiao and Zhang, Shuai and Chapman, Bruce and Song, Conghe and Sebastian, Antonia and Frizzelle, Brian and Frankenberg, Elizabeth and Clinton, Nicholas},
title = {Flood Extent Mapping During Hurricane Florence With Repeat-Pass L-Band UAVSAR Images},
journal = {Water Resources Research},
volume = {58},
number = {3},
pages = {e2021WR030606},
keywords = {L-band UAVSAR, polarimetric SAR, flooding, inundation extent, flooded vegetation, incidence angle},
doi = {https://doi.org/10.1029/2021WR030606},
url = {https://agupubs.onlinelibrary.wiley.com/doi/abs/10.1029/2021WR030606},
eprint = {https://agupubs.onlinelibrary.wiley.com/doi/pdf/10.1029/2021WR030606},
note = {e2021WR030606 2021WR030606},
year = {2022}
}
```

## References



## Resources
The material is made available under the **MIT License**: Copyright 2021, Chao Wang, Tamlin M. Pavelsky, of Global Hydrology Lab - University of North Carolina, Chapel Hill.
All rights reserved.
