---
layout: blog_default
title:  "Spherical harmonics and soft shadows"
date:   2020-12-05 00:20:59 +00:00
image: /images/CGT620.png
categories: blog
author: "Yichen Sheng"
permalink: /:categories/:year/:month/:day/:title.html
---
# Spherical Harmonics and Soft Shadows

## Motivation
The rendering equation shows we need to solve the integral equation recursively to render a good image. But solve the integral equation is intractable. A straightforward solution is the Monte-carlo integration. But it still suffers from high computation cost and high variance. If we "relax" the quality a little bit, we can turn to find some approximation algorithms. "Good" basis will help us approximate the equation. 

<!-- ![](/images/SH/basis1.png)
![](/images/SH/basis2.png)
![](/images/SH/basis3.png) -->

<center> <img src="/images/SH/basis1.png"> </center>  
<center> <img src="/images/SH/basis2.png"> </center>  
<center> <img src="/images/SH/basis3.png"> </center>  

<center> 
The first image shows project the integral function into the basis. The second image shows use the projected coefficients to reconstruct the original function. The third image shows the reconstructed/approximated functions.(Credit: Robin Green) 
</center>

By projecting the function into the basis, we just need to do a dot product to reconstruct the original function. Let's recap the rendering equation again:

$$     L(x, \omega_o) = L_e(x, \omega_o) + \int_S f_r(x, \omega_i, \omega_o) L(x', \omega_i)G(x,x')V(x,x') d\omega_i $$

Instead of solving the equation, there is a set of orthonormal basis that approximates the function extremly fast after some precomputation. This magic set of basis is called **spherical harmonics (SH)**.  

## What is Spherical Harmonics(SH) 
Spherical harmonics(SH) are functions defined on the (unit) sphere surface: 

$$ f(\theta, \phi): R^2 \rightarrow R $$

From a high level view, SH defines a set of orthonormal basis functions on the sphere surface. Due to the orthonormal property, it can be a potential tool for basis functions to solve some integrals, e.g. the rendering equations. 

To be specific: 

$$      y_l^m(\theta, \phi) = 
\begin{cases}
      \sqrt{2}K_l^m cos(m\phi) P_l^m(cos\theta), m > 0\\
      \sqrt{2}K_l^m sin(-m\phi) P_l^{-m}(cos\theta), m < 0\\
      K_l^0 P_l^0 cos(\theta), m = 0\\
\end{cases}     $$

$$     K_l^m = \sqrt{\frac{(2l+1)}{4\pi} \frac{(l-|m|)!}{(l+|m|)!}} $$

$$ l \in \mathbb{R}^+, -l \leq m \leq l $$

Here $$P(\cdot)$$ is a 1D Legendre polynomials. 

$$ \begin{align}
    (l-m)P_l^m &= x(2l-1)P_{l-1}^m - (l+m-1)P_{l-2}^m \\
    P_m^m &= (-1)^m (2m-1)!!(1-x^2)^{\frac{m}{2}}\\
    P_{m+1}^m &= x(2m+1)P_m^m\\
\end{align} $$

Those functions seem to be a little scary for the first time. But the implementation is very simple, see the [source](https://github.com/ShengCN/Spherical-Harmoincs-Shadows/blob/final_project/spherical_harmonics.cu#L73).

Another important feature of SH is that it keeps the low frequency details while filtered high frequency details. It is an anology to the fourier transform that maps the spatial domain to frequency domain. 

## Soft Shadows
As we discussed above, SH can be used to render **global illumination** effects. Soft shadows are one of the "side product" from **global illumination**. Soft shadows are the occlusion effects from surrounding objects. The exact value can be be formally defined as: 

$$ L(x) = \int_S V(x, \omega_i) d\omega_i$$

Since SH can help general integral equations, it can also be used for rendering soft shadows. An interesting question is what is the softness of the soft shadow as the order of the SH basis goes higher? Based on this question, I did this experiment.

## Experiments
In the scene, I set up a very small area light in the IBL. A bunny is standing on the plane as an occluder for the ground. 
 <table style="width:100%">
  <tr>
    <td>
        <figure>
            <img src="/images/SH/0004.png" style="width:100%"/>
            <figcaption style="text-align: center;">4th order</figcaption>
        </figure>
    </td>
    <td>
        <figure>
            <img src="/images/SH/0005.png" style="width:100%"/>
            <figcaption style="text-align: center;">5th order</figcaption>
        </figure>
    </td>
  </tr>
  <tr>
    <td>
        <figure>
            <img src="/images/SH/0006.png" style="width:100%"/>
            <figcaption style="text-align: center;">6th order</figcaption>
        </figure>
    </td>
    <td>
        <figure>
            <img src="/images/SH/0007.png" style="width:100%"/>
            <figcaption style="text-align: center;">7th order</figcaption>
        </figure>
    </td>
  </tr>
  <tr>
    <td>
        <figure>
            <img src="/images/SH/0008.png" style="width:100%"/>
            <figcaption style="text-align: center;">8th order</figcaption>
        </figure>
    </td>
    <td>
        <figure>
            <img src="/images/SH/0009.png" style="width:100%"/>
            <figcaption style="text-align: center;">9th order</figcaption>
        </figure>
    </td>
  </tr>
  <tr>
    <td>
        <figure>
            <img src="/images/SH/0010.png" style="width:100%"/>
            <figcaption style="text-align: center;">10th order</figcaption>
        </figure>
    </td>
    <td>
        <figure>
            <img src="/images/SH/0011.png" style="width:100%"/>
            <figcaption style="text-align: center;">11th order</figcaption>
        </figure>
    </td>
  </tr>
</table> 

<figure>
    <img src="/images/SH/gt.png" style="width:100%"/>
    <figcaption style="text-align: center;">Ground Truth(100 samples)</figcaption>
</figure>

As the order goes higher, the softness is decreasing. But actually for even 11th order(121 bases), the shadow is still a little blurry. 

## Conclusion
Pros:

* Efficient rendering in runtime
* General basis for integral of spherical surface
* Low frequency details are kept for low orders

Cons: 

* Not suitable for hard shadow rendering (of course)
* Shadow resolution may depends on the resolution of the shadow receiver
* For static scene only

It's not a unexpected results. The experiment seems to be uninteresting. But another problem becomes much more interesting. How to render all frequency shadows using basis approximation? When I did literature review for my [paper](https://arxiv.org/pdf/2007.08211v2.pdf), I found this paper solved this problem: [All-frequency shadows using non-linear wavelet lighting approximation](https://dl.acm.org/doi/abs/10.1145/1201775.882280?casa_token=jAc1H1E-uXUAAAAA:2XwnWZM4kZ0Mnbps934rRZhODXgQIk1wXx1Ahp3vt4YdOK3pcaHQ0OJVbirTARYmo0Huw63Gf8uj7w). The basic idea is to use Harr wavelet to compress the environment map and construct a shadow basis matrix, which is very similar to the shadow bases in our paper. It is quite interesting to find the coming ideas solved by others years ago.  