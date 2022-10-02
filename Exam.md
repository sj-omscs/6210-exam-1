# -1. Helpful Resources

[Test Questions](https://docs.google.com/document/d/1Y4LsvVK9yQWaJaJwAlarXVf2W4kfDEPNKUwQ4nYXbRI/edit?usp=sharing)

[Practice Questions](https://docs.google.com/document/d/1oy_Xa8F4Ql4VTws-oVIISL1j1bQLTqmTt-Bs5ho2Ko0/edit?usp=sharing)

# 0. (1 point)

OMSCS program was launched in

---

2014

# 1. OS Structures (33 points)

Imagine you are at the SOSP session wherein all three papers SPIN, Exokernel, and L3 microkernel are presented. Your friend and you are comparing notes on what you got out of each paper in terms of intellectual contributions that advances the state of the art in OS structuring.

## 1.1 (3 points)

Your friend thinks SPIN has shown conclusively that an entire OS can be written in a high-level language that provides strong type safety. Would you agree with her point of view?

---

No.

SPIN was primarily written in the high-level language Modula-3, but portions of it was written in C.

## 1.2 (3 points)

Upon page fault service by a library OS, the mapping `<vpn, pfn>` has to be installed into the TLB by Exokernel on behalf of the library OS. Therefore, your friend thinks that Exokernel cannot provide good performance compared to a monolithic design of an OS. How would you counter her argument?

---

* Exokernel will be slower when writing to the TLB
  * Secure Bindings
    * Exokernels must create a secure binding when writing to the TLB
    * STLB can cache these bindings
    * Cost of creating the binding can be amortized over the lifetime of the TLB entry
    * Programs with a working set larger than the TLB will see a greater slowdown
* Exokernels can be faster in other situations as shown by Aegis and ExOS
  * Handling system calls and dispatching exceptions
  * Using IPC mechanisms like pipes and shared memory

---

## 1.3

L3 microkernel requires each subsystem to be in distinct architecture enforced protection domain. L3 is implemented on an architecture wherein there is no address-space tagging support in the TLB. The architecture does support segment registers.

### 1.3.a (3 points)

Your friend thinks that the performance is going to be terrible compared to SPIN due to the need for using architecture-enforced protection domains. What would be your counter argument?

---

* Segment registers allow L3 to implement protection domains in one address space. The segment registers will enforce protection.
  * Using one address space allows L3 to skip a TLB flush for small protection domains
* L3 is able to handle system calls much more efficiently
* Experimental evidence proved that L3 is significantly faster than SPIN in IPC

### 1.3.b (3 points)

Your friend thinks that L3’s approach to OS structuring would incur more implicit cost for protection domain switch compared to a monolithic design. Is she right or wrong? Defend your stand with justification.

---

Wrong.

* L3 will not be any less efficient due to implicit costs than a Monolith
* The implicit cost of switching protection domains comes from loss of cache locality in the CPU and TLB.
* There is no added implicit cost when switching protection domains in L3 because the TLB does not need to be flushed since the address space can be shared between protection domains.
* Large protection domains will require a TLB flush which increases implicit cost, but the same cost would be incurred for a monolith with subsystems that have large working sets.

## 1.a (3 points)

Give one example of how SPIN’s intellectual contribution can be traced to the state-of-the-art in modern systems.

---

SPIN helped prove the idea of dynamically loading code into the kernel, which can be seen today with device drivers. 

## 1.b (3 points)

Give one example of how Exokernel’s intellectual contribution can be traced to the state-of-the-art in modern systems.

---

Many of the techniques that Exokernel used can be seen in modern hypervisors. One instance of this is how Exokernel keeps track of execution time.

## 1.c (T/F + justification) (3 points)

SPIN and Exokernel are fair in comparing the superiority of their respective specialization approaches for OS services relative to Mach micro-kernel.

---

True.

* Both SPIN and Exokernel are saught to provide an alternative to the monolithic OS architecture.
* Mach placed value on portability which led to its poor performance
* L3 may have been a better comparison since it focuses more on performance than portability, which is the same approach that both SPIN and Exokernel took
* Exokernel provided another well performing alternative to microkernels

## 1.d (8 points)

__HIDDEN__

## 1.e (T/F + justification) (2 points)

SPIN is a microkernel

---

False.

* The SPIN paper describes itself as a microkernel, but we have define microkernel to mean a kernel in which OS systems run in their own address spaces.
* SPIN dynamically loads extensions into the kernel's address space.
* This difference is a large part of SPINs greater performance.

> Aside: The title of the SPIN paper is "SPIN – An Extensible Microkernel for Application-specific Operating System Services", so I'm not sure why our professor is classifying it as something else.

## 1.f (T/F + justification) (2 points)

Exokernel is a microkernel

---

False.

* Exokernel only provides primitives, whereas a traditional microkernel might provide abstractions like threads and virtual memory.
  * This makes Exokernels signicantly more simple that a microkernel.
* Exokernel allows dynamically uploading code into the kernel similar to SPIN.

# 2. Virtualization (Paravirtualization) (34 points)
 
You have implemented Xenolinux on top of Xen. You have implemented the network layer that does packet send/receive using two I/O rings: one for transmit and one for receive.

## 2.a (Paravirtualization) (4 points)

How will you ensure zero-copy semantics (i.e., no copying from your Xinolinux to Xen) for transmitting a packet?

---

With the transmisison IO ring:

1. Xenolinux will allocate a buffer that contains the data to be transmitted
1. Xenolinux forms a descriptor containing the address of the buffer
1. Xenolinux pins the pages containing the buffer so that Xen can access them during transmission
1. Xenolinux issues a hypercall to queue the request
1. Xen transmits the data by directly accessing the buffer
1. Xenolinux poll the ring for a response from Xen
1. When Xen responds, Xenolinux can unpin the page

## 2.b (Paravirtualization) (4 points)

How will you ensure zero-copy semantics for receiving a packet from Xen into Xinolinux?

---

1. Xenolinux pre-allocates buffers to hold incoming packets
1. Xenolinux provides the locations of the buffers to Xen
1. Xenolinux pins the buffers so that Xen can access them
1. Xen directly copies data into a buffer provided by Xenolinux
1. Xen enqueues a descriptor to let Xenolinux know about the data in the buffer
1. Xenolinux handles the enqueued descriptor and is able to directly access the data

## 2.c (Paravirtualization) (4 points)

Multiple processes on top of Xinolinux wish to transmit at the same time. How do you handle this situation in your implementation?

---

* Xenolinux provides a mutex that is locked when attempting to transmit data
  * Processes will wait for their turn to access the mutex
* Xenolinux will check for space in the IO ring before transmitting data
  * Processes will wait until there is space in the IO ring
* Data transfer can occur as usual

Xen will use a round-robin algorithm to transmit packets acrosss multiple VMs to ensure fairness.

## 2.d (Full virtualization) (8 points)

Assume a guest-OS has started 4 processes in a fully virtualized environment on a 32-bit machine. Assuming 4K page size, explain how many entries this guest-OS has in the shadow page table.

---

__TODO__

## 2.e (Full virtualization) (4 points)

Assume an architecture which uses a page table for address translation. The CPU has a PTBR to point to the current page table used by the processor for address translation.

A process P1 is executing on top of a fully virtualized OS. The OS wishes to context switch from P1 to P2.

List the steps before P2 starts execution on the processor.

---

1. Guest OS tries to execute a privileged instruction to set the PTBR (page table base register) to the new process’s page table data structure (the PPN that contains this process’s page table).
1. The ensuing trap is caught by the hypervisor. 
1. Hypervisor uses the shadow page table for this Guest OS to get the MPN that corresponds to this PPN. This gives the page table for this process in machine memory.
1. Hypervisor sets PTBR to this MPN.
1. The Guest OS does the other necessary steps for the normal context switch. (e.g. load/store volatile state to/from PCB) all of which is accomplished using “trap and emulate” method.


## 2.f (10 points)

__HIDDEN__

# 3. Parallel Systems (32 points)

## 3.a (atomicity) (4 points)

You are implementing an invalidation-based cache coherent shared memory multiprocessor, wherein each processor has a private cache. You start with a uniprocessor as a basic building block. The ISA of the processor supports an atomic Test-and-Set (T&S) instruction. Your aim is to make sure that the T&S operation is globally atomic. What design choices are available to you to achieve this aim?

---

1. Execute the RMW operating in the cache controller: Use the invalidation based CC protocol to obtain exclusive ownership for the memory location on which T&S is being performed and execute the RMW operation 
1. Extended ownership of the interconnect: Give exclusive ownership of the interconnect (e.g., bus) so that the processor can complete the T&S operation to the memory location bypassing the cache. 
1. Execute the RMW in the memory controller: Bypass the cache and send the T&S request to the memory controller. The memory controller serializes the T&S operation from different processors to the same memory location.


## 3.b (memory model) (4 points)

Your co-worker wants to provide a sequential consistency memory model to the application programmer on top of your multiprocessor. How can you take care of her requirement in your cache coherent multiprocessor design?

---

* Sequential consistency can be ensured by using cache coherence protocols to ensure exclusive access to a memory location before writing to it. 
* For e.g., if we use an invalidate based coherence protocol, all cached copies of the cache line will be invalidated before a processor writes to that cache line. Other processors can fetch the updated cache line after the write (from the processor or memory).


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

False, this algorithm does use hardware cache coherence. It spins on the cached variable, waiting for it to be released. Once released, it will try to acquire the lock again

---

False.
Processors spin on cached value (While (L==locked)). The only way for them to come out of the spin loop is if the cached value changes which necessitates hardware cache coherence.

### 3.c.2 (T/F + justification) (No credit without justification) (2 points?)

The algorithm performs especially well under high lock contention.

---

True.
Spinlock with static delay is better than spinlock with dynamic delay when there is high lock contention. Each processor gets a different delay, so there is a sequentializing of the order of the processors.

---

True. 
Upon lock release, different processors wait for different amount of delay times thus reducing the contention on the bus

## 3.d (spinlock) (4 points)

The ticket lock algorithm shown below gives fairness and each spinning processor spins for a different amount of time commensurate with its expected wait time for the lock before testing if the lock is available.

![ticket lock algorithm](./screenshot.png)

What (if any) are the reasons for this algorithm to not work well?

---

Note that the question specifies we are using an invalidation based CC protocol

(For an update-based CC protocol):
Upon each lock release L->now_serving is updated which results in unnecessary bus contention, since the change is meaningful for only ONE processor that is next in line to get the lock.

(For invalidation-based protocol):
The duration of pause (spin loop) is just a guestimate of the expected wait time for a processor. Every time it comes out to check if it is its turn for the lock, there would be bus contention to get L->now_serving if its cached value has been invalidated (due to an unlock operation).

## 3.e (barrier) (T/F + justification) (No credit without justification) (6 points)

### 3.e.1 (2 points?)

MCS barrier will not work on a NCC-NUMA architecture.

---

False.
Processors using an MCS barrier spin on statically-determined flag variables only, so these locations can be located in the local memory of each processor, yielding good performance for NUMA architectures.
Arrival and wake-up in MCS use explicit signaling of parents/children using distinct memory locations, so these signals can be sent directly to the parent/children without relying on cache coherence protocols.

---

• False. MCS barrier spins on statically determined locations, so a child updating a parent’s memory (remotely) will also update their cache, since even in NCC-NUMA, the memory hierarchy is locally kept consistent.

### 3.e.2 (2 points?)

The total communication complexity of dissemination barrier is O(N\*Log2N)

---

True
The dissemination barrier is made up of ceil(log2N) ~ log2(N) rounds, and in each round each processor sends a message, leading to a total of N\*Log2N messages being sent.

### 3.e.3 (2 points?)

The tournament barrier works with both shared memory and message-passing (i.e., clusters) architectures.

---

True.
The algorithm works by allowing one statically determined representative process from each round to go to the next round. When on a shared memory architecture (SMP, NUMA/NCC), the algorithm relies on spinning on a statically determined spin location that indicates the state of the barrier. It also works with message passing in cluster architecture, since the algorithm simply requires that the representative process can signal to the other process.

## 3.f (2 points)

__HIDDEN__

## 3.g (scheduling) (4 points)

A multi-threaded multicore CPU is one in which each chip has multiple cores and each core has multiple hardware threads. The OS chooses the set of application threads to be scheduled on the hardware threads in each core. Given that the hardware threads share a single processor pipeline on the core,

### 3.g.1

What purpose is served by the hardware threads?

---

The purpose of multiple hardware threads in each core is to ensure efficient utilization of the single processor pipeline available to each core. If a hardware thread has to perform a long latency operation, e.g., miss in the last level cache (LLC) causing the processor to go off-chip to fetch the missing access from the memory (which can take 100 or more CPU cycles), the hardware can switch to another hardware thread. 

### 3.g.2

What should the OS do ensure that processor pipeline is utilized well? Why?

---

In scheduling the set of threads to run on the chip, the OS should ensure that the combined working sets of all the threads (number of cores X number of hardware threads per core) can be packed into the last level cache to reduce the occurrence of long latency operations (i.e., misses in the LLC).

## 3.h (memory manager for multiprocessor) (4 points)

You are implementing the virtual memory manager for your multiprocessor OS.

You have a page fault handler that executes independently in each processor. If there is a page fault for the currently executing thread/process, then the handler on that processor deals with it without disturbing the activities on the other processors. Your OS supports both single-threaded processes as well as multi-threaded processes. You implement your memory management system in the conventional manner with a page table per process that provides the mapping of the VPN to PPN (or the disk address if it is not in physical memory).

### 3.h.1 (2 points?)

Does your design ensure that if there are concurrent page faults incurred by independent processes running on different processors, they will be handled by your memory manager concurrently? Justify your answer.

---

Yes.
Since we have a page table per process, the page table needed to cater to different page faults would be different.

### 3.h.2 (2 points?)

Does your design ensure that if there are concurrent page faults incurred by threads of the same process running on different processors, they will be handled by your memory manager concurrently? Justify your answer.

---

No.
Since the mappings from VPN to PPN for different threads of the same process will be present in the same page table, the memory manager would need to serialize the handling of these page faults
