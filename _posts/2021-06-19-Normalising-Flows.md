---
layout: post
title:  "Normalising Flows"
date:   2021-06-19 12:03:29 +0530
categories: research
comments: true
---
This is an overview of recent progress in generative models with Normalising Flow approach.

<!--more-->
Generative modeling encompasses a range of techniques applied to learn the inherent distribution of data. There have been a lot of approaches in recent years which uses deep neural network to parameterise distributions. This post will cover some of the works which introduced and solidified Normalising Flow based generative models.<br>

<b>NICE: Non-linear Independent Components Estimation:</b><br>
The motivation for this work is to learn a transformation over the data (<b>h = f(x)</b>), such that the resultant distribution can be factorised into independent components.

![firstEqn alt ><]({{ '/assets/images/firstEqn.png'}}){:height="50px" width="172px"}
By considering the transformation to be invertible, and dimension of <b> h</b> same as <b>x</b>, change of variable can be used, from <b>x</b> to <b>h</b>. Then the change of variable rule can be applied so that,
 ![changofvar alt ><]({{ '/assets/images/changeofvar.png'}}){:height="100px" width="250"}
<b>f</b> is chosen such that the Jacobian is obtained easily and <b>f<sup>-1</sup></b> is available to sample <b>x</b>. In this work, the authors split the data into <b>x<sub>1</sub></b>, <b>x<sub>2</sub></b> and learn transformation  <b>y<sub>1</sub></b>, <b>y<sub>2</sub></b>. This is perfomed as follows:
![tfsm alt ><]({{ '/assets/images/tfsm.png'}}){:width="160"}
where <b>m</b> is a MLP with Relu activation. This can be easily inverted as shown below.
![tfsm2 alt ><]({{ '/assets/images/tfsm2.png'}}){:width="160"}
This can be learnt using Maximum likelihood,
![mle alt ><]({{ '/assets/images/mle.png'}}){:width="440"}
Here, the prior <b>p<sub>H</sub></b> can be a normal distribution. If the prior is factorial, then the maximum likelihood will be a deterministic transformation of a factorial distribution.
![mle2 alt ><]({{ '/assets/images/mle2.png'}}){:width="465"}
While inverting can lead to contracting the data, the Jacobian can be veiwed as penalty to contraction and encourages expansion to  high density regions.
The transformations <b>f</b> can be a set of sequential layers, where the Jacobian will be the product of each layers' Jacobian determinants. The basis for design choice is high expressivity with easy determinant computation. This is obtained by the coupling where <b>y<sub>1</sub></b> is an indentiy w.r.t x and <b>y<sub>2</sub></b> is a function of coupling law, such that the derivative becomes,
![det alt ><]({{ '/assets/images/det.png'}}){:width="190"}
thus, the determinent of the derivative = 1. Different modification of the coupling law can be used such as additive, multiplicative etc. The two subsets are exchanged in between many stacked coupling layers so that all dimensions are explored.
Since the layers have unit Jacobian determinant, a scaling layer is used at the end which weights the different dimensions.
The authors obtain likelihood scores close to the expected values on 3 image datasets.   
The authors introduce a simple-novel learning framework to learn invertible mapping between the data and transformation by training with exact likelihood. While the samples obtained are not excellent, this work was done in 2014 during the early days of deep generative models. This led to a lot of work in this sub-field.
<br><br><b>Variational Inference with Normalizing Flows:</b><br>
This works employs Normalising flows with variational inference to obtain better posteriors. In practice, different techniques such as mean field approximation is used with variational inference, due to which the true posterior is not reached. If we consider the Evidence Lower Bound (ELBO), the KL divergence between approximate and true posterior, it is zero when both are the same. Usually, gaussians are used to represent them due to which it will not match, most of the times. Thus, normalising flow offers a way to learn complex distribution to match the true posterior. The benifit to it is that we will not have the complex distribution available at the end, only that it will match the true posterior, so it eliminates the problem of hand-designing it to match it.   
![flow1 alt ><]({{ '/assets/images/flow1.png'}}){:width="190"}
The authors consider a class of transformations represented in the above equation, where <b>u</b>, <b>w</b> and <br>u</br> are parameters and <b>h</b> is a smooth element-wise differentiable non-linearity.
![flow2 alt ><]({{ '/assets/images/flow2.png'}}){:width="400"}    
The above formulation is referred to as planar flows. The authors also define another class of transformation that modify the initial density around a reference point, as shown below.
![flow3 alt ><]({{ '/assets/images/flow3.png'}}){:width="230"}    
![flow4 alt ><]({{ '/assets/images/flow4.png'}}){:width="470"}    
This is referred to as radial flows. The densities with the two types of flows is shown below, The first image showcases the initial density determined by the prior and the subsequent images represent densities with <b>K</b> flow steps. Different representations can be learnt with changes in how the transformation is defined. This allows a good amount of control in the latent structures.
![flow5 alt ><]({{ '/assets/images/flow5.png'}}){:width="600"}
The network can be trained replacing the KL term in ELBO by the Flow loss, use an inference model along with the flow parameters.

This work provides a way to reach closer towards the true posterior in a VAE setup. This also offers different types of flows.   

<b>Denisty estimation with Real-NVP:</b>
This is an improvement on the NICE paper. Instead of restriciting to a unit Jacobian (volume preserving), the transformation is designed such that a triangular matrix is obtained with which the determinant is the product of diagonal elements.
![flow6 alt ><]({{ '/assets/images/flow6.png'}}){:width="375"}
The determinant is,
![flow7 alt ><]({{ '/assets/images/flow7.png'}}){:width="375"}
Spatial checkboard and channel wise masking is perfomed to obtain two subsets of the data, so that they can be visited in alternate layers. A squeezing operation is defined which reduces the input image size by a factor of 2 and increase the number of channels by a factor of 2, trading receptive feild with channels. Three coupling layers with alternative checkboard is applied, followed by a squeeze operation and then 3 more coupling layers with channel wise mask. Half the channels are taken out at regular intervals to reduce computation.

This work improved the transformations applied on the normalising flow. It is still easy to compute the Jacobian while having much more expressice layers.

<b>Glow: Generative Flow with Invertible 1×1 Convolutions:</b><br>
This work defines a single transformation step with 3 operations- Actnorm, Invertible 1 x 1 convolution, affine coupling layer. Actnorm is a normalisation similar to batchnorm where per-channel transformation is performed with a scale and bias parameter.
These are initialised such that the output is standarised, after which the actnorm parameters are trainable. The invertible 1 x 1 convolution replaces the permutation operation in real NVP. This can be optimised by LU Decomposition of the convolution matrix.
The architecture is trained in a large scale by having multiple flows with multiple steps in each flow.
This work achieves remarkable synthesis perforance with very high quality images. This showcased that normalising flows can generate as good as SOTA generative models from other techniques such as autoregressive, GANs etc.

While there are much more work done with Normalising Flows along with its various applications in related(eg: ODEs) and unrelated (text synthesis) domains, this post covers the introductory work done which popularised the method. There are many reasons why normalising flows is a great tool for generative modelling. It allows for modelling to complex distributions from simple priors with which samples can be obtained easily. The loss is based on exact likelihood calculation, without the need for reconstruction loss which is common in most of the other generative models. The simple yet tricky methods desrcibed in these works for the transformations are great to have efficent loss computation. These models also used such that training and synthesis have similar computation.
