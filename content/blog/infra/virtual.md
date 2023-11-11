---
title: "Configuring Virtual Machines for Speed and Performance"
date: 2023-11-11T21:50:29+05:30
tags: ["virtualization", "vm", "hypervisor", "storage"]
draft: false
---

Configuring virtual machines (VMs) within a virtualization hypervisor involves a delicate dance of resource allocation. In this technological ballet, administrators must choreograph the interplay between CPU, memory, and storage settings to ensure not only optimal VM performance but also the efficient utilization of the underlying hardware resources.

In the realm of virtualization, achieving harmony between the demands of VM workloads and the capabilities of the host machine is akin to composing a symphony where each instrument, in this case, each virtual machine, contributes to the overall performance without overshadowing its counterparts. This article explores few nuanced art of configuring VMs, emphasizing the need to strike a delicate balance that maximizes both individual VM efficiency and the collective utilization of the hardware. Let's delve into the few concepts  that form the notes and rhythms of this orchestration, guiding administrators to create a virtuoso performance in the realm of virtualized infrastructure.


## CPU

> A CPU Socket is a physical connector on the motherboard to which a single physical CPU is connected. A motherboard has at least one CPU socket. Server motherboards usually have multiple CPU sockets that support multiple multicore processors. CPU sockets are standardized for different processor series. Intel and AMD use different CPU sockets for their processor families.

> A CPU (central processing unit, microprocessor chip, or processor) is a computer component. It is the electronic circuitry with transistors that is connected to a socket. A CPU executes instructions to perform calculations, run applications, and complete tasks. When the clock speed of processors came close to the heat barrier, manufacturers changed the architecture of processors and started producing processors with multiple CPU cores. To avoid confusion between physical processors and logical processors or processor cores, some vendors refer to a physical processor as a socket.

> A CPU core is the part of a processor containing the L1 cache. The CPU core performs computational tasks independently without interacting with other cores and external components of a “big” processor that are shared among cores. Basically, a core can be considered as a small processor built into the main processor that is connected to a socket. Applications should support parallel computations to use multicore processors rationally.

> Hyper-threading is a technology developed by Intel engineers to bring parallel computation to processors that have one processor core. The debut of hyper-threading was in 2002 when the Pentium 4 HT processor was released and positioned for desktop computers. An operating system detects a single-core processor with hyper-threading as a processor with two logical cores (not physical cores). Similarly, a four-core processor with hyper-threading appears to an OS as a processor with 8 cores. The more threads run on each core, the more tasks can be done in parallel. Modern Intel processors have both multiple cores and hyper-threading. Hyper-threading is usually enabled by default and can be enabled or disabled in BIOS. AMD simultaneous multi-threading (SMT) is the analog of hyper-threading for AMD processors.

> A vCPU is a virtual processor that is configured as a virtual device in the virtual hardware settings of a VM. A virtual processor can be configured to use multiple CPU cores. A vCPU is connected to a virtual socket.

> CPU overcommitment is the situation when you provision more logical processors (CPU cores) of a physical host to VMs residing on the host than the total number of logical processors on the host.

> NUMA (non-uniform memory access) is a computer memory design used in multiprocessor computers. The idea is to provide separate memory for each processor (unlike UMA, where all processors access shared memory through a bus). At the same time, a processor can access memory that belongs to other processors by using a shared bus (all processors access all memory on the computer). A CPU has a performance advantage of accessing own local memory faster than other memory on a multiprocessor computer.

Ref: https://www.nakivo.com/blog/the-number-of-cores-per-cpu-in-a-virtual-machine/

### Reservations

CPU reservations can be beneficial in scenarios where certain VMs require guaranteed access to CPU resources. For critical applications, such as databases or application servers, consider setting a CPU reservation. This ensures that the VM receives its allocated CPU resources even during periods of contention. However, be cautious about over-reserving, as it may lead to underutilization of resources.

### Shares

VMware allows the allocation of CPU shares to prioritize VMs during contention. VMs with higher share values receive a proportionally larger amount of CPU time when resources are constrained. For example:

>VM1: 2000 shares

>VM2: 1000 shares

VM1 would get twice the CPU time compared to VM2 during contention. Adjust share values based on the relative importance of VMs.

## Storage

**Thin Provisioning**: This method allocates storage on demand, saving space initially. While it allows for efficient use of storage, it's important to monitor and manage potential overconsumption to avoid running out of space.

**Thick Provisioning**: This method allocates all the storage space upfront. It provides better performance and avoids potential issues related to overconsumption, making it suitable for critical workloads. However, it may lead to less efficient use of storage.


## Conclusion
Configuring VMs requires a thoughtful approach to balance performance, resource utilization, and flexibility. By following best practices, administrators can ensure optimal VM performance while maximizing the use of available hardware resources. Regular monitoring and adjustments based on workload demands are key to maintaining an efficient and responsive virtualized environment.

