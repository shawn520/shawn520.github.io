---
title: Latex报错Missing $ inserted
categories:
- 好好学习
tags: 
  - Latex
date: 2019-07-26 18:19:46
---

今天晚上在用Latex写论文，编译时报错：

```shell
Missing $ inserted.
<inserted text> 
```

## 解决方案

原来是论文中用到了下划线“_”，我用的是`sync_mysql()`方法名，但是在latex中下划线是有特殊含义的，所以如果想使用下划线可以使用以下两种方案

```latex
\_
% 或者这样
\underline{\space}
```

<!-- more -->



## 参考资料

https://blog.csdn.net/zhangpeterx/article/details/86032665