---
title : Torch negative dimension
date : 2023-02-18 15:14:00 +0900
categories : [ML, Pytorch]
tags : [pytorch, AI, tools] #소문자만 가능
pinned : 0
---

```
torch.cat(x, -1)
```
What does the negative dimension mean here?
The "-1" essentially means the index among the tensor sizes, just like simple python.

For example, say we have a tensor with a size of (w * c * h * l).

<i> torch.cat(x, -1)</i> will concat the tensor along l (= torch.size()[-1]).

<i> torch.cat(x, -1)</i> can be rewritten as <i> torch.cat(x, 3)</i>.