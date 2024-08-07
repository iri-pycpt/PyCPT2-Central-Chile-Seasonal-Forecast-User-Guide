# Introduction to PyCPT

<!-- 
```{image} img/intro.png
:alt: fishy
:class: bg-primary
:width: 800px
:align: left
```
 -->

Welcome to __PyCPT Version 2.8 for Seasonal Climate Forecasts!__ The goal of this tutorial is to learn how to
 configure and run PyCPT, Version 2.5+, in order to make, calibrate and verify
multi-model seasonal forecasts of precipitation based on the [NOAA North American Multi-Model Ensemble (NMME) ](https://www.cpc.ncep.noaa.gov/products/NMME/)
 and European [Copernicus Climate Change Service (C3S) ](https://climate.copernicus.eu) databases, obtained from [IRI Data Library](http://iridl.ldeo.columbia.edu). PyCPT helps to implement the [NextGen approach](nextgen-section-label) using the [Climate Predictability Tool (CPT)](cpt-section-label).

 
The PyCPT set-up and results are illustrated for a Central Chile precipitation forecast example for Jun-Aug 2024, initialized in May 2024. The example uses a five-model multimodel ensemble of the ECCC CanSIPSIC3, NCEP CFSv2, NASA GEOSS2S, COLA-RSMAS CCSM4, and GFDL SPEAR models.

```{admonition} PyCPT 2.8 is:
- a rewrite of PyCPT based on [Xarray](https://docs.xarray.dev/en/stable/) and a [new set of Python libraries](https://github.com/iri-pycpt) that provide a Python interface to the CPT fortran-based package, and facilitate access to GCM forecasts/hindcasts and observational gridded datasets via [IRI Data Library](http://iridl.ldeo.columbia.edu)
- designed to be run natively using Python on linux, Windows and Mac 
- easily installed by downloading a python conda environment file for your platform
- easily customizable  to individual forecaster needs using Jupyter Notebooks
- enables the use of observational predictand gridded datasets, such as [ENACTS,](https://iri.columbia.edu/resources/enacts/) from the user’s laptop in  CPTv10.tsv format. 
```

 __Who is this guide for?__
 
Basic familiarity with Python, seasonal climate forecasting, and IRI's [Climate Predictability Tool (CPT)](http://iri.columbia.edu/tools/) is assumed.

% __How to use this tutorial__

__Acknowledgments__

The guide to PyCPT2.8 for Seasonal Climate Forecasts and its associated teaching and learning materials represents a collaborative effort, made possible by the IRI’s dedicated team of climate, sectoral, and technical experts in collaboration with the Enhancing Adaptive Capacity of Andean Communities through Climate Services (ENANDES) project.

[Enhancing Adaptive Capacity of Andean Communities through Climate Services (ENANDES)](https://wmo.int/projects/enhancing-adaptive-capacity-of-andean-communities-through-climate-services-enandes) is a project that aims to increase the resilience and adaptive capacity to climate variability and change of highly vulnerable communities living in the Andes. It is led by the World Meteorological Organization (WMO) and supported by the Adaptation Fund.

%## Useful resources

---