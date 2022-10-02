# -1. Helpful Resources

[https://docs.google.com/document/d/1Y4LsvVK9yQWaJaJwAlarXVf2W4kfDEPNKUwQ4nYXbRI/edit#](https://docs.google.com/document/d/1Y4LsvVK9yQWaJaJwAlarXVf2W4kfDEPNKUwQ4nYXbRI)

[https://docs.google.com/document/d/1oy_Xa8F4Ql4VTws-oVIISL1j1bQLTqmTt-Bs5ho2Ko0/edit](https://docs.google.com/document/d/1oy_Xa8F4Ql4VTws-oVIISL1j1bQLTqmTt-Bs5ho2Ko0)

# 0. (1 point)

OMSCS program was launched in

---

2014

# 1. OS Structures (33 points)

Imagine you are at the SOSP session wherein all three papers SPIN, Exokernel, and L3 microkernel are presented. Your friend and you are comparing notes on what you got out of each paper in terms of intellectual contributions that advances the state of the art in OS structuring.

## 1.1 (3 points)

Your friend thinks SPIN has shown conclusively that an entire OS can be written in a high-level language that provides strong type safety. Would you agree with her point of view?

---

* SPIN was written primarily in the high-level language Modula-3 which provided static typing and strong typing
  * This is in constrast to many other operating systems of the day, and even most operating systems today, which were written primarily in the low-level language C
* SPIN incorporated the C language alongside Modula-3 for some low-level operations
* SPIN's construction in Modula-3 proved that high-level languages can be used to build many parts of an operating system
* SPIN did not prove that an entire operating system can be witten in a high-level language

__TODO: revise__

## 1.2 (3 points)

Upon page fault service by a library OS, the mapping `<vpn, pfn>` has to be installed into the TLB by Exokernel on behalf of the library OS. Therefore, your friend thinks that Exokernel cannot provide good performance compared to a monolithic design of an OS. How would you counter her argument?

---

* Monoliths have a clear performance advantage when it comes to creating TLB entries
  * Monoliths can simply create the TLB entry when handling a page fault
  * Library OS's must create a secure binding when writing to the TLB. Creating a secure binding adds overhead
  * The cost of secure bindings is greatly reduced with a software TLB (STLB), but the cost is still present
  * The cost of writing to the TLB can be amortized over the lifetime of the TLB entry
    * This benefit is reduced or even entirely eliminated when a programs working set is greater than the capacity of the TLB 
* Exokernel can be faster than monoliths in other situations
  * Network packet filtering

## 1.3

L3 microkernel requires each subsystem to be in distinct architecture enforced protection domain. L3 is implemented on an architecture wherein there is no address-space tagging support in the TLB. The architecture does support segment registers.

### 1.3.a (3 points)

Your friend thinks that the performance is going to be terrible compared to SPIN due to the need for using architecture-enforced protection domains. What would be your counter argument?

---

__TODO__

### 1.3.b (3 points)

Your friend thinks that L3’s approach to OS structuring would incur more implicit cost for protection domain switch compared to a monolithic design. Is she right or wrong? Defend your stand with justification.

---

__TODO__

## 1.a (3 points)

Give one example of how SPIN’s intellectual contribution can be traced to the state-of-the-art in modern systems.

---

__TODO__

## 1.b (3 points)

Give one example of how Exokernel’s intellectual contribution can be traced to the state-of-the-art in modern systems.

---

__TODO__

## 1.c (T/F + justification) (3 points)

SPIN and Exokernel are fair in comparing the superiority of their respective specialization approaches for OS services relative to Mach micro-kernel.

---

__TODO__

## 1.d (8 points)

__HIDDEN__

## 1.e (T/F + justification) (2 points)

SPIN is a microkernel

---

False. SPIN is described as a library

> Aside: The title of the SPIN paper is "SPIN – An Extensible Microkernel for Application-specific Operating System Services", so I'm not sure why our professor is classifying it as something else.

## 1.f (T/F + justification) (2 points)

Exokernel is a microkernel

---

False. The Exokernel architecture stands apart from Microkernels through its use of low-level primitives. This is what allows Exokernels to have performance that is superior to microkernels. __TODO: Revise__

# 2. Virtualization (Paravirtualization) (34 points)
 
You have implemented Xenolinux on top of Xen. You have implemented the network layer that does packet send/receive using two I/O rings: one for transmit and one for receive.

## 2.a (Paravirtualization) (4 points)

How will you ensure zero-copy semantics (i.e., no copying from your Xinolinux to Xen) for transmitting a packet?

---

1. Data to be transmitted is stored in a buffer owned by Xenolinux
1. Xenolinux will create a descriptor and issue a hypercall to enqueue the descriptor into the IO ring
1. If Xenolinux has the capability to swap pages to disk, then Xenolinux must pin the virtual pages containing the buffer so that the data is in memory for Xen to access during transmission.

The above is sufficient to enable zero-copying of transmitted data between Xenolinux and Xen. Xen is able to directly access the data to be transmitted through the buffer owned by Xenolinux

## 2.b (Paravirtualization) (4 points)

How will you ensure zero-copy semantics for receiving a packet from Xen into Xinolinux?

---

This is very similar to the previous case.

1. Data to be received will be stored in a pre-allocated buffer owned by Xenolinux

__TODO: elaborate__

## 2.c (Paravirtualization) (4 points)

Multiple processes on top of Xinolinux wish to transmit at the same time. How do you handle this situation in your implementation?

---

__TODO__

## 2.d (Full virtualization) (8 points)

Assume a guest-OS has started 4 processes in a fully virtualized environment on a 32-bit machine. Assuming 4K page size, explain how many entries this guest-OS has in the shadow page table.

## 2.e (Full virtualization) (4 points)

Assume an architecture which uses a page table for address translation. The CPU has a PTBR to point to the current page table used by the processor for address translation.

A process P1 is executing on top of a fully virtualized OS. The OS wishes to context switch from P1 to P2.

List the steps before P2 starts execution on the processor.

---

__TODO__

## 2.f (10 points)

__HIDDEN__

# 3. Parallel Systems (32 points)

## 3.a (atomicity) (4 points)

You are implementing an invalidation-based cache coherent shared memory multiprocessor, wherein each processor has a private cache. You start with a uniprocessor as a basic building block. The ISA of the processor supports an atomic Test-and-Set (T&S) instruction. Your aim is to make sure that the T&S operation is globally atomic. What design choices are available to you to achieve this aim?

---

__TODO__

## 3.b (memory model) (4 points)

Your co-worker wants to provide a sequential consistency memory model to the application programmer on top of your multiprocessor. How can you take care of her requirement in your cache coherent multiprocessor design?

---

__TODO__

## 3.c (spinlock) (4 points)

Consider the following lock algorithm using T&S:

```C
while ((L == locked) or (T&S(L) == locked)) {
  while (L == locked); // spin
  delay (d[Pi]); // different delays for different processors
}
// success if we are here
```

### 3.c.1 (T/F + justification) (No credit without justification) (2 points?)

This algorithm does not rely on hardware cache coherence.

---

__TODO__

### 3.c.2 (T/F + justification) (No credit without justification) (2 points?)

The algorithm performs especially well under high lock contention.

---

__TODO__

## 3.d (spinlock) (4 points)

The ticket lock algorithm shown below gives fairness and each spinning processor spins for a different amount of time commensurate with its expected wait time for the lock before testing if the lock is available.

![ticket lock algorithm](./screenshot.png)

What (if any) are the reasons for this algorithm to not work well?

---

__TODO__

## 3.e (barrier) (T/F + justification) (No credit without justification) (6 points)

### 3.e.1 (2 points?)

MCS barrier will not work on a NCC-NUMA architecture.

---

__TODO__

### 3.e.2 (2 points?)

The total communication complexity of dissemination barrier is O(N\*Log2N)

---

__TODO__

### 3.e.3 (2 points?)

The tournament barrier works with both shared memory and message-passing (i.e., clusters) architectures.

---

__TODO__

## 3.f (2 points)

__HIDDEN__

## 3.g (scheduling) (4 points)

A multi-threaded multicore CPU is one in which each chip has multiple cores and each core has multiple hardware threads. The OS chooses the set of application threads to be scheduled on the hardware threads in each core. Given that the hardware threads share a single processor pipeline on the core,

### 3.g.1

What purpose is served by the hardware threads?

---

__TODO__

### 3.g.2

What should the OS do ensure that processor pipeline is utilized well? Why?

---

__TODO__

## 3.h (memory manager for multiprocessor) (4 points)

You are implementing the virtual memory manager for your multiprocessor OS.

You have a page fault handler that executes independently in each processor. If there is a page fault for the currently executing thread/process, then the handler on that processor deals with it without disturbing the activities on the other processors. Your OS supports both single-threaded processes as well as multi-threaded processes. You implement your memory management system in the conventional manner with a page table per process that provides the mapping of the VPN to PPN (or the disk address if it is not in physical memory).

### 3.h.1 (2 points?)

Does your design ensure that if there are concurrent page faults incurred by independent processes running on different processors, they will be handled by your memory manager concurrently? Justify your answer.

---

__TODO__

### 3.h.2 (2 points?)

Does your design ensure that if there are concurrent page faults incurred by threads of the same process running on different processors, they will be handled by your memory manager concurrently? Justify your answer.

---

__TODO__
