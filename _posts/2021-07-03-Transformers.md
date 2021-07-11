---
layout: post
title:  "Transformers"
date:   2021-07-03 10:03:29 +0300
categories: research
comments: true
---
Before I get into specific applications of transformers, let's look at the building blocks of transformers and some of it's inital works and directions it has taken.

<!--more-->
Transformer networks are turning out to be the leader across a multitude of problem statements in different input domains such as Natural Language Processing (NLP), Computer Vision (CV), Speech etc. It is also turning out to be excellent on modelling over multiple modalities.

<b>What is a transformer?</b><br>
Transformer is a combination of different representational blocks such as self-attention <b>(SA)</b>, position-wise feed forward <b>(PWFF)</b>, layer norm <b>(LN)</b>, positional encodings<b> (PE)</b> and residual connections. Many of these ideas were introduced in the years before this architecture was released and they were considered state of the art. Residual connections were widely popularised by the resnet paper - it was considered the gold standard in tasks like image classification. Different attention mechanisms were being developed and it worked as a steroid for sequence to sequence models.
In transformers, the authors found the right way to connect these blocks and it opened various new avenues in research.

The original transformer architecture was introduced for a NLP task - machine translation.
It involved sequences at input and output, with no rule on the length being the same. While older models such as LSTM look at the seqeuence positions one at a time, transformers looked at the entire sentence at once. This is performed by the self attention block. It obtains three learnable transformations - key, query and value from the input sequence and computes attention on the key and query as shown below.  
![attn1 alt ><]({{ '/assets/images/attn1.png'}}){:height="70px" width="350"}
Dot product is computed between the query and key followed by a softmax normalisation. This is multiplied with the value. These operations can be performed parallely where each self-attention operation is referred to as an attention-head and the entire self-attention is called multi-head attention <b>(MHA)</b>. In MHA, each attention head learns to focus on different relations within the sequence. The MHA is followed by PWFF. This is responsible for mixing the information learnt by the different attention heads. This operation is independent of the time-steps, i.e it does not mix information between sequence positions since it is explicitly extracted in SA. The architecture is shown below,
![attn2 alt ><]({{ '/assets/images/attn2.png'}}){:height="70px" width="180"}
Residual connections are present over the MHA and PWFF blocks and the combination of these two blocks are called a transformer layer. These layers are stacked sequencially <b>N</b> number of times (similar to how LSTM layers are utilised)

The decoder is built differently according to the task at hand. In the tasks where different sequence lengths are present, the output is usually provided as a conditioning to the decoder during training. This helps in learning the attention mechanism with another attention layer, with which the decoder can decode autoregressively on inference. To allow this, a causal mask is necessary on the decoder input attention since it should learn to predict when future time steps are absent. This is performed by masking the attention with a triangular mask and the MHA is usually called masked MHA. This is followed by a regular MHA wehre the encoder outputs and masked MHA outputs are combined.

A positional encoding is also important while using a transofrmer. This is because the positional information in the sequence is not remembered explicitly in the model. The authors use sine and cosine functions to represent the position. These are usually present at different frequencies to have a higher dimension representation.

Overall, all these components create a transformer model. It has a lot of benifits over recurrent models. Since it can look at the entire sequence, it easily enables learning long range dependencies. MHA allows lot of feature representation to be performed parallely, which speeds up the model. The main limitation in transformer is the O(n<sup>2</sup>) complexity in self attention. This hinders the modelling over longer sentences since it leads to extended GPU memory and also the increase in training time can be costly for large models.

Nevertheless, it has gained a lot of focus and has worked very well on a long list of tasks. It revolutionalised NLP with large scale models such as BERT, GPT etc. It has achieved optimal performance in various vision and speech tasks as well.

<b>What next?</b><br>
I will be touching upon specific research on transformers in the coming weeks. I intend to look at a few papers in areas such as
<li> transformers for vision </li>
<li> reducing time/memory complexity </li>
<li> different positional encodings  </li>
<li> architectures inspired by transformers  </li>
