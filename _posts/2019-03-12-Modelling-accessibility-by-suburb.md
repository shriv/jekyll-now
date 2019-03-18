---
mathjax: true
toc: true
toc_sticky: true
toc_label: "Table of Contents"
sidebar:
  nav: "acc_series"
---

# Introduction
*__WORK IN PROGRESS__*

## Technical details
To do this analysis, we need to overcome some technical aspects:
- Test a Bayesian model at the suburban level
- Compare suburban averages in Wellington
- Compare suburban heterogeneity in Wellington


# Datasets

| Dataset | Format | Link |
| :-----: | :----: | :--: |
| WCC playground locations  | .zip| [Wellington City Council](https://data-wcc.opendata.arcgis.com/datasets/c3b0ae6ee9d44a7786b0990e6ea39e5d_0)|
| WCC suburb boundaries  | .gdb | [Wellington City Council](https://data-wcc.opendata.arcgis.com/datasets/f534738cf3e648f7b1524a9697376764_0) |
| StatsNZ 2019 meshblock boundaries | .gdb | [Stats NZ](https://datafinder.stats.govt.nz/layer/98971-meshblock-higher-geographies-2019-generalised/data/) |
| Wellington street network without elevation | - | OpenStreetMap via osmnx |
| Wellington street network with elevation | - | OpenStreetMap + Google Elevation API via osmnx|



# Accessibility by Wellington suburb

![](../images/2019-03-12-Modelling-accessibility-by-suburb/output_18_0.png)


## Visualising accessibility within suburb boundaries


![](../images/2019-03-12-Modelling-accessibility-by-suburb/output_20_0.png)


![](../images/2019-03-12-Modelling-accessibility-by-suburb/output_21_0.png)


# Bayesian Modelling of accessibility
This section is all about writing Bayesian models with Stan.


```python
uni_norm_model = su.load_or_generate_stan_model('stan', 'univariate_normal')
lower_trunc_norm_model = su.load_or_generate_stan_model('stan', 'lower_truncated_univariate_normal')
trunc_norm_model = su.load_or_generate_stan_model('stan', 'truncated_univariate_normal')
```


## Normal Model
Inference for Stan model: anon_model_cc3fc1beb21cbbe7b94ad66105c98210.
4 chains, each with iter=2000; warmup=1000; thin=1;
post-warmup draws per chain=1000, total post-warmup draws=4000.

         mean se_mean     sd   2.5%    25%    50%    75%  97.5%  n_eff   Rhat
mu      22.99  6.3e-3   0.35   22.3  22.75  22.99  23.23  23.67   3113    1.0
sigma   14.28  4.3e-3   0.26  13.79   14.1  14.27  14.45  14.79   3630    1.0
y_pred  22.71    0.23   14.3  -5.62  12.89  22.82  32.59  50.67   4000    1.0
lp__    -5086    0.02    1.0  -5089  -5087  -5086  -5085  -5085   1809    1.0

Samples were drawn using NUTS at Mon Mar 18 15:07:35 2019.
For each parameter, n_eff is a crude measure of effective sample size,
and Rhat is the potential scale reduction factor on split chains (at
convergence, Rhat=1).


## Truncated Normal model for better fit

| Normal Model | Truncated Normal Model |
| :----------: | :--------------------: |
|![](../images/2019-03-12-Modelling-accessibility-by-suburb/output_26_0.png) |![](../images/2019-03-12-Modelling-accessibility-by-suburb/output_50_0.png) |


## Checking model performance with posterior predictive

### Good fit
![](../images/2019-03-12-Modelling-accessibility-by-suburb/output_34_0.png)


### Reasonable fit with some issues
![](../images/2019-03-12-Modelling-accessibility-by-suburb/output_49_0.png)

 - Reasonable fit
 - Doesn't capture modes - likely due to the fact that Rongotai has both a residential and an industrial area.

### Poor fit
![](../images/2019-03-12-Modelling-accessibility-by-suburb/output_35_0.png)

- Poor fit
- Model is overwhelmed by the large spike at 60 minutes
- Makara is basically semi-rural and shouldn't be modelled the same as urban suburbs


## Hierarchical modelling

## Results for $\mu$
![](../images/2019-03-12-Modelling-accessibility-by-suburb/output_43_0.png)


## Results for $\sigma$
![](../images/2019-03-12-Modelling-accessibility-by-suburb/output_44_0.png)

## Quadrant visualisation of $\sigma$ and $\mu$
![](../images/2019-03-12-Modelling-accessibility-by-suburb/output_45_0.png)

| suburb | quadrant | $\mu$ | $\sigma$ |
|--- |--- |--- |--- |
|Khandallah|High $\sigma$ and $\mu$|8.685588|4.641167|
|Karori|High $\sigma$ and $\mu$|7.424094|8.664240|
|Tawa|High $\sigma$; Low $\mu$|-6.406689|2.839867|
|Brooklyn|High $\sigma$; Low $\mu$|-7.285322|15.768424|
|Pipitea|Low $\sigma$; High $\mu$|16.522374|-2.964546|
|Hataitai|Low $\sigma$; High $\mu$|7.069155|-3.707169|
|Wellington Central|Low $\sigma$; High $\mu$|4.671688|-2.408587|
|Kelburn|Low $\sigma$; High $\mu$|5.273022|-2.693675|
|Miramar|Low $\sigma$; High $\mu$|3.321520|-2.618647|
|Mount Cook|Low $\sigma$; High $\mu$|2.831357|-4.369413|
