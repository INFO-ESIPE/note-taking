[[Cloud Computing]]
****
**Table of Contents**
```table-of-contents
```

****
## Virtualization ?

Virtualization is the **Abstraction of physical resources into virtual resources**
This is used in a lot of technologies:
- **Operating systems**: Virtual memory, threads…
- **Emulators**: Instruction translation
- **Language virtual machines**: Optimised emulator
	*JVM for Java, Python ...*
- **Containers**: Virtual OS
	*Docker, LXC ...*
- **Virtual Machines**: Virtual Hardware
	*QEMU/KVM, Microsoft Hyper-V, VMWare ESXi, VirtualBox (poop)*


### Hardware Virtualization

We virtualize hardware **for multiple OSes at the same time**
	*Virtual CPUs, Additional level of memory addressing, virtual network & storage, IRQs ...*

Actors of hardware virt:
1. Hypervisor
2. Virtual Machines (where guest OSes are run)
3. User Interface (libvirt)

It's the job of the **Hypervisor** to **run Guest OSes in Virtual Machines**.


### Hypervisor

An hypervisor (HV) is **a special OS that runs and manages VMs** (and the guest OSes running in it).

There are two types of hypervisors:
- **Native (type 1)**: Bare metal, guest OSes are processes
	Optimised for **maximum resource virtualization**
	Has only one big task: run OSes 
	*Xen, Citrix...*

![[native.png]]

- **Hosted (type 2)**: HV is a process on an OS, and the guest OSes are subprocesses
	Easy install and use, but not very performant 
	*VirtualBox, QEMU, VMWare Workstation ...*

![[hosted.png]]

For these reasons, **Cloud relies on Type 1 HVs** rather than Type 2... 


### Virtual Machines

A Virtual Machine (VM) is a **cohesive ensemble of virtualized resources that represent a complete machine**
==So, VMs refers to the hardware virtualization. A guest OS is needed!==

VM can have three states:
- **Running**: Memory, I/O queues, processor registers and flags...
	Snapshots allows for backup and checkpointing
- **Suspended**
- **Stopped**: Turned into a disk image
	Stores the guest OS, and allows easy replication

It looks like this:
![[vms.png]]


### Libvirt

Libvirt is an API for controlling virtualization engines
	*Works with XEN, QEMU/KVM, VirtualBox... and can manage networks and storage*

We can launch an interactive libvirt shell with the `virsh` command.

**Basic commands and concepts:**

```bash
# Create a storage pool (a collection of VM images)
virsh pool-define-as mypool dir - - - - /path/to/pool/images
virsh pool-build
virsh pool-start

# Create a volume (a VM image)
virsh vol-create-as mypool myvolume 10GiB --format qcow2

# Create a domain (a VM specification) using virt-install and a given install method (here, CD-ROM). Note that it also creates the volume (so you can skip vol-create-as)
virt-install --name myubuntudomain --os-type=linux --os-variant=ubuntu20.04 \
		--memory 4096 --vcpus=4,maxvcpus=8 --network bridge=virtbr0 \
		--virt-type kvm --cpu host --disk 10,format=qcow2 --cdrom ubuntu.iso

# Start a domain (run a VM)
virsh start myubuntudomain

# Interactively jump into the domain
virt-viewer --connect qemu:///session myubuntudomain

# Shutdown a domain (terminate a VM)
virsh shutdown myubuntudomain
virsh destroy myubuntudomain # Force terminate

# Manually edit XML configuration of a domain
virsh edit myubuntudomain
```


****
## Virtualization in the Cloud

VMs are the cornerstone of the cloud, they are everywhere due to their convenience:
- **Easy deployment**: One VM image can be used to create several VMs (services)
- **Easy administration**: All software, no hardware
- **Seen as a resource unit in the cloud**


### Resource Management

A major advantage of Cloud is the **scalability** it offers.
There are two ways to scale our cloud project:
- **Horizontal (Add VMs)**
	Under load spikes (heavy traffic), we **replicate services**
	When the traffic stabilises, we kill useless replicates
	**Automatic Scaling** can be used
	
- **Vertical (Enlarge VMs)**
	Since all hardware is virtual, we can perform dynamic additions of vCPUs, memory...
	This method is **not recommended** as it is relatively **hard to implement**
		*how to unmap unused memory from the guest OS ?*


But how to fit `N` VMs on `M` physical hosts ?
	*There are a lot of resources to take into account, and hard-to-solve optimisation issues*

Two solutions:
- **Overcommitment**: Since resources are virtual, we assign more than physically available
	*BAD IDEA! Very harmful when it collapses*

- **Migration**: VMs are loosely attached to hosts, so move them around
	*Optimises both resource usage on physical hosts, and datacentre usage by powering only needed hosts*
![[migration.png]]


### Memory Management

Difficult to manage as it involves **spatial sharing**.
	*Unlike CPU which involves **time sharing**, you can simply wait !*

Two solutions again:
- **Overcommitment**: Same as above, still dogshit ...
- **Ballooning**: Reclaim memory from guests by asking for memory pages (inflate).


### Security and Reliability

VMs are completely isolated by nature
	*Different guest OS, virtual hardware, and access policies is managed by the HV*

Checkpointing and resuming mechanism (snapshots) helps handling failures.


****
## Modes of virtualization

There are three ways to virtualize a guest OS:
1. **Full virtualization**: Total simulation of virtual hardware
	*Unmodified guest OS, Binary translation*

2. **Paravirtualization**: Virtualization interface between guest OS and HV
	*Paravirtualized drivers, HV interactions through **hypercalls***

3. **Hardware-assisted virtualization**: The physical hardware helps executing virtualized OS operations
	*Unmodified guest OS, hardware support for virtualized execution (Intel VT-x, AMD-V...)*

![[virtmodes.png]]


****
## CPU Virtualization

The problem we have now is the following: **An operating system requests unlimited and exclusive control over the hardware**
	*It is the OS' job to make abstraction of hardware in order to provide them to processes.
	Unlimited control is given to the Hypervisor :/
	No exclusive control either since several guest OSes will use the hardware :/*

We can fix this issue via **VM Context switching** and **Protection Ring changes**


### Refresher: CPU Protection rings

As briefly covered [[01 - Introduction aux OS#Modes d'exécution|here]], a mechanism called **CPU protection rings** is here to separates different levels of privilege in a computer system. They are used to control access to resources and prevent lower-privilege code from interfering with higher-privilege operations.
	It's a **hardware-enforced security mechanism**, which means this system is **directly on the CPU hardware**...

It is de-composed like so:
- **Ring 0 (Kernel Mode)**: Highest privilege level, **used by the operating system kernel to manage hardware** and critical system functions.
	*If an application needs hardware, the kernel does it on their behalf via [[01 - Introduction aux OS#Appels Systèmes (syscalls)|syscalls]]*

- **Ring 1 & 2 (Drivers or Middleware)**: Historically reserved for device drivers or privileged services.
	*It is pretty much unused nowadays.*

- **Ring 3 (User Mode)**: Lowest privilege level, used by applications and user-level processes, isolated from critical system functions.

![[ring-default.png]]

> Usually what happens is the following: The OS places itself in ring 0 so he can handle everything, and have unique and unlimited control over the hardware.


### Ring Deprivileging - Full Virtualization

Guest userspace is in ring 3.
	*Use hardware through `syscalls` on the kernel*

(Unmodified) Guest kernels are moved to ring 1, and the **Hypervisor is placed in ring 0**.

The Hypervisor **trap requests from guest kernels** (privileged instructions) to ensure he is the only one who can execute instructions directly on the hardware. Normal instructions are executed in the last ring as usual, nothing happens...

![[ring-full.png]]


When a privileged instruction is captured, a technique called **shadowing** is used to provide the guest OS with the illusion that it is interacting directly with hardware, while the hypervisor maintains control and isolation.
	*The hypervisor maintains **shadow copies** of these resources and simulates the guest’s operations.*

For instance:
1. The guest modifies its page table to map virtual memory to physical memory.
2. The hypervisor **intercepts this (trap)** and updates a **shadow page table** that points to the actual physical memory, ensuring isolation between virtual machines (VMs).

> Now, Guest OS thinks its managing the hardware, while he is actually "tricked" by the HV


**Unmodified OS, but huge performance impact**


### Ring Compression - Paravirtualization

Like full virt, Guest userspace is still ring 3 and uses syscalls.
Guest kernel is in ring 0, but the kernel is **modified for paravirtualization**
	*Instead of using the hardware on its own, it will ask the hypervisor to do so via **hypercalls** (API)*

Hypervisor is placed in ring 0 as well.

![[ring-para.png]]

**Offers good performance, but requires work beforehand to modify the guest OS to make it aware of the hypervisor**


### Ring -1 Privilege - Hardware-assisted Virtualization

Modern CPUs provide virtualization extensions (e.g., Intel VT-x, AMD-V) with additional privilege levels like **Ring -1** for the hypervisor.

The hypervisor runs in an **over-privileged ring: -1**, while guest operating systems can use **Ring 0** natively, improving performance by reducing the need for emulation.

![[ring-hardware.png]]

On privileged operations from the guest, transition from VM context to HV context
	*The hard work is moved down to the processor (Intel VT-x, AMD-V)*

**Unmodified guest, very good performance, no downside**


### KVM virtualization and vCPU

Pseudo-code of a vCPU thread:
```c
open("/dev/kvm") // Opens the KVM device, providing access to virtualization features via the kernel
ioctl(KVM_CREATE_VM) // Creates a VM. This allocates resources and prepares it to run
ioctl(KVM_CREATE_VCPU) // Creates a virtual CPU (vCPU) for the VM. This represents a CPU core within the virtualized environment.

for (;;) { 
	ioctl(KVM_RUN) // Executes the VM until it hits an exit condition (e.g., performing I/O, halting, or requiring emulation).

	// Handle virtualization: I/O, VM halt
	switch (exit_reason) { 
		case KVM_EXIT_IO: /* ... */ 
		case KVM_EXIT_HLT: /* ... */ 
	} 
}
```

KVM has this codeflow for Intel VT-x:
![[kvmflow.png]]


****
## Memory Virtualization

Forewords: An **MMU (Memory Management Unit)** is a hardware component in a CPU that translates virtual addresses used by programs into physical addresses in memory. It enables features like virtual memory, memory protection, and address space isolation

The guest OS and the hypervisor both need to manage memory, but they operate in separate layers. 
	*Guest OS believes it has full control over the physical memory, but the hypervisor must abstract and remap this memory to ensure isolation and manageability.*

This is how it works natively:
![[memnative.png]]

And this is how it looks for virtualization:
![[memvirt.png]]

**Question:** How do we implement the virtual MMU at the centre ?


### Shadow Table

The traditional solution. 
Make the hypervisor maintain a **shadow page table** to **map Guest Physical Address (GPA) to Host Physical Address (HPA)**, in addition to the guest OS’s own page table for Guest Virtual Address (GVA) to GPA.

Workflow:
1. The hypervisor intercepts any guest OS changes to its page table.
2. Updates its shadow page table to reflect these changes.
3. Applies the mapping of GPAs to HPAs, ensuring proper redirection.

This solution works, but trapping every guest page table change is computationally expensive and has huge latency.


###  Second Level Address Translation (SLAT)

SLAT (also known as **Nested paging**) is Hardware-assisted memory translation. It optimises memory virtualization by leveraging various hardware features
	*Intel EPT (Extended Page Tables), AMD NPT (Nested Page Tables)*

The CPU directly handles two levels of translation:
- Guest virtual to guest physical (GVA → GPA).
- Guest physical to host physical (GPA → HPA).

This reduces the need for hypervisor intervention, and results in low latency and good performance.


****
## I/O and Devices Virtualization

I/O operations are challenging because guest VMs need to communicate with physical devices while being isolated from one another. 
This requires emulation or redirection of I/O operations.


### Traps and Emulation

The hypervisor traps all guest I/O instructions, and emulates the device behaviour in software using the hypervisor’s physical drivers.

**This is very slow and resource-intensive, we need something else**


### Paravirtualization (virtio)

The guest OS uses **paravirtualized drivers** designed specifically for virtual environments.

It is composed of a **Front-End Driver** running in the guest OS which communicates with the hypervisor, and a **Back-End Driver** running in the hypervisor which manages the actual physical device. 


### Hardware Assistance

Uses hardware features like **IOMMU (Input-Output Memory Management Unit)**, which provides device-level isolation by **mapping guest memory for Direct Memory Access (DMA)**

