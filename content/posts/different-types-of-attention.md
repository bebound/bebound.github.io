+++
title = "Different types of Attention"
author = ["KK"]
date = 2019-07-15T00:16:00+08:00
lastmod = 2023-12-18T21:38:37+08:00
tags = ["Machine Learning"]
draft = false
noauthor = true
nocomment = true
nodate = true
nopaging = true
noread = true
+++

\\(s\_t\\) and \\(h\_i\\) are source hidden states and target hidden state, the shape is `(n,1)`. \\(c\_t\\) is the final context vector, and \\(\alpha\_{t,s}\\) is alignment score.

\\[\begin{aligned}
c\_t&=\sum\_{i=1}^n \alpha\_{t,s}h\_i \\\\
\alpha\_{t,s}&= \frac{\exp(score(s\_t,h\_i))}{\sum\_{i=1}^n \exp(score(s\_t,h\_i))}
\end{aligned}\\]


## Global(Soft) VS Local(Hard) {#global--soft--vs-local--hard}

Global Attention takes all source hidden states into account, and local attention only use part of the source hidden states.


## Content-based VS Location-based {#content-based-vs-location-based}

Content-based Attention uses both source hidden states and target hidden states, but location-based attention only use source hidden states.

Here are several popular attention mechanisms:


#### Dot-Product {#dot-product}

\\[score(s\_t,h\_i)=s\_t^Th\_i\\]


#### Scaled Dot-Product {#scaled-dot-product}

\\[score(s\_t,h\_i)=\frac{s\_t^Th\_i}{\sqrt{n}}\\]
where n is the vectors dimension. Google's Transformer model has similar scaling factor when calculate self-attention: \\(score=\frac{KQ^T}{\sqrt{n}}\\)


#### Location-Base {#location-base}

\\[socre(s\_t,h\_i)=softmax(W\_as\_t)\\]


#### General {#general}

\\[score(s\_t,h\_i)=s\_t^TW\_ah\_i\\]

\\(Wa\\)'s shape is `(n,n)`


#### Concat {#concat}

\\[score(s\_t,h\_i)=v\_a^Ttanh(W\_a[s\_t,h\_i])\\]

\\(v\_a\\)'s shape is `(x,1)`, and \\(Wa\\) 's shape is `(x,x)`. This is similar to a neural network with one hidden layer.

When I doing a slot filling project, I compare these mechanisms. **Concat** attention produce the best result.


## Ref {#ref}

1.  [Attention Variants](http://cnyah.com/2017/08/01/attention-variants/)
2.  [Attention? Attention!](https://lilianweng.github.io/lil-log/2018/06/24/attention-attention.html)
3.  [Attention Seq2Seq with PyTorch: learning to invert a sequence](https://towardsdatascience.com/attention-seq2seq-with-pytorch-learning-to-invert-a-sequence-34faf4133e53)
