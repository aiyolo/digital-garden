---
{"dg-publish":true,"permalink":"/pages/py-torch/","title":"PyTorch 打印模型的输入输出","tags":[null]}
---


# PyTorch 打印模型的输入输出

```python
import torch
from torch import nn
from torchvision import models

model = models.alexnet(pretrained=True)

def register_hook(model):
 def hook_fn_forward(module, input, output):
 # input:tuple
 print("{}\ninput:{}, output:{}\n".format(module, input[0].size(), output.size()))
 if len(list(model.children()))==0:
        model.register_forward_hook(hook_fn_forward)
 else: 
 for name, module in model.named_children():
            register_myhook(module)

register_hook(model)


x = torch.randn(1,3,224,224)
o = model(x)
```
