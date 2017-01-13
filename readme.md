This implements a sigmoid gated convolutional network, as per https://arxiv.org/pdf/1612.08083v1.pdf.

![x](https://raw.githubusercontent.com/astanway/gated-conv-nets/master/arch.png)

## To run
`python model.py --train True`

## Discussion:
  The technique described in this paper is an attempt to set up a convolutional network to achieve the same sorts of contextual inputs that an LSTM or RNN is traditionally good at, while taking advantage of the CNNs non-temporal nature to effect big speed gains. 
  
  For the language modeling task, this means that the inputs are a sequence of learned word embeddings, and the outputs are that same sequence, but shifted to the left. The final output embedding for a word (a vector within the final hidden layer) is thus trying to predict the word in front of it, a probability which is calculated with the softmax for each vector in the final hidden layer.

  In this way, all output projections are computed simultaneously through the convolutional layers. Care must be taken to properly pad the inputs such that each convolution kernel cannot see any context in front of it, as this would constitute data leakage.
  
  We can see that "context" for each word in this scenario is limited to the receptive field of the last layer, which is the width of the kernel - 1. On the surface, this would appear to give this type model less capacity than a recurrent architecture, which theoretically has unlimited context. However, this makes this model analogous to n-gram based models, where the n is defined by the kernel width. In traditional n-gram based models, it is rare to have n larger than typical kernel widths for CNNs (2-5), which would appear to mean that one could use a gated CNN as a stand-in substitute for any n-gram model.

  As with most language modeling tasks, the most expensive part of the computation is the softmax stage, where each output vector must calculate the probability with respect to the target word. This is expensive because all of the output probabilities must sum to one, so the total across all possible tokens in the vocabulary must be computed as the divisor for normalization. 
  
  The authors use a newer technique called the adaptive softmax to approximate the softmax for speed. I have opted for Tensorflow's native implementation of sampled softmax for now until I have the time to read the adaptive softmax paper [0] and understand how it works. The sampled softmax works by simply approximating the total by sampling over a handful of classes in the vocabulary. This tends to work because most vocabularies tend have a Zipfian distribution of word frequencies.
  
  In this implementation, I use a depthwise 2d convolution, treating each input embedding dimension as a different channel. As per the paper, I progressively increase the dimensionality and context size as the layers increase. This particular net is 8 layers deep, with 2 residual layers, and an output embedding projection of 1024 dimensions. The sequence length is set at 20. I use weight normalization [1], as per the paper. The batch size is set to 750.
  
  Note: In the original paper, the initial sequence padding is listed to be k/2 (k= kernel width). It should in fact be k-1, as k/2 would allow future context to leak into the current word. I have verified this with the authors and the error will be corrected in the next version of the paper.
 
 
TODO:
- Implement adaptive softmax

[0] https://arxiv.org/pdf/1609.04309v2.pdf

[1] https://arxiv.org/pdf/1602.07868v3.pdf
