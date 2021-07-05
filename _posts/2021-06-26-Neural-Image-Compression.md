---
layout: post
title:  "Neural image compression"
date:   2021-06-26 01:03:29 +0200
categories: research
comments: true
---
Understanding neural network approaches in image compression.

<!--more-->
Compression is the ability to store the data with lesser size (number of bits) and being able to reconstruct back the original version. It can be lossless - perfect reconstruction or lossy - some level of distortion in reconstruction. It is a vital tool in today's digital world where more than 1 quintillion bytes is created every day. Being able to lower the compression rate and speed is very beneficial to achieve lower storage, bandwith etc. In this post, I'll go though some of work done in Neural Image Cpmpression in the years 2017-2018. Images are commonly used in formats such as .jpeg, .tiff, .png etc. How are neural networks store and retrive images? Let's find out.


<b>Lossy Image Compression with Compressive Autoencoders</b>:<br>
In this work [1], the authors focus on lossy compression. The main limitations on neural network approaches in lossy compression is due to the loss and quantisation being non-differentiable.
To overcome this, simple modifications are introduced in this paper.

A compressive autoencoder is defined to have an encoder <b>f</b>, decoder <b>g</b> and a probablistic model <b>Q</b>. The probalitistic model is responsible for entropy coding. The goal is to optimise trade off between number of bits and distortion.
![tradeOff alt ><]({{ '/assets/images/tradeOff.png'}}){:height="70px" width="300"}
Quantisation is performed by rounding off to nearest interger and <b>β</b> controls the tradeoff.
In the above equation, quantisation and entropy coding is not differentiable.

To overcome it, smooth approximation is used for quantisation in the backward pass. A continuos, approximate intergral is used for entropy encoding in the region it is not differentiable. An unbiased estimate of the upper bound is taken by sampling over the space. A trainable scale parameter is included which can be used to fine tune the model repeatedly with different entropy rates to encourage variable entropy usage. The loss now becomes,
![scale alt ><]({{ '/assets/images/scale.png'}}){:height="70px" width="500"}
Gaussian mixture scale is used to model the quantisation coefficients with 6 scales over channels. This is used based on prior work on GSM towards modelling filter responses in images. The overall model architecture is shown below,
![compress1arch alt ><]({{ '/assets/images/compress1arch.png'}}){:height="70px" width="700"}
The training is done incrementally, starting with most of the coefficeints masked. They are unmasked one by mask determeined by a training threshold. After training it, it is fine tuned for different rate-distortion trade-off.

The authors compare the decompressed images with JPEG 2000 and obtain better scores on metrics such as MOS (Mean Opinion Score), SSIM ( Structural Similarity Index). This model allows reconstruction at different bit rates. This can be very benificial on practical use-case since neural networks usually do not provide customisations. Though the ideas introduced are simple, there are still a lot of design elements in training this model incrementally. It should be noted that only 434 images were used to train it and tested with 24 images from a different dataset. While the cross dataset metrics may be sufficent to prove generalisability, it is still very less data considering the scale at which models are trained currently. It was trained for over 100k iterations, this might be one reason why less number of images were considered for training. Overall, a good paper to start with neural compression.

<b>Variational image compression with a scale hyperprior:</b><br>
In this work [2], the authors peform image compression with the variational vatoencoder (VAE). Minimising the KL divergence in VAE is seen as optimising the compression model, as shown below.  
![klcompress alt ><]({{ '/assets/images/klcompress.png'}}){:height="120px" width="700px"}
The inference density <b>q</b> is made 0 by choosing a uniform distribution. The likelihood is the reconstruction loss., a squared error loss in case of gaussian distribution. The third term is seen as cross entropy between the marginal and the prior of the latent. The prior is defined as -
![paper2infer alt ><]({{ '/assets/images/paper2infer.png'}}){:height="120px" width="400px"}

During training, quantisation is perfomed by adding uniform noise in the inference process as shown below. In this equation, <b>y</b> is obtained by the encoder. The prior is modelled as shown below as a nojn paramteric fully factorised density model.
![priorcompress alt ><]({{ '/assets/images/priorcompress.png'}}){:height="120px" width="450px"}

Here, <b>Ψ</b> contains the parameters of each univariate distribution. All the densities are convolved with standard uniform density. The authors visualise quantised latents and infer that probabilistic coupling is present which is not represented by the prior used, Due to this, usage of hyperprior is visited.

Additional random variable <b>z</b> is introduced from which standard deviation is predicted. This i used to contruct a zero mean gaussian convolved with unit uniform as preiously used as show below.
![priorcompress2 alt ><]({{ '/assets/images/priorcompress2.png'}}){:width="500px"}
The inference now consists of both <b>y</b> and <b>z</b> as shown,
![priorcompress3 alt ><]({{ '/assets/images/priorcompress3.png'}}){:width="700px"}
The loss now has the distrion term, cross entropies on <b>y</b> and <b>z</b>. The model structure is shown below. All the different sub networks consist of convolution+relu blocks.
![priorcompress4 alt ><]({{ '/assets/images/priorcompress4.png'}}){:width="700px"}
One million images were retrieved from the web and screened further and then used for trained. Evaluation performed as Kodak daraset with metrics such as PSNR and MS-SSIM.

The authors utilise the variational inference framework as a compression mechanism. They use uniform noise to quantise the inference, which simplifies the VAE loss to reconstruction error and entropy of latent. They find that using a non paramteric prior is not representative enough of the latent space so they define another latent variable. A standard deviation is predicted from the second prior and used to construct a zero mean gaussian with the first prior. While hierarchical generative modelling has been visited, this wprk combines it in the compression framework.

<b>Conditional Probability Models for Deep Image Compression:</b><br>

In this paper [3], the authors model the compression with a 3D CNN encoder and decoder. A quantiser is present in the bottleneck to discretize encoder outputs, which can be encoided with a bitstream. This can be decoded and passed through the decoder to reconstruct the input image.
The quanisation is restricted to finite set of centers and processed as:
![paper3eqn alt ><]({{ '/assets/images/paper3eqn.png'}}){:width="300px"}
This has a modification in the backward pass to make it differntiable.
![paper3eqn2 alt ><]({{ '/assets/images/paper3eqn2.png'}}){:width="300px"}
This has a differentiable version for the backward pass, following the paper paper mentioned here. To model the entropy, the latent distribution is factorised as a product of consitional distributions, by indexing the 3D latent in rastor scan. This idea is utilised from PixelCNN, pixelRNN papers, which using masking to ensure the causality required for conditional. With this, another 3D CNN is used on the latent, which predicts which symbol should be present at each location and trained using cross entropy loss.
In order to reduce the number of bits in latent, it is multiplied by a importance map acquired by a learnable mask. This is done by comparing values in spatial locations with channel number. Due to this, some of the channels can be zero. This masking is done after the context model, which acts as the quantisation step.
![paper3fig alt ><]({{ '/assets/images/paper3fig.png'}}){:width="400px"}
The models are trained using MS-SSIM metric with the imagenet dataset and tested on imagenet, B100, Urban100, Kodak datasets.
This paper utilises masked convultion perfomed in PixelCNN and extends it to 3DCNN and views it as a compression tool. It also uses importance map techniques utilised in the field and incorporates it to quantise in this architecture.

<b>Joint Autoregressive and Hierarchical Priors for Learned Image Compression: </b><br>
This work [4] extends on the work done in papers [2], [3]. The architecture introduced consists of an encoder and decoder with an autoregressive context model. It also includes a hyper encoder and hyper decoder which works on the encoder latents. The hyper deocder and cotext model outputs are used to predict both mean and scale paramters for the entropy model. The entire setup is shown below:
![paper4arch alt ><]({{ '/assets/images/paper4arch.png'}}){:width="600px"}
Each encoder latent is modelled as a gaussian convolved with an unit uniform distribution and the hyper latent modelled as a non-paramteric factorised density model. The loss has 3 terms - reconstruction error (distrotion), encoder latent cross entropy and hyper encoder latent cross entropy as shown below:
![paper4loss alt ><]({{ '/assets/images/paper4loss.png'}}){:width="600px"}
Peak Signal to noise ratio (PSNR) is used as a metric to measure compression capability. Other metrics such as MSE and MS-SSIM are also reported on Kodak dataset.

<b>Discussion:</b><br>
The neural compression scheme has two components to it - compression rate and distortion. A trade off exists between the two as better reconstruction can be obtained with larger latent structure to derive it from. The compression process involves a feature transformation into a latent space followed by quantisation. This can be stored as bits through arithmetic coding. This is then decoded and recontructed back into images. This is naturally perfomed by an autoencoder framework, where the entopy of the bits used to store the file can be represented by the entropy of the latent space and the distortion can be measured by the reconstruction error. The papers summarised in this post is from 2017-18, so it may not be the state of the art as of 2021. The ideas here mainly consists of either a autoencoder with variational framework or models in latent space like PixelCNN for entropy coding. This field is closely related to generative modelling so it is natural to find that SOTA generative models do well here. The techniques are usually validated with metrics such as PSNR and MS-SSIM on datatsets such as kodak. It should be interesting to revisit this field again and look at some of the recent works soon.
References:
1. https://arxiv.org/abs/1703.00395
2. https://arxiv.org/abs/1802.01436
3. https://arxiv.org/abs/1801.04260
4. https://arxiv.org/abs/1809.02736
