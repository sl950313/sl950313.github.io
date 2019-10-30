---
layout: post
title:  "treelite验证报告"
date:   2019-10-30 13:00:00
categories: GitHub
tags: validation
---

* content
{:toc}

# treelite验证报告
----

## 1. treelite概览
---

[treelite 文档](https://treelite.readthedocs.io/en/latest/ "treelite document")

[source code](https://github.com/dmlc/treelite)

xgboost也是他们搞得，这个库应该是靠谱的。安装按照要求来就可以了。

## 2. 验证xgboost模型
---

参考自官方文档[部署过程](https://treelite.readthedocs.io/en/latest/tutorials/deploy.html)

### 2.1 构建xgboost模型
这里的模型是我们从python处理得到的xgboost模型。以此为输入，开始后面的c api使用过程。

### 2.2 构建库(.so)和相关头文件


    import treelite
    model = treelite.Model.load('your_model.model', 'xgboost')
    platform = 'unix'
	toolchain = 'gcc'
	model.export_srcpkg(platform=platform, toolchain=toolchain,
                    pkgpath='./mymodel.zip', libname='mymodel.so',
                    verbose=True)

生成的mymodel.zip包含了我们需要的库和头文件了。官方文档应该是老版本的了，文档里显示的zip里的和最新的release版本不一样，不过没关系，稍微改下就可以了。

解压缩，并编译得到动态库和相关头文件。

### 2.3 使用库
这里的矩阵输入使用的是CSR模式[wikipedia](https://en.wikipedia.org/wiki/Sparse_matrix)。文档写的已经是非常详细了。这里只需要按照他写的验证下输出是否是否一致就可以了。验证代码在145机器上的~/software/treelite/python/XGB_model下(这是一个回归模型)。结果和直接用python的xgboost包跑出来的结果一样，在输入为`[[1.0, 1.0, 0.0, 1.0, 1.0, 0.8]]`时的输出都为0.712207。在我的机器上运行时间平均为85us(`Intel(R) Core(TM) i5-7500 CPU @ 3.40GHz`).

部署的话只需要用上编译出来的一个.so以及.h就可以了。