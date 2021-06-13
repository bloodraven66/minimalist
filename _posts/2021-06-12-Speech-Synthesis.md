---
layout: post
title:  "Speech Synthesis"
date:   2021-06-12 01:03:29 +0530
categories: research
comments: true
---
This is an overview of recent progress in Text to Speech models.

<!--more-->

In PROGRESS

There has been decades of research in speech synthesis. Speech synthesis programs have been available in computers since the 1990s. Nowadays, it is easy to access services which can provide TTS freely. You don't need to look further than your smartphones. This doesn't mean Speech Synthesis has been solved. There are still a lot of challenges assosiated with the technology. Some of these are as follows:
<ul>
  <li><b>Large amount of training data:</b>

  TTS algorithms require a large amount of clean training data in order to perform well. While this is available in english and other commonly used langauges, it can be hard to build good TTS machines on every langauge.</li>
  <li><b>Adapting to different speakers:</b>

  Many of the TTS algorithms in use today is trained on single speaker dataset. Transfering to a different voice with changes in speaking style, accent etc is non-trivial.</li>
  <li><b>Natural and realistic speech generation</b>:

  TTS algorithms are not trained on audio directly. It is much easier to learn towards a representation such as spectograms, after which it can be reconstructed using vocoders. It is still an active research problem on building good vocoders which can generate realistic speech. </li>
  <li><b>Duraion modelling</b>

  A key challenge assosiated with speech synthesis is that the sequence lengths for input text and output speech are going to be much different. These are going to change for different sentences uttered, with a lot of variations on pitch, pronunciation, pace, etc. While there are supervised and unsupervised methods available to deal with it, it has a trade off w.r.t computational complexity, on each of the methods.
  </li>
</ul>
 These challenges are in the core formulation of synthesis tasks. We shall cover these in detail while going through some of the recent papers.

The research direction has changed over the years and is currently adopting the widely used transformer architectures. It is natural that the machine learning research communitiy in general is shifiting towards transformers. It has achieved tremendous success in Natural Language Processing (NLP) with development of large scale systems such as BERT, GPT. It offers a lot of advantages which is suited for sequence tasks such as synthesis. This however, is not a review on transformers, it's a topic for another day. For now, let's start with an architecture which came out in 2017 and still used for comparision, which isn't really the norm in current research involving deep learning .

<b>Tacotron:</b><br>
The tacotron [1] simplified the steps involved in the speech synthesis task and in many ways, lowered the bar on expertise required towards building such systems. In many of the older works, it was necessary to design different blocks for linguistic features, acoustic models, duration model, complex signal processing based vocoders etc. A lot of domain knowledge was required in building the models. Tacotron enabled the neural network architecture to handle a lot of these components and making it end-to-end trainable. <br>
![taco]({{ '/assets/images/taco.png' | relative_url }})
<br>
The encoder consists of a charecter embedding followed by few dense layers referred to as encoder pre-net. The output is passed through a CBGH module. CBGH consits of 1D convolution layers followed by a GRU. This is passed to a autoregressive decoder with tanh attention. This attention mechanism is responsible for alignment between the input and output sequence. GRU with vertical connections are used, where the input to each time step is the attention context vector, previous time step output and hidden units. These outputs are then passed to another CBGH module., The Mel Spectogram is choosen as a target.  The authors modified the autoregressive framework such that <i>r</i> non-overlapping frames can be predicted instead of 1 a time. A lot of benifits were observed with it towards faster, stable training with lesser parametrs. The audio is reconstructed using Griffin Lim reconstruction algorithm.<br>

Tacotron has become a standard architecture which researchers look at improving on. It has a lot of contributions to it. It provides a way to perform unsupervised duration modelling with attention. It's choice of Mel spectogram as target is now commonly used in all synthesis research. Even many of the parameters used in it's implementation is used as the standard and applied on other works without any modifications.

<b>Tacotron2:</b><br>
This work [2] is an improvement on the Tacotron which come out in the same year as tacotron. The model structure is same that as Tacotron. It uses LSTM layers instead of GRU and local sentive attention in the decoder. It also incorporates a convolutional post-net layer with residual connection instead if CBHG layers used in Tacotron. Overall, I would say different modules of Tacotron were replaced by similar alternatives and experimented on, the best perfoming layers were selected for this model.

![taco2]({{ '/assets/images/taco2.png' | relative_url }})
<br>

This work also uses wavenet as a vocoder instead of Griffin Lim reconstruction. Griffin Lim is known to generate audio with lower quality, it has it's charecteristic metallic sound to it.

Now, let us look at transformer architectures applied for this task.

<b>Transformer TTS:</b><br>
This work [3] looks at the resemblence of a view of Tacotron2 setup with original Transformer architecture. Both consists of encoder and decoder, with an attention component combining information from encoder to decoder states. With this formulation, they diretly utilise transformers for speech synthesis task with mel spectogram as target.

![ttts]({{ '/assets/images/ttts.png' | relative_url }})

Similar to tacotron, an embedding is used for input. This is passed to transformer encoder. At the same time, spectogram is passed to the decoder, which consists of dense layers followed by masked self attention layer. A multi head attention of transformer replaces the local senstive attention from tacotron2. The encoder and decoder states are combined in this attention. This is followed by Post net and stop token layers as utilised in tacotron2. A different version of positional encoding was used which allowed for training a scale parameter for the encodings.

Key advantage is the usage of transformers and the benifits it brings in terms of learning better and faster representation.

<b>FastSpeech:</b> <br>
FastSpeech [4] simplifies the architecture by using encoder architecture for the decoder as well. There is no Mel spectogram conditioning involved at the decoder. The duration modelling is done by an expliit duration predictor consisting of few 1D convolution layers. This makes the encoder and decoder fully parallel at all times on the sequence.

![fs]({{ '/assets/images/fs.png' | relative_url }})<br>
The model predicts duration for each phoneme present in the input. The encoder representation for each phoneme is then replicated to match the number of frames for decoder, as predicted by the durations.
This has insipred a lot of similar architectures with small modifications.



<b>FastPitch:</b><br>
This work [5] starts with the FastSpeech architecture and adds an additional component of predicting pitch at the encoder representation.

<b>FastSpeech2:</b><br>
FastSpeech2 [6] incororates duration, pitch and energy prediction in the encoder representation. It also has an additional component called fastspeech2s. This module can perform end to end waveform gegenration with the TTS module.

![fs2]({{ '/assets/images/fs2.png' | relative_url }}) <br>
It consists of non causal convolutionals and gated activation as used in wavenet. It takes slices of decoder hidden and predicts the time steps required. The mel spectogram decoder is discared on inference. It uses adversarial training by having a discriminator based on Parallel Wavegan.
The key contribution od this work is being able to train waveform generation by the same model. More work will be inspired by this for sure!

<b>Hierarchical Prosody Modeling for Non-Autoregressive Speech Synthesis:</b><br>
Again, this work [7] builds on the fastspeech architetcure, with a focus on incorporating prosody information. Prosody is the variation in speech signals such as rhythm, intonation, stress etc. At the encoder representation, it has an additional component which predicts prosody.

![prosody]({{ '/assets/images/prosody.png' | relative_url }}) <br>

The ground truth prosody is extracted by rule based as well as Conv 1D layers. These are predicted by convolutional layers and all the predictions are concatenated and used to contruct decoder sequence length.

<b>Conclusions:</b><br>
I've covered only a few work done in this field. There is much more work done in recent years towards models like DeepVoice, Flowtron etc which I will cover in a different article. Taking a look at the papers mentioned here, tacotron can be regarded as a monumental shift in the research direction. A lot of work is coming up with FastSpeech style models with focus on conditioning additional information into the model to be able to create realistic audio. It looks like transformers are here to stay in TTS research, looking forward to see how different challenges are addressed in future works.

Note: All architecture images are taken from their respective papers available on arxvi.org

References:<br>
[1] https://arxiv.org/abs/1703.10135 <br>
[2] https://arxiv.org/abs/1712.05884 <br>
[3] https://arxiv.org/abs/1809.08895 <br>
[4] https://arxiv.org/abs/1905.09263 <br>
[5] https://arxiv.org/abs/2006.06873 <br>
[6] https://arxiv.org/abs/2006.04558 <br>
[7] https://arxiv.org/abs/2011.06465 <br>
