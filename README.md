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

### Integrals in Underworld2

Underworld2 currently supports area (volume) and surface integrals. But there is no inherent capacity to integrate a 2D function independently along separate axes. However, we can generate a Fourier /(wavenumber) representation of such an integral.

Consider a 2D scalar field $F(x,y)$, where we want to determine the function representing the the integrated value over the y-axis:

$$f(x) = \int F(x,y) \mathop{dy}  = A_0 +  \sum_{k=1}^N a_k \cdot cos(kx)  +  b_k \cdot sin(kx) $$


$$A_0 = \int_\Omega F(x,y) \mathop{dx}\mathop{dy}  $$


$$a_k = \int_\Omega  cos(kx)\cdot F(x,y)  \mathop{dx}\mathop{dy} $$

$$ b_k = \int_\Omega  sin(kx)\cdot  F(x,y)\mathop{dx}\mathop{dy}  $$


The Underworld2 `function` module provides all the necessary apparatus us to compute these integrals. This proceedure provides the following advantages:

* it can be performed in parallel, and hence during Underworld simulations, rather than as a post-processign step
* the proceedure utilises (numerical) Fourier integrals (rather than FFT), meaning that the proceedure can be applied to deformed meshes.

These spectral integrals are, however, relatively expensive to compute (i.e compared, to a similar proceedure using FFTs). They are therefore likely to be practical only in 2D.

A specific application that can make use of this proceedure is the spectral domain calculation of gravity anomalies, also known as __upward continuation__.

### A simple example

First, let's consider a simple example, with a 2D function over the unit square with an origin at (0,0):


$$ F(x,y) = \cos(\pi x) * \cos(\pi y)$$

$$f(x) = \int F(x,y) \mathop{dy}  = \frac{2}{\pi} \sin(\pi x) $$

$$ A_0 = \int_{-0.5}^{0.5} \int_{-0.5}^{0.5} F(x,y) \mathop{dx}\mathop{dy}  = \left(\frac{2}{\pi} \right)^2 $$

[This](https://github.com/dansand/gravityPt1/blob/master/pt1_Integrals.ipynb) notebook shows how we can calculate the integral using Underworld2 functions.


### Upward continuation of gravity

Given a known subsurface density structure $\rho(x,y)$, the surface (upward continued) gravity signal can be written as:

$# \Delta g (x) = 2\pi G \left[ \sum_k e^{ikx} \int_{z_0}^{z_t} \hat \Gamma(k,z)e^{-kz} dz  \right]$$

[This](https://github.com/dansand/gravityPt1/blob/master/pt2_gravityEx1.ipynb) notebook compares the spectral upward continuation approach with an analytic solution for a buried cylinder.

### References

Parker, R. L. "The rapid calculation of potential anomalies." Geophysical Journal International 31.4 (1973): 447-455.
