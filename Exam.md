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

---

1. L3 can leverage segment registers to organize protection domains the same hardware address space;
2. Because we've organized protection domains in the same hardware address space, we won't need to flush the TLB so we don't give up locality when switching domains
3. the cost of switching domains will be limited to saving/updating segment registers corresponding to the domains

---

SPIN does not rely on hardware address space to enforce logical protection domain while L3 does. However, L3 construction shows that the cost for border crossing, including TLB and cache misses is only 123 processor cycles. As matter of fact, even if you hardcode machine instructions, it would still take 107 processor cycles to complete border crossing. Thus the cost of border crossing due to hardware- enforced protection domain is not as expensive as believed to be.

---

Even without AS tagging, L3 takes advantage of whatever the hardware arch offers for hardware enforced segment bounds. Hardware address space can be carved out into segment domains. Even without address tagging, the process knows which segment registers aka hardware address spaces it can access. For larger protection domains, the address space switching doesn’t even matter since implicit cache costs dominate, an issue that exists in both SPIN and L3.

### 1.3.b (3 points)

Your friend thinks that L3’s approach to OS structuring would incur more implicit cost for protection domain switch compared to a monolithic design. Is she right or wrong? Defend your stand with justification.

---

1. L3 minimal ensures warm cache for small protection domains via minimal abstractions (to avoid polluting the cache) in the mKernel + optimization for the architecture 
2. Monolith with separate hardware address spaces will need to flush the TLB in this architecture (no address-space tagging support) to distinguish between VPNs
This means L3 would incur less implicit costs (cache locality)

---

Wrong.
Implicit cost comes from loss of cache locality.
In L3, small protection domains are packed into the same HW address space and protected by segment registers associated with the protection domains. There is no HW address space switch when switching between these small protection domains, so there is no more loss of locality than there would be in a monolithic OS.
While switching between large protection domains in L3 requires a HW address space switch, the resulting loss of locality would be similar in a monolithic kernel. Since the protection domain is large, its working set is likely not in the cache anyway, so non-locality when jumping into a large protection domain is inherent to the protection domain, regardless of whether it is implemented in L3 or a monolithic kernel.

## 1.a (3 points)

Give one example of how SPIN’s intellectual contribution can be traced to the state-of-the-art in modern systems.

---

1. SPIN influenced the dynamic loading of device drivers in modern OS's (e.g., downloading code to the kernel) 
2. Modern OS's, like monoliths, have adopted internal micokernel-based design influenced by SPIN

---

Ideas of extensibility (espoused in SPIN and Exokernel as exemplars) have had no impact on the state-of-the-art in operating systems structuring.
False.
Many modern operating systems have internally adopted a microkernel-based design.
Similarly, technologies like dynamic loading of device drivers has come out of the thoughts that went into extensibility of OS services. 

## 1.b (3 points)

Give one example of how Exokernel’s intellectual contribution can be traced to the state-of-the-art in modern systems.

---

Ideas that hypervisors take from exokernel: keeping a personality for each guest, de-multiplexing external events (e.g., interrupts) to the specific guest, accounting for time for each guest OS execution.
all trace back to exokernel

---

Today, much of the computing happens in datacenters. The secret sauce for running any application in the cloud is virtualization of the hardware so that ANY OS can run in a datacenter which in turn allows ANY application on that OS to run in a datacenter.

The role of the hypervisor which is in charge of the physical hardware bears a lot of resemblance to the ideas in Exokernel. The main difference is while was exposing hardware at a very fine granularity to the library OS executing on top (to allow specialization of individual system services), the hypervisor provides access to the hardware resources to do specialization at an OS-level granularity as opposed to individual system services. 

Specifically, ideas in virtualization such as keeping a personality for each guest OS, de-multiplexing external events (e.g., network interrupt) to the specific guest OS, accounting time for each guest OS execution can all be traced to similar mechanisms in Exokernel.

## 1.c (T/F + justification) (3 points)

SPIN and Exokernel are fair in comparing the superiority of their respective specialization approaches for OS services relative to Mach micro-kernel.

---

SPIN and Exokernel both aim for extensibility and performance while Mach focus on portability. Mach's focus on the ability to run independently of hardware architecture leads to a bloated code base. This results in lesser locality and longer latency for border crossing. Overall the Mach's memory footprint is the culprit for expensive border crossing and not the microkernel based structure itself. This is the tradeoff in terms of performance for portability. 

---

1. True. SPIN and Exokernel succeeded in designing a microkernel-based OS that provides extensibility without giving up performance. 
2. False. MACH was designed for portability and extensibility using a microkernel design and did not aim to optimize for performance. This is why MACH performs poorly (bloated kernel) in order to optimize for portability and it is NOT fair to compare MACH to SPIN and EXO on performance

## 1.d (8 points)

__HIDDEN__

## 1.e (T/F + justification) (2 points)

SPIN is a microkernel

---

False.

* SPIN only provides header files for functions for each subsystem that form the core components of an OS, and an event mechanism to upcall to these functions. It is not a microkernel.

> Aside: The title of the SPIN paper is "SPIN – An Extensible Microkernel for Application-specific Operating System Services", so I'm not sure why our professor is classifying it as something else.

## 1.f (T/F + justification) (2 points)

Exokernel is a microkernel

---

False.

* Exokernel does not provide any abstractions (e.g, threads, IPC, virtual memory) that a microkernel would.
* It is just a set of mechanisms to securely expose hardware resources at a fine granularity for building system services above exokernel. Exokernel is not a microkernel

# 2. Virtualization (Paravirtualization) (34 points)
 
You have implemented Xenolinux on top of Xen. You have implemented the network layer that does packet send/receive using two I/O rings: one for transmit and one for receive.

## 2.a (Paravirtualization) (4 points)

How will you ensure zero-copy semantics (i.e., no copying from your Xinolinux to Xen) for transmitting a packet?

---

By using the transmission I/O ring: 
1. Xenolinux pins the chosen page frames until transmission is complete (i.e., the memory buffer).
2. XenoLinux checks if a descriptor is available in the xmit ring buffer it shares with Xen
3. XenoLinux populates the descriptor (i.e., origin+dest address, pointer to memory buffer/pinned page) onto the transmit I/O ring
4. Xen uses this descriptor with the pegged page to DMA with NIC
5. Xen enqueues the response in the descriptor;
6. XenoLinux polls the xmit ring for a response from Xen, unpins the page after receiving the result
7. The result of the transmission is available;

## 2.b (Paravirtualization) (4 points)

How will you ensure zero-copy semantics for receiving a packet from Xen into Xinolinux?

---

By using the receiving I/O ring:
1. Xen puts received packets in Xenolinux's recv-ring if it is the intended destination.
2. XenoLinux must pin the pages (i.e., buffer) that it wants to put the packets on.
3. Xen allocates a descriptor for the recv-ring and fills it with the details of the incoming packet.
4. We avoid copying because Xenolinux pins pages for Xen to use the DMA received packets via Xenolinux's pinned page - we are exchanging received packet for XenoLinux's page

thus, we have completed a reception without copying

## 2.c (Paravirtualization) (4 points)

Multiple processes on top of Xinolinux wish to transmit at the same time. How do you handle this situation in your implementation?

---

- XenoLinux must queue requests using the I/O rings.
- XenoLinux is the only writer to the rings' request producer pointer; so it has exclusive control over making requests; Xenolinux can orchestrate when requests are queued
- Xenolinux can tell if it must wait via the distance between the request producer and request consumer pointers in the I/O rings
- if ever there is a process that wants to make a request but there is no slots left in the I/O ring, xenolinux can schedule the process to try again later
- After Xenolinux has made its queueing decisions, it makes a hypercall to alert Xen (source Xen paper)
- in turn, Xen can alert Xenolinux of a response either by software interrupt OR having xeno poll for response using the response consumer pointer
- Xenolinux de-multiplexes responses the descriptor to the proces

The above means Xenolinux can coordinate multiple transmissions from different processes thanks to I/O ring and de-multiplexing via descriptors.

Xen on the otherhand can round robin rings to handle requests

__TODO: this was copied and needs to be revised__

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
