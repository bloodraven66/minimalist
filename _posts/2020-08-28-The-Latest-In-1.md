---
layout: post
title:  "The latest in Self-supervised"
date:   2020-08-28 01:03:29 +0200
categories: research
comments: true
---
An overview of recent progress in self-suprvised learning research in DL.

<!--more-->

NOTE: not completed

Supervision is an important aspect of teaching machines. Almost all the AI based products being in use are a result of accurate supervised data during the training stages. In the recent years, a lot of work has been put in towards learning data without labels. The creation of an algorithm which can reliably perform this can drastically change the use of Machine learning for real world applications.
<br>
Self-supervision has been studied from a long time. Geoffrey Hinton developed pretraining technique for training autoencoders in the mid-2000s, which is known as the restricted boltzman machine. After the deep learning boom, work has been building up towards creating a learning framework which is discriminative without the presence of labels. Jigsaw solving exploits the dependence between different regions of an image. This paper uses rotation as a discriminating factor to train the network. Recently, huge improvements have been made in the field using contrastive loss and data augmentation.

Contrastive loss has been in use for over a decade in works such as [1], [2]. In NLP, with Word2Vec, the surrounding words are predicted using contrastive loss. Qouting [2] - 'In less mathematical terms, the idea behind noise-contrastive estimation is learning by comparison'. It is used as a loss function on latent space projections of input data. How exactly it is done, what is the data and what it is compared to is the crux of research in this field. I will go through some of the recent papers utilising contrastive learning to learn without supervision.



- <b><a href="https://bloodraven66.github.io/2020/08/27/The-Latest-In-1#cpc-paper">Representation Learning with Contrastive Predictive Coding[3]</a></b>
This work combines autoregressive modelling with a contrastive loss to train the model in an end to end manner. The authors present the results in 4 domains - vision, text, speech and RL. Initially an encoder projects the input data into a compact latent space. An autoregressive model is used to make predictions of next latent representation. <br>
![cpc]({{ '/assets/images/Selection_068.png' | relative_url }}){: class="center"}
<br><br>
The authors reason that predicting the next time step data from latent space is computationally expensive and often ignores the context of previous time step. So to learn the features of the data in use, they encode both the latent embedding and the target and compare it with a contrastive loss. They name their contrastive loss as InfoNCE which is as follows:
<br>
![infonce]({{ '/assets/images/Selection_070.png' | relative_url }}){: class="center"}
<br>
This loss function tries to maximise the mutual information between the encoded representations. For training, 2 sets of examples are drawn from a distribution. When the embedding and predicted embedding are from same input, it is a positive pair and when they are from different inputs, then it is called a negative pair. MI is maximised for positive pairs while it is minimised for negative pairs. This way, the model learns the features of a particular data point. The authors use strided convolutions and GRU for the autoregressive model.
<br>
I think this paper is pretty impressive based on it's performance on various domain data, showing the significance of such methods to generalise towards any real world data.
<br>


- <b><a href="https://bloodraven66.github.io/2020/08/27/The-Latest-In-1#cmc-paper">
Contrastive Multiview Coding[4]</a></b><br>
In this work, the authors learn a representation that aims to maximise mutual information between different views of the same image. Here, the views are image channels such as luminance, chrominance, depth, and optical flow.
Similarly as in CPC, two views are projected into a latent space. If they are positive pairs, then the MI should be maximised otherwise, for negative pairs it is minimised.<br>
![cmc]({{ '/assets/images/Selection_071.png' | relative_url }}){: class="center"}
<br>
The objective funtion tries to bring the two views closer. Cosine similairty between the two network predictions is calculated and negative log likelihood of these scores is computed which acts as the loss. One way to train this would be to take one view to be optimised over and take pairs with other views. A more, general formulation would be taking pairs of all the views. Then the objective function would be the summation of functions of all view pairs.<br>
A memory bank is used to retrieve negative samples for the positive samples whenever required. This helps in maving more negative examples though they won't be from the current weights so it might harm the performance. One key idea is that they want to maximise the good information between views and minise the bad information. So the idea behind CMC is that this can be a chieved by doing infomax learning on two views that share the signal but have independant noise. They also perform experiments to test this hypothesis.<br>



- <b><a href="https://bloodraven66.github.io/2020/08/27/The-Latest-In-1#moco">
Momentum Contrast for Unsupervised Visual Representation Learning[5]</a>
</b><br>
In this paper, the authors treat training encoder with contrastive learning as a dictionary look-up task. There are two networks. Netwoirk 1 is called  an encoder which takes a query as an input. Network two takes a mini batch and the outputs are enqueued into a dictionary. The dictionary size is larger than the mini batch so that a large number of negative samples can be drawn, inclusing old samples. Old batches are dequeued when dictionary size limit is reached.
<br>
![moco]({{ '/assets/images/Selection_072.png' | relative_url }}){: class="center"}
<br>
Basically, for every image going to the encoder, a large number of images are present to compare whether it is the same. Encoder is trained with backprop with contrastive loss. network 2 is a moving average of encoder. The same batch is fed to both the networks but with different data augmentations. Postive and negative pairs are formed and InfoNCE loss is used to distinguish between the type of pairs. The encoder is updated with the loss while the nother network is updated from encoder weights.
<br>

- <b><a href="https://bloodraven66.github.io/2020/08/27/The-Latest-In-1#simclr">
A Simple Framework for Contrastive Learning of Visual Representations[6]</a>
</b><br>
As the name suggests, this looks pretty simple and straightforward. Though, it needs specific functions, augmentations to bring out the best results.
<br>
![simclr]({{ '/assets/images/Selection_073.png' | relative_url }}){: class="center"}
<br>
For every minibatch, two sets of augmentation drawn and they are projected  using 2 networks. Then similairty between every pair of image is computed. Then negative log softmax of the pairwise similarity score across the batch is taken as the loss. Basically, for every pair in a N size batch, 2N - 1 projected data is considered as negative pairs. They show that CL works well with very large batches(4096), for which they use the LARS optimizer. They reveal that only few selected data augmentation give out good results, color augmentations being the important ones.



<br>
- <b><a href="https://bloodraven66.github.io/2020/08/27/The-Latest-In-1#byol">
 Bootstrap Your Own Latent: A New Approach to Self-Supervised Learning[7]</a>
</b><br>
In this work, negative examples are not used. This again has 2 networks to predict latent of 2 different augmented pairs. From a given representation called target, a different representation called online is trained by predicted the target. The target netowrk is an exponential moving average of the online network. By iteratingn the procedure, the authors' idea is to build a sequence of representations with increasing quality.
<br>
![byol]({{ '/assets/images/Selection_072.png' | relative_url }}){: class="center"}
<br>
 A mean squared error is used between the noramlised predictions of both networks. The augmentations are the same as the ones used in SimCLR.

<br>


So there were some of the recent papers in this area. With BYOL, the accuracy on imagenet stand at around 75-80% on linear evaluation protocol. As pointed out in the byol paper, one of the reason the research has moved towards contrastive learning is that the with cross view prediction based works leads to collapsed representations in the latent space. Contrastive learning removed this problem with he use of negative examples. However, using negative examples leads to high memory and computation costs. To have a large number of negative samples, many batch outputs have to be stored(as in moco paper). then the infonce loss has to be computed for all. BYOL achieves SOTA without the negative examples, they relay on predicting views. A lot of different contrastive loss functions have been in use. In the latent space, it is some kind of similaity function like cosine and then a temperature regulated normalised softmax. It is still a cross entrophy loss commonly used in supervised setup, as this is a discriminating method.<br>
A lot of progress has been made in the past few years and these works seem to be very similar in architectures. I think we will see some more progress in this area by the end of the year. I'm excited to see if anyone comes up with a different way to go ahead with this. One direction where I am very much interested in is be able to discriminate classes without labels as these works make use of labels for linear evaluation. We will see how it goes.
<br>
<br><b>References</b>:<br>
<br>[1] Raia Hadsell, Sumit Chopra, and Yann LeCun. Dimensionality reduction by learning an invariant mapping. In CVPR,
2006
<br>[2] Michael Gutmann and Aapo Hyvärinen. Noise-contrastive estimation: A new estimation principle for unnormalized statistical models. In Proceedings of the Thirteenth International Conference on Artificial Intelligence and Statistics, pages 297–304, 2010.<br>
<a id="cpc-paper">[3] arXiv:1807.03748</a><br>
<a id="cmc-paper">[4] arXiv:1906.05849</a><br>
<a id="moco">[5] arXiv:1911.05722</a><br>
<a id="simclr">[6] 	arXiv:2002.05709</a><br>
<a id="byol">[7] 	arXiv:2006.07733</a>
