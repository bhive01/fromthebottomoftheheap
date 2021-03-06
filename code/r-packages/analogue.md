---
title: analogue
subtitle: palaeoecological data analysis with R
status: publish
layout: page
published: true
type: page
tags: []
active: code
---
## What is analogue?
**analogue** is an R package for use with palaeoecological data. Originally, **analogue** was intended as an R implementation of analogue methods such as analogue matching, <acronym title="Receiver Operator Characteristic">ROC</acronym> curves, and <acronym title="Modern Analogue Technique">MAT</acronym> transfer function models, and the computation of dissimilarity coefficients. Since then the scope of the package has grown to include a number of other methods applicable to data routinely encountered in palaeoecology and palaeolimnology.

### Features

 * Transfer functions
     * MAT
     * Weighted Averaging with monotonic, inverse, and classical deshrinking, with and without tolerance down-weighting
     * Principal Component Regression (using ecologically-relevant transformations)
     * Cross-validation (Bootstrapping, leave-one-out, *k*-fold)
     * Analogue statistics
 * Analogue matching
 * Dissimilarity coefficients
     * Chord, Bray-Curtis, Gower's Generalised coefficient, Manhattan, ...
 * Dissimilarity decisions thresholds
     * <abbr title="Receiver Operator Characteristic">ROC</abbr> curves
     * Monte Carlo resampling
     * Logistic regression
 * Stratigraphic diagrams
 * Principal curves
 

## Bugs, feature requests
Bug reports and feature requests should be filed as [issues](https://github.com/gavinsimpson/analogue/issues) on [github](https://github.com).

## Licence
analogue is released under the [GNU General Public Licence Version 2](http://www.gnu.org/licenses/gpl-2.0.html).

## Links

 * [CRAN page](http://cran.r-project.org/package=analogue)
 * [Development site](https://github.com/gavinsimpson/analogue/) on [github](https://github.com)
