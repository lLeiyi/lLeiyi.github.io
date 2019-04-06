---
title: ArcEngine中featurecursor分别在sde数据库和gdb数据库下的遍历问题
tags:
- ArcEngine
- IFeaturecursor
categories:
- ArcEngine
- 记录&&踩过的坑
---

## 问题描述
```C#{.line-numbers}
 //一般的循环遍历步骤
 IFeatureCursor featureCursor = featureClass.Search(null,false);
 IFeature feature = null;
 while((feature = featureCursor.NextFeature())!=null)
 {
 }
```
 以上是常用的循环遍历的步骤，最后遍历到feature为null后退出while循环，这时如果再执行一次featureCursor.NextFeature()语句：
 1.在gdb和mdb数据库下会得到null；
 2.在sde数据库下会得到“调用的函数顺序有误 [Function called out of sequence]”的报错。