---
title: TC275CPU子系统
date: 2020-02-18 13:33:58
tags: [tricore, TC275]
categories: tricore
toc: true
---
## TC275CPU配置
TC275有三个CPU，它们的配置如下表：
![Processor and local memory configuration](Processor_and_local memory_configuration.png)
## CPU实现的特定功能

### 上下文存储CSAContext Save Areas
CPU对函数调用，中断和陷阱使用统一的上下文切换方法。 在所有情况下，任务的上级上下文都会由硬件自动保存和恢复。 下级上下文的保存和还原可以选择由软件执行。
TC1.6E的CSA空间放在DSPR或者扩展内存。
TC1.6P的CSA空间放在DSPR中。
### 中断
