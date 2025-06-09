---
name: virtual-machine
title: Virtual Machine Monitor/Hype
date: 2025-01-20
categories:
- OS
---



# Hypervisor (VMM)

Hypervisor 是一种运行在基础硬件和操作系统之间的中间软件层，可允许多个操作系统和应用共享硬件。也可叫做虚拟机监视器，即 VMM（ virtual machine monitor ）。（操作系统的操作系统）

动机：

1. 提高硬件的利用率，一套硬件跑多个操作系统
2. 方便多平台软件的开发与调试

- 当服务器启动并执行 Hypervisor 时，它会加载所有虚拟机客户端的操作系统同时会分配给每一台虚拟机适量的内存，CPU，网络和磁盘。它允许多个操作系统（称为**Guest OS**）在同一台物理机器（称为**Host**）上同时运行，每个操作系统被隔离在独立的虚拟机中。
- Hypervisor 负责资源分配，如 CPU、内存和存储，协调着这些硬件资源的访问，而且在各个虚拟机之间施加防护，处理虚拟机之间的通信和安全隔离。

## Classification

![KVM vs VMware – Uma comparação de Hypervisors - Wintech](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/VMwarevsKVM1.png)

- **Type 1 (裸机型):** 直接运行在物理硬件上，例如 VMware ESXi、Microsoft Hyper-V、KVM。

  ![体系结构图，显示 VM 如何运行完整的操作系统（独立于主机操作系统）](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/virtual-machine-diagram.svg)

  KVM (Kernel-Based Virtual Machine) 是 x86 硬件上 Linux 的完整虚拟化解决方案。它由提供核心虚拟化基础设施的可加载内核模块 kvm.ko 和处理器特定模块 kvm-intel.ko 或 kvm-amd.ko 组成。采用硬件辅助虚拟化技术 Intel-VT、AMD-V，内存的相关如 Intel 的 EPT 和 AMD 的 RVI 技术

- **Type 2 (托管型):** 运行在操作系统之上，例如 VMware Workstation、Oracle VirtualBox。

- **QEMU**: 

  - 最初 **QEMU**(Quick Emulator)是一个系统模拟器，它可以在一种体系结构的系统上模拟另一种体系结构的系统。不但模拟 CPU，还模拟各种外设。其中 CPU 的模拟主要是通过指令翻译的方式，所以速度比较慢：

    ![img](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/e45iakbuhu.jpeg)

  - KVM 只靠上面的两个内核模块不能创建完整的虚拟机，KVM 是一种中间件，属于 Linux 的内核模块，将 Linux 变成一个 Hypervisor，可以虚拟化 CPU 和内存，在系统需要虚拟化功能的时候，内核把 KVM 模块调入内存中运行。但用户无法直接控制内核，所以需要一个处于内核和用户之间的一个桥梁 QEMU。KVM 团队 fork 了 QEMU 作为用户态的部分，一起实现了虚拟机(QEMU 模拟系统的其他组件如磁盘等；KVM 模拟 CPU 和内存)。

  - 从 KVM 的角度看，它的用户态部分在 QEMU 里，一起虚拟出完整 Guest OS；

  - 从 QEMU 的角度看，它有一个 Virtualization 模式，就是使用 KVM 来模拟 CPU 内存网络，实现加速。QEMU 通过 KVM 的用户态部分访问 KVM 的内核态部分。

    ![figure1](https://pub-9e727eae11e040a4aa2b1feedc2608d2.r2.dev/PicGo/kvm-qemu-1735307591402-91.png)

某些文献中，VMM 特指 Type 2 Hypervisor 中用于模拟 CPU 指令集和设备驱动的核心软件组件，特别是在微内核环境中，对 Hypervisor 和 VMM 进行了区分。在那里，两个组件形成了某个系统的整体虚拟化堆栈。 Hypervisor 指的是内核空间功能，VMM 指的是用户空间功能。具体来说，在这些上下文中，Hypervisor 是实现虚拟化基础设施的微内核，由于技术原因必须在内核空间中运行，例如 Intel VMX。实现虚拟化机制的微内核也称为微管理程序。将此术语应用到 Linux 中，KVM 是一个 Hypervisor，而 QEMU 是利用 KVM 作为 Hypervisor 的 VMM。

## CPU Virtualization

## Memory Virtualization