# MIT 6.824 Distributed System

----

# Lecture 1: Introduction

## Why we build distributed system?

- parallelism(并行性)：多台系统多个CPU并行工作

- fault tolerance：多台机器做同样的工作，当某台机器出错时，有其他机器可以成功完成该工作

- physic：物理上需要机器分布在不同的地方，例如相同的银行业务需要在多地展开

- security/isolated：可能有问题的/不信任的程序单独运行在一个地方，而可信任的安全的部分运行在另一个地方，实现隔离

其中，最重要的是前两点。



## Distributed System is hard

分布式系统总是很复杂的，如果一个工作放在一台机器上就能够完成，那就放在一台机器上，不要使用分布式系统。例如在一台机器上运行排序算法很简单，但是要放在大量的机器上共同为一个大的数据集排序，就会让工作变得很复杂。So, don't consider distributed system unless necessary.

- concurrency：需要考虑多台机器多个CPU的协同工作

- partial failure：机器多了导致系统出错时难以准确定位和处理（机器出错or系统出错or网络出错）

- performance：建立分布式系统的目的就是增加机器的数目以提高解决问题的速度，而当使用1000台机器的时候如何才能让工作效率提高1000倍，需要良好的设计


