---
title: about
date: 2025-12-29 21:59:55
---

### 我是谁(Who am I)

---

我是周权，目前就读于中国科学技术大学软件学院，研究生一年级。对底层技术痴迷，喜爱研究 Linux 系统(Arch)，目前正在学习 C/C++ 语言。

Github: [Qizi706](https://github.com/Qizi706)

<br/>
<br/>

### 个人项目

---

#### MIT 6.s081 基于 RISC-V 指令集架构的操作系统内核

时间：2026年2月 - 2026年4月 个人项目

_C, Linux_

**项目描述**：本项目以基于RISC-V指令集架构的微型操作系统xv6 为基础，对其内存管理，进程管理，文件系统，中断处理等模块进行扩展与优化。

**主要工作：**

1. 实现 xv6 独立内核页表与用户页表映射机制，掌握 trampoline、trapframe 与页表切换流程，完善内核访问用户虚拟地址的数据拷贝路径
2. 基于 page fault 实现 lazy allocation 与 copy-on-write fork，维护物理页引用计数，减少 fork 时不必要的物理页复制
3. 扩展 trap 与 syscall 机制，实现 alarm 系统调用，支持用户态定时回调注册与恢复执行上下文
4. 实现非抢占式用户级线程库，支持线程创建、上下文保存/恢复与协作式调度，对比理解 xv6 内核进程调度与上下文切换机制
5. 扩展 xv6 文件系统，支持三级间接索引与软链接；实现 mmap/munmap，支持文件页按需映射与回写

#### CMU15-445 基于C++开发的支持简单 SQL 操作的单机数据库

时间：2025年11月 - 2025年12月 个人项目

_C++, Linux_

**项目描述：**这是一个面向磁盘的数据库管理系统（DBMS），名为 bustub。实现了缓冲池管理，B+ 树数据库索引，SQL 查询的操作执行器，多版本并发控制等。

**主要工作：**

1. 实现线程安全 BufferPoolManager，维护 page_id 到 frame_id 的映射、pin count、dirty flag 与页面换入换出流程
2. 实现 LRU-K 页面替换器，根据页面历史访问时间戳选择 victim frame，并与 Buffer Pool 的 evictable 状态协同工作
3. 实现支持并发访问的 B+Tree 索引，支持搜索、插入、删除、节点 split/merge/redistribution，以及基于叶节点链表的有序迭代器
4. 基于 Volcano Iterator 模型实现查询执行器，统一 Init/Next 接口，支持 Scan、Join、Aggregation、Sort、Limit、Distinct 及增删改算子
5. 实现规则优化器，将索引列等值谓词下的 SeqScan 改写为 IndexScan，并将等值连接条件下的 NestedLoopJoin 改写为 HashJoin
6. 实现基于 MVCC 的事务模块，通过版本链和 undo log 支持快照隔离下的并发读写
