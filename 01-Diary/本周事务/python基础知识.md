---
date: 2025-05-12 22:47:15
status: doing
excerpt: 为啥要做？做完后有何收获感想体会？
tags: 
alias: 
rating: ⭐
---
def __getitem__(self, item):  
    image, label = self.dataset[self.idxs[item]]  
    return torch.tensor(image), torch.tensor(label)

魔方函数：