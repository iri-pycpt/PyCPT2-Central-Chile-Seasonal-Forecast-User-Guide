# The multimodel NextGen ensemble 

First we need to select a subset of GCMs to include in the MME. Factors to consider are:
- deterministic skill maps for the individual models: All skill scores should be generally positive across the map
- their CCA/EOF modes: The CCA modes of each model should be physically interpretable and the CC coefficient should generally be about 0.4 or more.
- the forecast maps: Any differences in polarity between models should be confirmed by looking at the global maps from the NMME and C3S websites.  

Although the CanSIPSIC3 model skill is low in our Central Chile example, we will still include it in our two-model MME for demonstration purposes.

```python
ensemble = ['CanSIPSIC3.PRCP, 'CFSv2.PRCP']
```

The MME is constructed as the __unweighted average__ of forecasts or hindcasts of the individual models. This is done for the deterministic values as well as for the tercile-category probabilities, in order to create both multi-model deterministic and probabilistic hindcasts and forecasts. The prediction error variances are also averaged across models in order to compute the spread of the MME flexible forecasts (see below).

###### 1. Plot MME Hindcast Skill 

The multi-model deterministic and probabilistic hindcasts are evaluated using a suite of skill metrics using CPT's `GCM Validation` and `Probabilistic Forecast Verification` procedures.


```{tip} 
To see all the skill scores available, insert a new cell in the Notebook (before the plotting cell) and type `nextgen_skill`. You will see an xarray dataset that contains all the skill scores available for the MME hindcasts.
```

We will select four skill metrics, two deterministic (Spearman and 2AFC) and two probabilistic (GROC and RPSS).

```python
skill_metrics = ['spearman','2afc', 'generalized_roc', 'rank_probability_skill_score']
``` 
Comparing the two deterministic measures with the individual model maps, the MME __Spearman__ correlation is generally positive and it slightly enhanced compared to the better individual model (`CFSv2`), and is much better than that of `CanSIPSIC3`. However, the __2AFC__ eludes a simple interpretation compared to that of the individual models; the MME 2AFC exceeds that of climatology in the central region of Central Chile.

The MME RPSS is positive everywhere--exceeding climatological skill--confirming that the hindcasts are probabilistically reliable.  The MME's GROC is rather low throughout most of the region except for the far northern extent of Central Chile, exceedubg that of climatology almost everywhere.

(need more guidance on this)

```{image} img/mme_skill_scores.png
:alt: fishy
:class: bg-primary
:width: 800px
:align: left
```
#
###### 2. Plot the MME forecast maps

The MME forecast is shown below in both tercile probabilities and anomaly (deterministic) format. The below-normal category dominates the forecasts almost everywhere, and the seasonal rainy days anomaly shows a weaker negative signal throughout most of the region.
    
```{image} img/mme_forecast_wet_days.png
:alt: fishy
:class: bg-primary
:width: 600px
:align: left
```

```{admonition} Interpretation
 The `CanSIPSIC3` and `CFSv2` calibrated rainy days forecasts over Central Chile have similar structures: anomalously slightly negative throughout most of the region with a near-normal number of rainy days expected in north-central Chile. The MME of the two models together is an even blend of both the 'CanSIPSIC3' and `CFSv2` models. However, using more than two models would account for more variation. This highlights the limitation of our 2 model MME. __The use of at least 3-4 models is recommended.__  
```


##### 2. Plot MME Flexible forecast 

The final section of the notebook plots a map of the probability of exceedance (PoE) of any selected threshold, and the exceedance and PDF curves for a selected gridpoint:

```python
# if 'isPercentile is True, the threshold is a percentile (e.g., 0.5)
# else in the unit of the predictand (e.g., mm, degC, ...)
threshold = 0.5
isPercentile = True

# choose a gridpoint within the predictand domain to plot the forecast and climatological
# probability of exceedance and PDF curves. If set to None, the centroid of the predictand domain will be used.
point_latitude = -35
point_longitude = -72
```

Here we select the median and the point (72W, 35S) located over Chile, which is marked by a red cross on the map below.

Consistent with the tercile-format forecast, the PoE the median is mostly negative. 

At our Chile gridpoint, the forecast PoE curve (red, left panel) is shifted downward and the PDF (red, right panel) is shifted to the left of the climatological curve (blue). The selected threshold value (here the median) at this gridpoint is shown by the black dashed vertical linne - here about 25 days. 

```{image} img/prob_of_exc_plots_wet_days.png
:alt: fishy
:class: bg-primary
:width: 800px
:align: left
```

```{note} 
The empirical distributions of the climatology are shown in pale blue on the PoE and PDF plots. These represent the 35 observed annual values of Jun-Sep precipitation total at the Chile gridpoint (1993--2016 period). 
(need these both in pale blue)
```

The final cells record information about the python environment that will be useful if we need to reproduce this configuration in the future.

```python
!conda list
!conda list --explicit
```
