---
date: 2025-08-27 16:14:01
tags: dailynote
rating: ⭐
excerpt: 
---

## 发给豆包的prompt
```


```


## 第一步:deep research找instruction
```
我现在在用 instruction–answer–concept 框架去测试模型的 deep research 能力（在测试时我不会上传这篇论文）。 

任务描述：
我将提供一篇Thesis（硕博学位论文，题源论文TR25-058）。请你作为一位深耕领域前沿的学者，找出Thesis中解决的instruction（关键问题），该instruction能够在Thesis中找到一个结构简单的answer（答案），再找出解决instruction得到answer所需要的3-5条Concept知识点，并重点检查以下方面的问题： 
---
instruction 应该是一个论文中确实解决了的、但能用一个非常简单的量或结论来回答的问题。不能过于开放。
---
请检查answer的结构是否符合结构简单的要求，answer的形式应该是是标量、产物名、简短的结论等。
---
请检查每一条Concept是否出现在Thesis中，并且是解决instruction所必须的。
每一条Concept中只包含一个概念或定理。
请保证每一个concept都要有一个非本论文的参考文献（给出ref在Thesis中的具体参考文献编号，在原文中是如何使用的），如果你的这个结论没有其他论文提及的话，你可以把他放进instruction里做已知信息。
---
我希望你可以聚焦于整篇论文的核心成果，如果核心成果设计的instruction的answer结构过于复杂，你也可以先关注某个章节或具体技术的成功。 

----

输出格式建议：
请按照以下结构回答：
Instruction：（领域前沿的问题）
Answer：（结构简单的答案）
Concept/Theorem：（解决instruction得到answer所需要的知识点）
- concept_n:...
- theorem_n:...
（以此类推）
---
我想跟你讲解一下我现在要做的事情：我在测试模型是否能自己推出正确答案。我需要提交的字段【图片】已经上传给你了，但你不需要一下子给我所有字段的回答。现在我们一步一步来。首先，我需要找到一个合适的instruction，这个instruction中不能包含只存在于题源论文（也就是我上传给你的A zero-knowledge PCP THEOREM....这篇论文）中的特有的知识。我会把这个instruction发送给豆包，豆包将无法计算出正确的答案。
第二，我需要你给我整理几条concepts。检查每一条Concept是否出现在题源论文中，并且是解决instruction所必须。并且，每一条concepts出现在题源论文中时，需要是通过引用其他参考文献得到的concepts，而不能是该题源论文中独特的知识/推导中间过程。如果它是特有的知识，那么你需要把它放到instruction中，你明白吗？每一个Concept中只包含一个概念或定理。 其次，我希望concept中不要包含关键推导，而只是给出一些构造性证明的关键构造。我把【concepts+instruction】都发送给豆包，我希望豆包模型有时能推导出正确答案，有时不能推导出正确答案。
第三，我需要你再次检查你给出的每一条concepts是否能在题源论文中找到参考文献。我到时会把这些参考文献下载下来发送给你，你需要为我找到每一条concepts出自该参考文献的位置。

以上三部分你是否理解了？如果你理解了，你可以先为我进行第一步。
```


```
请检查：instruction的内容中，是否存在只存在于我上传的这篇论文中特有的内容？
我在对豆包进行训练的时候，不会上传题源论文。
```

```
目前豆包模型无法通过concepts推导出instruction的正确答案，请你重新设计concepts。
我认为concept需要更具体，例如具体的做法，计算公式等。

但你需要注意：
1.每一条Concept/Theorem是否出现在论文中，并且是解决instruction所必须。
2.每一条Concept/Theorem中只包含一个概念或定理。
3.注意！Concept/Theorem不能是题源论文推论，需要由外部的参考文献引入。 concepts优先在当前题源论文的参考文献中找，如果无法在参考文献中找到，可以在互联网上检索。
请根据我的建议，修改你的设计。

```
