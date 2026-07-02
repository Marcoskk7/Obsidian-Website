
1. CubeSandBox, agent 沙箱

- 项目架构图
![[Pasted image 20260701181207.png]]
- 核心优势
![[Pasted image 20260701181244.png]]

关键词解释:
1. 什么是 E2B:
	专为大模型代码设置的沙箱环境, 一套安全沙箱中间件标准
2. 什么是 CoW 技术, 为什么他能实现极致的内存调用
	Copy-on-Write, 写时复制, 当多个进程需要使用同一个数据时, 会让所有用户共享同一份数据, 并设置为只读
3. eBPF 是什么
	**eBPF** 全称是 **Extended Berkeley Packet Filter（扩展的伯克利包过滤器）**。在过去，如果你想改变 Linux 内核的网络、安全或监控行为，你必须去修改内核源码，或者编写高风险的内核模块（容易导致系统蓝屏/崩溃）。 而 eBPF 允许开发者在不修改内核源码、不重启系统的情况下，安全地在 Linux 内核中运行沙箱程序。


前置学习:
AI Agent的沙箱是什么？它和Docker容器/虚拟机有什么区别[https://www.bilibili.com/video/BV14sorBiEgP](https://www.bilibili.com/video/BV14sorBiEgP)

虚拟机, 是通过物理裸金属机器, 一个个虚拟划分的, 每个有独立的操作系统
