Longitudinal Disease Progression Modeling
================
Adam Lang

-   [Overview](#overview)
-   [Model fitting process](#model-fitting-process)
-   [FitDiseaseProgressionCurve](#fitdiseaseprogressioncurve)
-   [Example](#example)
    -   [Data](#data)
    -   [Model Fitting](#model-fitting)
-   [Comparison](#comparison)

Overview
========

The trajectory of a given AD biomarker may take decades, however we are
usually limited to much shorter follow up data in ADNI and other AD
focused studies. This can impose a challenge when estimating the full
course of AD disease progression. Using methodology described in Budgeon
et. al \[1\], we can estimate a full-term disease pathology curve from
short term follow up data.

<br>

Model fitting process
=====================

The model fitting process is outlined in detail in [Budgeon et.
al](https://pubmed.ncbi.nlm.nih.gov/28444781/)

<br>

FitDiseaseProgressionCurve
==========================

**FitDiseaseProgressionCurve** requires the following arguments:

***data*** (data.frame) a data frame containing:

   Outcome of Interest <code>(numeric)</code>  
   Column <code>ID</code> subject ids <code>(factor)</code>  
   Column <code>Time\_Since\_Baseline</code> <code>(numeric)</code> time
since baseline obs

***formula.fixed*** <code>(character)</code> fixed effect formula
<code>(should be “Outcome \~ Time\_Since\_Baseline”) </code>

***formula.random*** <code>(character)</code> random effect formula
<code>(default is “\~1+Time\_Since\_Baseline\|ID”)</code>

***n\_iter*** <code>(numeric)</code> number of iterations for
bootstrapping, <code>(default is 1)</code>

***n\_sample*** <code>(numeric)</code> proportion of data preserved for
bootstrapping <code>(default is 1)</code>

***lme.control*** <code>(list)</code> lmeControl list for mixed effect
modeling <code>(optional)</code>

***individual.lm*** <code>(logical)</code> use linear models for each
subject instead of one mixed effects model <code>(default is
FALSE)</code>

***seq.by*** <code>(numeric)</code> \# of times to integrate along
polynomial reciprocal domain <code>(default is 1000)</code>

Example
=======

Here we will generate simulated data to show model fitting process

Data
----

``` r
#Functions for simulating data in SimulationFunctions.R
set.seed(123)

#generate data based on sigmoid curve
example.data.list <- construct.simulated.dataset(0.4, 13, .01, .98, 20, seq(0,25,.1),
                                            length.subj = 3,  start.sim = 51, eps=0.5, id.start = 1)
```

<br>

Population sigmoidal curve that we will base our data generating process
on

[![init-curve.png](https://i.postimg.cc/tJ3r351M/init-curve.png)](https://postimg.cc/K3YrFtDP)
<br>

Longitudinal subject data generated based on our sigmoidal curve with
noise introduced

[![curve-lines.png](https://i.postimg.cc/j5G9Q2ML/curve-lines.png)](https://postimg.cc/1nrJymhQ)

<br>

Subjects realigned so that baseline is T=0

[![time-bline.png](https://i.postimg.cc/vHy2QpkF/time-bline.png)](https://postimg.cc/xqs35F9t)

<br>

Example of data frame for model

``` r
set.seed(123)
data <- example.data.list$data
knitr::kable(data[1:14,])
```

| Time\_Since\_Baseline | Simulated\_Response | ID  |
|----------------------:|--------------------:|:----|
|                   0.0 |           0.0099764 | 1   |
|                   0.5 |           0.0099843 | 1   |
|                   1.0 |           0.0099923 | 1   |
|                   1.5 |           0.0100002 | 1   |
|                   2.0 |           0.0100082 | 1   |
|                   2.5 |           0.0100161 | 1   |
|                   3.0 |           0.0100240 | 1   |
|                   0.0 |           0.0099753 | 2   |
|                   0.5 |           0.0099876 | 2   |
|                   1.0 |           0.0099999 | 2   |
|                   1.5 |           0.0100123 | 2   |
|                   2.0 |           0.0100246 | 2   |
|                   2.5 |           0.0100369 | 2   |
|                   3.0 |           0.0100492 | 2   |

Model Fitting
-------------

``` r
model.fit  <-  FitDiseaseProgressionCurve(data           = data, 
                                          formula.fixed  = "Simulated_Response ~ Time_Since_Baseline", 
                                          formula.random = "~1 + Time_Since_Baseline|ID", 
                                          n_iter         = 200, 
                                          n_sample       = .75, 
                                          seq.by         = 1000, 
                                          verbose        = FALSE, 
                                          individual.lm  = FALSE) 
```

Estimated population model

``` r
modfit<-model.fit$Model_Output$Model_Plot
ggsave("modelfit.tiff", units="in", width=5, height=4, dpi=500, compression = 'lzw')
```

[![modelfit.png](https://i.postimg.cc/yN2n25cG/modelfit.png)](https://postimg.cc/wtchmwW5)
<br>

Comparison
==========

[![curvewithlines.png](https://i.postimg.cc/c1RT5FHD/curvewithlines.png)](https://postimg.cc/ThPrKqnm)

Red is the population average, black is the estimated curve
