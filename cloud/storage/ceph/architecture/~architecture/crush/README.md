# 第一章 计算为王 —— 基于可控制哈希的受控副本分布策略 CRUSH

大型分布式存储系统需要解决的问题：

### 数据再均衡问题

    * 当新设备加入后，数据会随机地从不同的老设备迁移过来
    * 当老设备因为故障退出后，其原有数据会随机迁出到其他正常设备

目的：

* 使整个系统一直处于动态平衡过程之中，使之能适应任何形式的负载和拓扑结构变化

解决方案：

* 将数据以足够小的粒度打散，然后完全随机的分布到所有存储设备之间

好处：

* 因为任何数据（文件、块设备等）都被打散成为多个碎片然后存储到不同的存储设备，因此可以获得更高的 I/O 并发和汇聚带宽

CRUSH 算法在传统哈希算法的基础上解决了两个问题：

* 如果系统的存储设备数量发生变化，应当最小化数据迁移量从而使系统尽快恢复平衡
* 大型（PB 级以上）分布式存储系统中，数据包含多个备份，需要合理分布这些备份从而尽可能地使数据具有更高的可靠性

CRUSH 算法的输入：

* 数据唯一标识符
* 集群拓扑结构
* 数据备份策略

CRUSH 算法的优势：

* 客户端和服务端都可以通过计算获得数据所在的底层存储设备位置并直接与其通信，从而避免查表操作，实现了去中心化和高并发
* 支持多种数据备份策略（镜像、RAid、纠删码），受控地将数据的多个备份映射到集群不同区域中的底层存储设置之上，从而保证数据可靠性

## 简介

## 目录

* CRUSH 算法 —— straw & straw2
* CRUSH 原理/详解
  * Cluster Map
  * CRUSH Rule
  * Device Classes
* CRUSH 调制

## 优劣

优势：

* 具有计算寻址、高并发、动态数据均衡和可定制的副本策略等基本特性
* 能方便实现诸如去中心化、有效低于物理结构变化并保证性能随集群规模呈线性扩展、高可靠等高级特性

劣势：

* 基于计算寻址的设计，导致当需要对 CRUSH 升级时必须要求客户端和服务端同时进行，这将对内核客户端（krbd、kcephfs）类型的用户造成影响

## 参考

* [大话Ceph - CRUSH那点事儿](http://www.xuxiaopang.com/2016/11/08/easy-ceph-CRUSH/)
