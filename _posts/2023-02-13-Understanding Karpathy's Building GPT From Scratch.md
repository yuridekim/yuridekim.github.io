---
title : Understanding Karpathy's Building GPT From Scratch
date : 2023-02-13 02:07:00 +0900
categories : [ML, NLP]
tags : [NLP, GPT, AI, transformers, attention, TDD] #소문자만 가능
pinned : 1
image:
    path: /assets/img/posts/skip_connection_1.png
---

## Introduction
With all the hype surrounding ChatGPT, let's try to understand and wrap our head around how the GPT model essentially works.

And for this, we have this amazing resource that Karpathy he himself provides: 

[Let's build GPT: from scratch, in code, spelled out.](https://www.youtube.com/watch?v=kCc8FmEb1nY)
<iframe width="560" height="350" src="https://www.youtube.com/embed/kCc8FmEb1nY" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
Before we dive into this, it is recommendable to read [Attention is All You Need](https://arxiv.org/abs/1706.03762)

## Terminology and concepts

### Block Size

When we train the transformer, we train random little chunks at a time and the maximum length here will be called <b> block size</b>(a.k.a context length). When we think of language, the length increases with time so this can also be thought of as the <i>time dimension</i>.

For example when we have 9 elements in a list, we have 8 examples of dependencies that can be used to predict the next input.

```
tensor([18, 47, 56, 57, 58, 1, 15, 47, 58])
```

![block size](/assets/img/posts/block_size.png)

This will make it more efficient for us as we can start sampling generation later on with context as little as of one word. After block size, we will need to truncate for transformers can not deal large sizes of data.

### Batch Size
On another note, we also care about batch dimensions. When we have multiple patches of sequences, how are we going to parallelize these?
For the training efficiency of GPUs we stack them to a single tensor but in the training phase, it does not affect each other.

![Batch Stacked](/assets/img/posts/stack_batch.png)

The tensors for our inputs and targets are seperately defined. You can see that both have the shape of batch size * block size - 4 * 8 in our case. This way for each batch we can generate sequences of 8 blocks and the 4 independent batches are sent to traverse through our model.

## Bigram Model
The model will refer to the embedding table to find the corresponding row to the given index(word).

This embedding table will take the shape of (B, T, C) which denotes B: batch size, T: time(block size) and C: channels, meaning vocab size.

However the results from the bigram model won't be stellar. It's a simple one, so what do you expect? There's definitely room to improve.

### Past context
Now we want the tokens in a single batch to interact with each other but in a unidirectional way, from the left to the right. So we take the average of the preceding components.

![weights](/assets/img/posts/past_weights.png)

The weights for the embedding matrix in the bigram model is (T, T) while the input will be (B, T, C). As the dimensions do not match for matrix multiplication, torch will add a batch dimension at the top hence (B, T, T) @ (B, T , C) = (B, T, C)

Tril will take the lower triangle of the matrix which is important for us because we only take information into account that travels to the right.

![weights](/assets/img/posts/weight_softmax.png)

Now we will also run the weights through the softmax. If you think about the nature of softmax functions, the negative infinity will mask the upper triangle as the exponential in the softmax will return values to 0.

### Position Embeddings
We also encode the position embeddings alongside the token embeddings. This will become useful for self-attention heads.

## Self-attention
Now we include the data context into our weights using self-attention. Every single token will emit a query and a key.

<i><b>Query: What am I looking for?</b></i>

<i><b>Key: What do I contain?</b></i>

And the dot product of these two elements become weights. If the two align, it will result in a high value.
The queries here are still indepedent of each other but the dot product between the queries and keys will render weights to be data-dependent.

![Self-attention](/assets/img/posts/self_attention.png)

Now we introduce a new variable, "value" which is just x propagated through a linear layer. The output will be the dotproduct of weights and values.

So the queries will specify what they are looking for: which position it is in, if it's looking for a vowel after a consonant or not. And the <i>key</i> will tell you the answers to it. These two combined, will tell you the affinity of these interactions.
Now the value will take these weights and be added into the output. <i>"Assess how important you think I am and include me!"</i>

### Notes
- Self-attention is a communication mechanism and can be applied to any arbitrary directional graph.
- There is no notion of space, so you need positional encodings. Meanwhile, CNNs do provide an idea of positions.
- All examples in the batch dimension are independent.
- If you are doing sentiment analysis, you might do away with tril and let all the nodes communicate with each other.
- Cross-attention: Keys, queries and values don't need to come from the same source. Keys and values can be from seperate sources that we would like to pool attention from.
- Since weights are fed into softamx, it's important for them to be fairly diffused - that is with fixed variance. If the values are very extreme positive & negative values, softmax will converge to one-hot vectors. We can implement this by dividing with the root of head size.

## Multi-headed Self-attention
Applying multi heads and concatenate them, similar to group convolutions.
After the attention head, the model needs time to process the data it was given. This is implemented through feed forward which consists of a linear layer and relu.
This happens per node and therefore is independent. 

Now we need to intersperse the communication with the computation. So it groups them and communicates them.
To make things work channel-wise, we set head size to the number of embeddings divided by number of heads. 

### Skip Connections
To maintain the connections whilst depth, we use skip connections. The residual pathway can be forked away from. While backpropagation, it travels through back equally for addittions, so the gradients will make it all the way back from the supervision to the input, unimpeded.

In the beginning, it's almost as if it's not there but with optimization, they came online overtime and start to contribute.

![Skip Connection 1](/assets/img/posts/skip_connection_1.png)

![Skip Connection 2](/assets/img/posts/skip_connection_2.png)

### LayerNorm
Normalizes the rows instead of the columns. They will have unit Gaussian.

![Layer Norm](/assets/img/posts/layer_norm.png)

### Dropouts 
Every forward/backward path, it shuts off some subset of neurons, randomly drops them to 0 and train without them, so it trains an ensemble of subnetworks. If we scale up the model, it will help prevent overfitting.
