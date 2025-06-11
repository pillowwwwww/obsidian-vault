---
Title: python基本语法
tags:
  - python基本语法
原始链接:
---

### Python中的for循环: for+ 循环变量+ in +可迭代对象:
range是Python中的一个内置函数, 起到的效果就是得到一个"可迭代对象", 这个可迭代对象中就包含了一系列的整数
range(beg, end), 从beg开始到end结束, 左闭右开
range(beg, end, 步长), 步长默认是1, 步长为每两个数的间隔

### 执行顺序
Python作为解释型语言不需要预编译，从模块顶部开始执行，而Java必须从main()方法开始。Python中的"if __name__ == '__main__'":用于标识程序入口，但不同于Java的强制性main()。在导入模块时，只有被运行的模块的"__name__"才会 https://cs.fmy1024.cn/html_online/2503_119388716online.html