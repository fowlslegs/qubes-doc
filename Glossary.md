---
layout: doc
title: Glossary
permalink: /doc/Glossary/
redirect_from: /wiki/Glossary/
---

Glossary of Qubes Terminology
=============================

!! Possibly separate into Virtualization and Trusted Computing sections !!

**Domain or Virtual Machine (VM)**
Xen uses the term domain is used to refer to a VM present on the system. A VM is an emulation of a particular computer system. VMs are software implementations designed to execute programs like a physical machine. They are classified by the degree of completeness to which they emulate a real machine. A brief description of the types of virtual machines supported by Xen (and most **virtual machine monitors (VMMs)**) is as follows:

  **Paravirtualized machines (PVMs)** are run at a ring level above 0 (ring 1 in x86 and ring 3 in x86-64). Thus, if they try to execute privileged CPU instructions (e.g., system calls), these instructions silently fail and the OS running in the PVM is left to deal with the consequences. In practice, however, we should not have to deal with this problem. That is because when an OS is ported to Xen all privileged instruction it may execute are replaced with their respective Xen hypercalls.

  **Hardware-assisted virtual machines (HVMs)** use Intel VT-x or AMD-V hardware technology, both of which extend the x86 instruction set architecture in order to virtualize any OS without the need to replace its privileged instructions with Xen hypercalls. Conceptually, the hypervisor runs in 'ring -1', while the kernel runs in ring 0 (in implementation, more than one ring is added). The OS kernel believes it is running directly on the hardware, while in reality hypervisor is silently trapping and emulating privileged instructions that would otherwise have silently failed.

  **Paravirtualized hardware-assisted virtual machines (PVHVMs)** are HVMs that use software and hardware technologies to speed up system calls, I/O, and memory address translation associated with the second level of indirection that is a result of running a guest OS on top of a hypervisor. I/O is sped up through the use of lightweight interfaces to the devices instead of emulating hardware devices and running their respective drivers in the OS. On a normal HVM, the guest OS must first translate an abstract request (e.g., fetch this adorable cat gif) into a set of instructions its relevant device driver(s) can understand. These instructions are then re-translated into an abstract form that Xen can understand before. Further, PVHVMs may take advantage of second level address translation (SLAT), which moves translation from virtual page from inside the guest system to the memory management. In both these cases we are optimizing how we handle a second layer of indirection.

  ?? Do Qubes PVHVMs actually use SLAT ?? What about accelerated system calls ??
  !! Explain accelerated system calls in better detail !!

**Domain Zero (dom0)**
Also known as the **host** domain, dom0 is the initial domain started by the Xen hypervisor on boot. Dom0 runs the Xen management toolstack and has special privileges relative to other domains, such as direct access to most hardware. Two Xen daemons run in dom0. **xend** provides an administrative interface to Xen to allow a user to define a **policy**. **xenstored** provides the backend storage for the **Xen Store**.

  !! Create xend entry maybe !!

**Policy**
A system policy refers to a complete configuration of the trusted computing base (TCB) of that system.

**DomU**
Unprivileged Domain. Also known as **guest** domains, domUs are the counterparts to dom0. All domains except dom0 are domUs. By default, most domUs lack direct hardware access.

**AppVM**
Application Virtual Machine. Any VM which depends on a TemplateVM for its root filesystem. In contrast to TemplateVMs, AppVMs are intended for running software applications.

**TemplateVM**
Template Virtual Machine. Any standalone VM which supplies its root filesystem to other VMs, known as AppVMs. In contrast to AppVMs, TemplateVMs are intended for installing and updating software applications.

**Standalone(VM)**
Standalone (Virtual Machine). In general terms, a VM is described as **standalone** if and only if it does not depend on any other VM for its root filesystem. (In other words, a VM is standalone if and only if it is not an AppVM.) More specifically, a **StandaloneVM** is a type of VM in Qubes which is created by cloning a TemplateVM. Unlike TemplateVMs, however, StandaloneVMs cannot supply their root filesystems to other VMs. (Therefore, while a TemplateVM is a standalone VM, it is not a StandaloneVM.)

**Template-BasedVM**
Opposite of a Standalone(VM). A VM, that depends on another TemplateVM for its root filesystem.

**NetVM**
Network Virtual Machine. A type of VM which connects directly to a network and provides access to that network to other VMs which connect to the NetVM. A NetVM called `netvm` is created by default in most Qubes installations.

**ProxyVM**
Proxy Virtual Machine. A type of VM which proxies network access for other VMs. Typically, a ProxyVM sits between a NetVM and a domU which requires network access.

**FirewallVM**
Firewall Virtual Machine. A type of ProxyVM which is used to enforce network-level policies (a.k.a. "firewall rules"). A FirewallVM called `firewallvm` is created by default in most Qubes installations.

**DispVM**
Disposable Virtual Machine. A temporary AppVM which can quickly be created, used, and destroyed.

**PV**
Paravirtualization. An efficient and lightweight virtualization technique originally introduced by the Xen Project and later adopted by other virtualization platforms. Unlike HVMs, paravirtualized VMs do not require virtualization extensions from the host CPU. However, paravirtualized VMs require a PV-enabled kernel and PV drivers, so the guests are aware of the hypervisor and can run efficiently without emulation or virtual emulated hardware.

**HVM**
Hardware Virtual Machine. Any fully virtualized, or hardware-assisted, VM utilizing the virtualization extensions of the host CPU. Although HVMs are typically slower than paravirtualized VMs due to the required emulation, HVMs allow the user to create domains based on any operating system.

**StandaloneHVM**
Any HVM which is standalone (i.e., does not depend on any other VM for its root filesystem). In Qubes, StandaloneHVMs are referred to simply as **HVMs**.

**TemplateHVM**
Any HVM which functions as a TemplateVM by supplying its root filesystem to other VMs. In Qubes, TemplateHVMs are referred to as **HVM templates**.

**PVH**


