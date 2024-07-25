# Configuring PyCPT 2.8

To configure and run PyCPT, you are going to be using the Jupyter Notebook web browser interface 

The PyCPT 2 workflow consists of the following steps, which we will illustrate using a Central Chile number of rainy days example:

- name a case directory for your results
- choose the desired MOS technique (CCA or PCR)
- select the GCM predictor and observational predictand datasets from IRI Data Library
- set the parameters for the analysis (predictor & predictand spatial domains, forecast initialization month and  target season, and hindcast period for regression model fitting
- for each GCM hindcast set, fit the regression model over the common hindcast period
- plot cross-validated deterministic skill scores for each GCM for CPT model validation
- plot the CCA and/or EOF modes and check for physical consistency 
- if needed, adjust the predictor domain, the maximum number of EOF/CCA modes allowed, and other parameters including any transformation the predictand data, dry mask etc. 
- select the subset of GCMs to include in the multi-model combination (MME)
- make MME hindcasts and examine their deterministic skill
- make the deterministic forecast in the desired format (e.g. forecast anomalies), and the probabilistic forecast in tercile and flexible formats.


## Central Chile rainy days example 

In this example, we are going to predict June-August number of rainy days over Central Chile (40°S-30°S), from the CPC forecast model initialized in the month prior of May. We will use __CCA__ to MOS-correct the predicted precipitation fields from five GCMs (`ECCC CanSIPSIC3`, `NCEP CFSv2`, `NASA GEOSS2S`, `COLA-RSMAS CCSM4`, and `GFDL SPEAR`) over the Central Chile region, using `CPC` data for model selection and validation. We will actually be leaving our `predictand_name` blank as '[ ]' and will be using a `local_predictand_file` later on in the process.

We will name our case directory `caseDir = "pycpt_CChilleJJA_startMay2024"`. Make sure to choose a descriptive name so that you can easily find your past experiments!

```python
case_dir = Path.home() / "Desktop/PyCPT" / "pycpt_CChilleJJA_startMay2024"
```

Note that we have specified the location to be within a directory call `PyCPT` on the Desktop, but this is up to you. The CPT output files will be saved to this case directory, into a subdirectory that identifies the predictand geographical domain.

###### 1. Choose the MOS technique and predictor and predictand datasets
```python
MOS = 'CCA' # must be one of 'CCA', 'PCR', or "None"
predictor_names = ["CanSIPSIC3.PRCP", "CFSv2.PRCP", "GEOSS2S.PRCP", "CCSM4.PRCP", "SPEAR.PRCP"]
predictand_name = 'PRCP'
```

The predictor and predictand names each indicate the dataset.variable. Type `dl.observations.keys()` to see all options for predictand, and `dl.hindcasts.keys()` to see all options for predictors. For example, UCSB.PRCP denotes the CHIRPS dataset (UCSB=University of California, Santa Barbara). _We would use this name if sourcing from the predictand dataset, but we will be using a `local_predictand_file` for this specific forecast._

```{important} 
1. Some sets of hindcasts have been superseded by new model versions. This [table](https://www.dropbox.com/s/q4xt92632r8cft5/NMME_C3S_models_table.pdf?dl=0) contains an inventory of which GCM forecasts are currently available in real time in IRI Data Library, along with the associated hindcast datasets.

2. Make sure your `first_year` & `final_year` of the hindcast training period are compatible with your selections for your predictors and predictands. 

3. At present, PyCPT 2.5 is limited to gridded predictand data. ***Station datasets are not yet supported.*** 
```

```{important}
For this forecast, we will be supplying an off-line gridded-data predictand file. If we were supplying from the dataset, we would use an ID such as `UCSB.PRCP`, but in this case, we will give our `local_predictand_file` as name. Give this file an easily-recognizeable name such as 'wet_days_freq.tsv' __(see instructions below).__
```


```python
local_predictand_file = 'rainy_days_central_chile.tsv'
```

```python
# To download predictand observations from the Data Library, set
#
#   local_predictand_file = None
#
# To read predictand data from a local file instead,
# set local_predictand_file to the full pathname of the file. e.g.
#
#   local_predictand_file = 'rainy_days_central_chile.tsv'
#
# The file should be formatted according to the following guidelines:
# https://cpthelp.iri.columbia.edu/CPT_use_input_gridded.html7
```

###### 2. Specify the forecast dates and geographical domains

__a) Forecast initialization date and hindcast period__   

The following parameters specify the __initialization date__ of the model forecasts `fdate`, together with the `first_year` & `final_year`  of the __hindcast period.__ 

```python
   # 'fdate':
   #   the initialization date of the model forecasts / hindcasts
   #   this field is defined by a python datetime.datetime object
   #   for example: dt.datetime(2023, 5, 1) # YYYY, MM, DD as integers
   #   The year field is only used for forecasts, otherwise ignored
   #   The day field is only used in subseasonal forecasts, otherwise ignored
   #   The month field is an integer representing a month - ie, May=5
  'fdate': dt.datetime(2024, 5, 1),  
    
   # 'first_year':
   #   the first year of hindcasts you want. **NOT ALL MODELS HAVE ALL YEARS**
   #   double check that your model has hindcast data for all years in [first_year, final_year]
   #   This field is defined by a python integer representing a year, ie: 1993
  'first_year': 1993, 
    
   # 'final_year':
   #   the final year of hindcasts you want. **NOT ALL MODELS HAVE ALL YEARS**
   #   double check that your model has hindcast data for all years in [first_year, final_year]
   #   This field is defined by a python integer representing a year, ie: 2016
  'final_year': 2016, 
```

Here we have chosen 1993-2016 as the training period, for which the `CanSIPSIC3`, `CFSv2`, `GEOSS2S`, `CCSM4`, and `SPEAR`, models have hindcasts available (the `GEOSS2S` hindcasts stop in 2016). The [CHIRPS dataset](http://iridl.ldeo.columbia.edu/SOURCES/.UCSB/.CHIRPS/.v2p0/.daily-improved/.global/.0p25/) (UCSB.PRCP) starts in 1981 and runs to the present.

`fdate` is the initialization date of the model forecast/hindcasts. Here PyCPT will make a forecast for Jun-Aug 2024, initialized on May 1, 2024, calibrated using hindcasts initialized in May of each year 1993-2016.

```{admonition} What are hindcasts?
Hindcasts are forecasts for past years, made using the current version of the GCM forecast model.
```

__b) Forecast lead time and target season__

The following parameters specify the __lead time__ of the forecasts/hindcasts. `lead_low` is the number of months from the first of the initialization month to the center of the __first__ month included in the target period. `lead_high` isa similarly the number of months from the first of the initialization month to the center of the __last__ month included in the target period, given by `target`.

For the Jun-Aug target with an `fdate` of May 1, we need to set `lead_low=1.5` and `lead_low=3.5`. 

```python     
   # 'lead_low': 
   #   the number of months from the first of the initialization month to the center of 
   #   the first month included in the target period. Always an integer + 0.5. 
   #   this field is defined by a python floating point number 
   #   for example  a lead-1 forecast would use lead_low=1.5, if you want init=may, target=Jun-..
  'lead_low': 1.5,
    
   # 'lead_high': 
   #   the number of months from the first of the initialization month to the center of 
   #   the last month included in the target period. Always an integer + 0.5. 
   #   this field is defined by a python floating point number 
   #   for example  a forecast initialized in may, whose target period ended in Aug, 
   #   would use lead_high=3.5
  'lead_high': 3.5, 
    
   # 'target': 
   #   Mmm-Mmm indicating the months included in the target period of the forecast. 
   #   this field is defined by a python string, with two three-letter month name abbreviations 
   #   whose first letters are capitalized, and all other letters are lowercase
   #   and who are separated by a dash character. 
   #   for example, if you wanted a JJA target period, you would use 'Jun-Aug'
  'target': 'Jun-Aug',
```

__c) Geographical domains of the predictor and predictand__

Finally, the following parameters give the geographical bounding boxes of the climate model data (`predictor_extent`) and observation data (`predictand_extent`).  

```python    
   # 'predictor_extent':
   #   The geographic bounding box of the climate model data you want to download
   #   This field is defined by a python dictionary with the keys "north", "south",
   #   "east", and "west", each of which maps to a python integer representing the 
   #   edge of a bounding box. i.e., "north" will be the northernmost boundary,
   #   "south" the southernmost boundary. Example: {"north": 90, "south": 90, "east": 0, "west": 180}
  'predictor_extent': {
    'east': -65,
    'west': -80, 
    'north': -30, 
    'south': -40
  }, 
   # 'predictand_extent':
   #   The geographic bounding box of the observation data you want to download
   #   This field is defined by a python dictionary with the keys "north", "south",
   #   "east", and "west", each of which maps to a python integer representing the 
   #   edge of a bounding box. i.e., "north" will be the northernmost boundary,
   #   "south" the southernmost boundary. Example: {"north": 90, "south": 90, "east": 0, "west": 180}
  'predictand_extent': {
    'east': -65,
    'west': -80, 
    'north': -30, 
    'south': -40
  },
```

```{admonition} How large should the predictor and predictands geographic domains be?
The predictand domain should cover your region of interest and can be quite small (e.g., Togo). The predictor domain should generally be larger to represent GCM seasonal rainfall variations over a larger region (e.g., West Africa). Select predictor domains and variables that reflect well-known physical climate mechanisms (e.g., the West African monsoon).
```

###### 3. Set the CPT analysis parameters

The following parameters control the behavior of CPT. 

```python
 cpt_args = { 
    'transform_predictand': None,  # transformation to apply to the predictand dataset - None, 'Empirical', 'Gamma'
    'tailoring': 'Anomaly',  # tailoring - None, 'Anomaly'
    'cca_modes': (1,3), # minimum and maximum of allowed CCA modes 
    'x_eof_modes': (1,8), # minimum and maximum of allowed X Principal Componenets 
    'y_eof_modes': (1,6), # minimum and maximum of allowed Y Principal Components 
    'validation': 'crossvalidation', # the type of validation to use - 'crossvalidation'
    'drymask': False, #whether or not to use a drymask of -999
    'scree': True, # whether or not to save % explained variance for eof modes
    'crossvalidation_window': 5,  # number of samples to leave out in each cross-validation step 
    'synchronous_predictors': True, # whether or not we are using 'synchronous predictors'
}
```

```{Note} 
Check the transform/tailoring options!! 
```


```{important}
The following step is only for accessing datasets not currently a part of __dl.observations.keys()__ such as `CPC`. If using a dataset such as `USCB.PRCP`, set `_local_predictand_file_='None'` and omit __Step 4__ below.
```

###### 4. Download the local data file

Once we've set our CPT analysis parameters, we must direct our predictand data sourcing to the local predictand file, in this case the `CPC` global precipitation dataset. Jupyter will show a box as the following:

```python
URL_fw = "https://iridl.ldeo.columbia.edu/SOURCES/.NOAA/.NCEP/.CPC/.UNIFIED_PRCP/.GAUGE_BASED/.GLOBAL/.v1p0/.extREALTIME/.rain/T/({first_year})/({final_year})/RANGE/X/({predictand_extent['west']})/({predictand_extent['east']})/RANGE/Y/({predictand_extent['south']})/({predictand_extent['north']})/RANGE/%28{target}%29/10/0.90/seasonalfreqGT/-999/setmissing_value/T//pointwidth/0/def/(months%20since%201960-01-01)/streamgridunitconvert/T/2.5/shiftGRID/T/T//pointwidth/3/def/replaceGRID/{'%5BX/Y%5D%5BT%5Dcptv10.tsv' if filetype == 'cptv10.tsv' else 'data.nc'}"
dl.download(URL_fw, 'rainy_days_central_chile.tsv', **download_args, verbose=True, use_dlauth=False)
```
Once the source has been recognized, the notebook will develop a URL which indicates the dataset, our climate variable (.rain), the timeframe of analysis, our domain, our months of focus, and a threshold of minimum precipitation. In this case, we've defined our minimum threshold to be 1 **(in units of millimeters)**.


```{admonition} What is __force_download?__
The predictor and predictand datasets can be large and lengthy to download from IRI Data Library. Set `force_download = False` to avoid re-downloading files you have already downloaded. Note: if you have changed anything in `download_args` since the data were last downloaded, you must set `force_download = True``, otherwise you will use the old data instead of the new.
```







     