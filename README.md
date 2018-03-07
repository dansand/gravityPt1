---
layout: post
title:        Gravity calculation in Underworld2
description: 'Examples of how to compute gravity / geoid using spectral integrals'
category: ['Gravity','geoid', 'FAQ'] #  categories which are automatically filtered: faq, developer
date: 2018-03-05  
comments: true
author: Dan Sandiford & John Mansour
image:  # over-ride default with feature as image path
    feature:
---

### Overview

The aim of this post is to demonstrate a not-so-obvious application of Underworld2 volume integration; namely that we can generate a spectral representation of integrals over a single axis (a functionality which is not directly supported). This is of interest primarily because it has applications to gravity calculation (upward continuation). 

### Integrals in Underworld2

Underworld2 currently supports area (volume) and surface integrals. But there is no direct capacity to integrate a 2D function independently along separate axes. However, we can generate a spectral (Fourier) representation of such an integral. Throughut this post, we'll refer to this procedure as "spectral integration". 

Consider a 2D scalar field $F(x,y)$, where we want to determine the function representing the integrated value over the y-axis:

\begin{equation}f(x) = \int_0^L F(x,y) \mathop{dy}   \end{equation}

We can write this as a sum of periodic functions:


\begin{equation}f(x) = \frac{1}{W}  A_0  + \sum_K \frac{2}{W}  a_k \cos(kx) + \frac{2}{W}  b_k sin(kx) \end{equation} 

where:

\begin{equation} a_k = \int_0^L  \int_0^W \big( \mathop{F(x,y)} \cos(k x) \big)\mathop{dx}\mathop{dy} \end{equation} 


\begin{equation} b_k = \int_0^L  \int_0^W \big( \mathop{F(x,y)} \sin(k x) \big)\mathop{dx}\mathop{dy} \end{equation} 


\begin{equation} A_0 = \int_0^L  \int_0^W  \mathop{F(x,y)} \mathop{dx}\mathop{dy} \end{equation} 

The Underworld2 `function` module provides all the necessary apparatus us to compute these volume (area) integrals, and hence find the spectral coefficients  $A_0, a_k, b_k $.

This procedure provides the following advantages:

* it can be performed in parallel, and hence during Underworld simulations, rather than as a post-processign step
* the procedure utilises (numerical) Fourier integrals (rather than FFT), meaning that the procedure can be applied to deformed meshes.

These spectral integrals are, however, relatively expensive to compute (i.e compared to FFTs/quadrature on the same sized arrays). They are therefore likely to be practical only in 2D.

A specific application that can make use of this procedure is the Fourier domain calculation of gravity anomalies, also known as __upward continuation__.


## Examples


### An introductory example

To introduce the concepts, we start by considering the problem of integrating a scalar 2D function ($F(x,y)$) over the y axis:

$$f(x) = \int F(x,y) \mathop{dy} $$

[This](https://github.com/dansand/gravityPt1/blob/master/pt1_Integrals.ipynb) notebook shows how we can calculate the function $f(x)$ using Underworld2 spectral integration.


### gravity upward continuation

Given a known subsurface density structure $\rho(x,y)$, the surface gravity signal (upward continued) can be written as:

$# \Delta g (x) = 2\pi G \left[ \sum_k e^{ikx} \int_{z_0}^{z_t} \hat \Gamma(k,z)e^{-kz} dz  \right]$$

We can perform this gravity calculation using spectral intregration. Compared with the first example, the primary difference is that we have to build the Underworld function for $e^{-kz}$ (which is known as the upward continuation kernel). 

[This](https://github.com/dansand/gravityPt1/blob/master/pt2_gravityEx1.ipynb) notebook demonstrates gravity upward continuation using spectral integration. The proceedure is benchmarked against an analytic solution for a buried cylindrical density anomaly.

###  Gravity anomalies for isoviscous convection

Lateral density anamolies contribute to surface gravity anomalies directly, as well as through perturbations to the mantle boundaries (dynamic topography). In [this](https://github.com/dansand/gravityPt1/blob/master/pt3_gravityEx2.ipynb) notebook, we compare our numerical proceedure to an analytic solution for (isoviscous) convection in the mantle. This demonstrates how to account for the perturbation (topgraphy) of radial density transitions, such as the core mantle boundary. 


### References

Parker, R. L. "The rapid calculation of potential anomalies." Geophysical Journal International 31.4 (1973): 447-455.

Parsons, Barry, and Stephen Daly. "The relationship between surface topography, gravity anomalies, and temperature structure of convection." Journal of Geophysical Research: Solid Earth 88.B2 (1983): 1129-1144.

Davies, Geoffrey F. "Mantle convection under simulated plates: effects of heating modes and ridge and trench migration, and implications for the core—mantle boundary, bathymetry, the geoid and Benioff zones." Geophysical Journal International 84.1 (1986): 153-183.

King, Scott D. "On topography and geoid from 2‐D stagnant lid convection calculations." Geochemistry, Geophysics, Geosystems 10.3 (2009).