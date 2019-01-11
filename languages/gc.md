# 垃圾回收

## 定义

垃圾回收（Garbage Collection，缩写：`GC`），是一种自动的存储器管理机制。当计算机上的动态存储器不再需要时，就应该予以释放，让出存储器。

垃圾回收器可以让程序员减轻许多负担，也减少了程序员犯错的机会。

垃圾回收起源于 LISP 语言，目前大多数语言如 JAVA、GoLang、Python 都支持垃圾回收.

## 特征

垃圾回收有两个基本原理：

1. 考虑某个对象在未来的程序运行中，将不会被访问
2. 向这些对象要求归回存储器

## 分类