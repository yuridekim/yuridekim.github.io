---
title : Torch adding custom loss functions
date : 2023-04-30 00:20:00 +0900
categories : [ML, Pytorch]
tags : [ML, tools, pytorch] #소문자만 가능
pinned : 0
---

# Adding custom Pytorch loss functions
Even with the most "commonly" used Pytorch loss functions - such as the MSE or L1 loss, there are times where we feel the need to tweak the functions, or even come up with a novel loss function to give feedback to our model.

## How to define a loss function?
First, you will want to add a class to implement the function. This, will take in `nn.Module` as a parameter.

```python
class CustomLoss(nn.Module):
    def __init__(self):
        super(CustomLoss, self).__init__()

    def forward(self, preds, targets):
        # some kind of loss function that involves preds and targets
```

Like any other layers of the model, add the implementation into the forwarding of the loss. Simple and easy up to this stage.

### Attach gradients
However the tricky part is that when you define a loss function, you want to make sure that all the calculations attach <i>gradients</i> with it by using `requires_grad_()`. This begins recording the calculations between the tensors.

```python
def forward(self, preds, targets):
    preds = torch.exp(preds)
    targets = torch.exp(targets)
        
    divided_calc = torch.div(preds, targets).requires_grad_()
```

Without the `requires_grad_()` itself, the autograd graphs that connect the layers by which the operation gradients transfer, will be disconnected. 

### Pitfalls
Common pitfalls not only include dividing but concatenating and max caculations.

```python
cat_array = torch.cat((array_first, array_second), 1).requires_grad_()
maxed_array = cat_array.max(1)
q_error = torch.sub(maxed_array.values, 1) # make sure to use values

```

- Concatentation: The concatention itself involves creating a new tensor, therefore the gradients should be specified for them to flow.

- Max operations: When torch max operations are put through, the index of the max values are attached as the results along with the actual values. Therefore, for the operation to be differentiable, the <b>values</b> of the max operations should be fed into the next steps. This is one of the most common mistakes during defining manual loss functions.

### Turning off autograd
However there may be some situations where you do want to operate the gradients but not want it to be included in the feedback loop of the model.

For example, you would want the model to use the mean squared error as the loss function but want to keep track of the L1 error for experimental purposes.

In this case, there are two options.

1. Detach from pytorch
By detaching the tensors from pytorch and rendering it into a numpy array, the gradients won't travel back to the layers.
```python
pred.cpu().detach().numpy()
```

2. torch.no_grad()
You can explicitly wrap the loss operations inside the no_grad which specifies that the gradients will not be calculated for the model feedback loop.

```python
with torch.no_grad():
    calculated_loss += loss_function(target, pred).item()
```