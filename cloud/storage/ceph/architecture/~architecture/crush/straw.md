# CRUSH 底层算法

## 简介

CRUSH 基本选择算法对比：

| 对比项\算法                    | unique | list | tree      | straw |
| ------------------------------ | ------ | ---- | --------- | ----- |
| 时间复杂度                     | O(1)   | O(N) | O(log(N)) | O(N)  |
| 添加元素产生的数据迁移量的评价 | 差     | 最好 | 好        | 最好  |
| 删除元素产生的数据迁移量的评价 | 差     | 差   | 好        | 最好  |

这里的 “元素” 可以是磁盘、主机，设置机架。

### straw 及 straw2 算法简介

容灾域：存储网络中的不同层级具有不同的灾难容忍程度

典型 Ceph 集群的层次结构

```c
+--------------------------------------------------------------------------------------------------------------------------------------+
|                                                              Data Center                                                             |
|    *----------------------------------------------------------------------------------------------------------------------------*    |
|    |                    Rack1                                   Rack2                                  Rack3                    |    |
|    |    +----------------------------------+    +----------------------------------+    +----------------------------------+    |    |
|    |    |  *----------------------------*  |    |  *----------------------------*  |    |  *----------------------------*  |    |    |
|    |    |  |            Host0           |  |    |  |            Host1           |  |    |  |            Host2           |  |    |    |
|    |    |  | +------+ +------+ +------+ |  |    |  | +------+ +------+ +------+ |  |    |  | +------+ +------+ +------+ |  |    |    |
|    |    |  | | OSD0 | | OSD1 | | OSD2 | |  |    |  | | OSD3 | | OSD4 | | OSD5 | |  |    |  | | OSD6 | | OSD7 | | OSD8 | |  |    |    |
|    |    |  | +------+ +------+ +------+ |  |    |  | +------+ +------+ +------+ |  |    |  | +------+ +------+ +------+ |  |    |    |
|    |    |  *----------------------------*  |    |  *----------------------------*  |    |  *----------------------------*  |    |    |
|    |    |                                  |    |                                  |    |                                  |    |    |
|    |    |  *----------------------------*  |    |  *----------------------------*  |    |  *----------------------------*  |    |    |
|    |    |  |            Host3           |  |    |  |            Host4           |  |    |  |            Host5           |  |    |    |
|    |    |  | +------+ +------+ +------+ |  |    |  | +------+ +------+ +------+ |  |    |  | +------+ +------+ +------+ |  |    |    |
|    |    |  | | OSD9 | | OSD10| | OSD11| |  |    |  | | OSD12| | OSD13| | OSD14| |  |    |  | | OSD15| | OSD16| | OSD17| |  |    |    |
|    |    |  | +------+ +------+ +------+ |  |    |  | +------+ +------+ +------+ |  |    |  | +------+ +------+ +------+ |  |    |    |
|    |    |  *----------------------------*  |    |  *----------------------------*  |    |  *----------------------------*  |    |    |
|    |    |                                  |    |                                  |    |                                  |    |    |
|    |    |  *----------------------------*  |    |  *----------------------------*  |    |  *----------------------------*  |    |    |
|    |    |  |            Host6           |  |    |  |            Host7           |  |    |  |            Host8           |  |    |    |
|    |    |  | +------+ +------+ +------+ |  |    |  | +------+ +------+ +------+ |  |    |  | +------+ +------+ +------+ |  |    |    |
|    |    |  | | OSD18| | OSD19| | OSD20| |  |    |  | | OSD21| | OSD22| | OSD23| |  |    |  | | OSD24| | OSD25| | OSD26| |  |    |    |
|    |    |  | +------+ +------+ +------+ |  |    |  | +------+ +------+ +------+ |  |    |  | +------+ +------+ +------+ |  |    |    |
|    |    |  *----------------------------*  |    |  *----------------------------*  |    |  *----------------------------*  |    |    |
|    |    +----------------------------------+    +----------------------------------+    +----------------------------------+    |    |
|    |                                                                                                                            |    |
|    *----------------------------------------------------------------------------------------------------------------------------*    |
|                                                                                                                                      |
+--------------------------------------------------------------------------------------------------------------------------------------+
```

* 单台主机包含多个磁盘
* 每个机架包含多个主机并采用独立的供电和网络交换系统
* 为了实现高可靠，实际上要求数据的多个副本分布在不同机架的主机磁盘之上


## straw

基本原理 —— 抽签（draw）过程：

1. 将所有元素比作吸管
2. 针对指定输入，为每个元素随机计算一个长度（签长）
3. 从中选择长度最长的那个元素（吸管）作为结果输出

如何计算签长：

执行结果取决于三个因素：

* 固定输入
* 元素编号
* 元素权重

伪代码：

```python
max_x = -1
max_item = -1
for each item:
    x = hash(input, r) # input: 固定输入；r: 随机因子
    x = x * item_straw # item_straw: 计算得到的签长
    if x > item_straw:
        max_x = x
        max_item = item
return max_item
```

item_straw 通过权重计算得到：

```c
reverse = rearrange all weights in reverse order
straw = -1
weight_diff_prev_total = 0
for each item:
    item_straw = straw * 0x10000
    weight_diff_prev = (reverse[current_item] - reverse[prev_item]) * items_remain
    weight_diff_prev_total += weight_diff_prev
    weight_diff_next = (reverse[next_item] - reverse[current_item]) * items_remain
    scala = weight_diff_prev_total / (weight_diff_prev_total + weight_diff_next)
    straw *= pow(1 / scale, 1 / items_remain)
```

## straw2

伪代码：

```python
max_x = -1
max_item = -1
for each item:
   x = hash(input, r)              # x = random value from 0..65535
   x = ln(x / 65536) / weight
   if x > max_x:
      max_x = x
      max_item = item
return max_item
```

## 参考

* [ceph 的crush算法 straw](http://www.zphj1987.com/2017/01/05/ceph-crush-straw/)