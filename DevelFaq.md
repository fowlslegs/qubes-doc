---
layout: doc
title: DevelFaq
permalink: /doc/DevelFaq/
redirect_from: /wiki/DevelFaq/
---

Qubes Developers FAQ
====================

1.  1.  [Q: Why does dom0 need to be 64-bit?](#q-why-does-dom0-need-to-be-64-bit)
    2.  [Q: Why do you use KDE in Dom0? What is the roadmap for Gnome support?](#q-why-do-you-use-kde-in-dom0-what-is-the-roadmap-for-gnome-support)
    3.  [Q: What is the recommended build environment?](#q-what-is-the-recommended-build-environment)
    4.  [Q: How to build Qubes from sources?](#q-how-to-build-qubes-from-sources)
    5.  [Q: How do I submit a patch?](#q-how-do-i-submit-a-patch)
    6.  [Q: Why was the storage domain never implemented?](#q-why-was-the-storage-domain-never-implemented)

### Q: Why does dom0 need to be 64-bit?

Often it is more difficult to exploit a bug on the x64 Linux than it is on x86 Linux (e.g. ASLR is sometimes harder to get around). While we designed Qubes with the emphasis on limiting any potential attack vectors in the first place, still we realize that some of the code running in Dom0, e.g. our GUI daemon or xen-store daemon, even though it is very simple code, might contain some bugs. Plus currently we haven't implemented a separate storage domain, so also the disk backends are in Dom0 and are "reachable" from the VMs, which adds up to the potential attack surface. So, having faced a choice between 32-bit and 64-bit OS for Dom0, it was almost a no-brainer, as the 64-bit option provides some (little perhaps, but still) more protection against some classes of attacks, and at the same time do not have any disadvantages (except that it requires a 64-bit processor, but all systems on which it makes sense to run Qubes, e.g. that have at least 3-4GB memory, they do have 64-bit CPUs anyway).

### Q: Why do you use KDE in Dom0? What is the roadmap for Gnome support?

There a few things that are KDE-specific, but generally it should not be a big problem to also add Gnome support to Qubes (in Dom0 of course). Those KDE-specific things are:

-   Qubes requires KDM (KDE Login Manager), rather than GDM, for the very simple reason that GDM doesn't obey standards and start `/usr/bin/Xorg` instead of `/usr/bin/X`. This is important for Qubes, because we need to load a special "X wrapper" (to make it possible to use Linux usermode shared memory to access Xen shared memory pages in our App Viewers -- see the sources [here](https://github.com/QubesOS/qubes-gui-daemon/tree/master/shmoverride)). So, Qubes makes the `/usr/bin/X` to be a symlink to the Qubes X Wrapper, which, in turn, executes the `/usr/bin/Xorg`. This works well with KDM (and would probably also work with other X login managers), but not with GDM. If somebody succeeded in makeing GDM to execute `/usr/bin/X` instead of `/usr/bin/Xorg`, we would love to hear about it!

-   We maintain a special [repository](/doc/KdeDom0/) for building packages specifically for Qubes Dom0.

-   We've patched the KDE's Window Manager (specifically [one of the decoration plugins](https://github.com/QubesOS/qubes-desktop-linux-kde/tree/master/plastik-for-qubes)) to draw window decorations in the color of the specific AppVM's label.

If you're interested in porting GNOME for Qubes Dom0 use, let us know -- we will most likely welcome patches in this area.

### Q: What is the recommended build environment?

Any rpm-based, 64-bit. Preferred Fedora.

### Q: How to build Qubes from sources?

See [the instruction](/doc/QubesBuilder/)

### Q: How do I submit a patch?

1.  Make all the changes in your working directory, i.e. edit files, move them around (you can use 'git mv' for this), etc.

1.  Add the changes and commit them (git add, git commit). Never mix different changes into one commit! Write a good description of the commit. The first line should contain a short summary, and then, if you feel like more explanations are needed, enter an empty new line, and then start the long, detailed description (optional).

1.  Test your changes NOW: check if RPMs build fine, etc.

1.  Create the patch using 'git format-patch'. This has an advantage over 'git diff', because the former will also include your commit message, your name and email, so that \*your\* name will be used as a commit's author.

1.  Send your patch to qubes-devel. Start the message subject with the '[PATCH]' string.

### Q: Why was the storage domain never implemented?

If you read the [original architecture document](http://files.qubes-os.org/files/doc/arch-spec-0.3.pdf) you would have read extensive exposition on an early idea for a driver domain which would access the disk device driver via Intel VT-d. If you also are running Qubes you may have noticed it's not there. Implementing the storage domain "assumes that we leave no disk controller in Dom0 (because in 99.99% of cases there is only one disk controller on desktop systems). This, in turn, implies that both system’s /boot as well as the whole Dom0’s filesystems are served by the *untrusted* storage domain.  And in order to make this domain really untrusted we need to use smart crypto tricks and trusted boot, as described in the Qubes Architecture Specification. This all would require some complex changes to Qubes core."

"Additionally this would decrease hardware compatibility, as at the very least we would need to require TPM, preferably TPM + TXT, and we would need to set up a trusted boot during installation (which would requite asking user e.g. to enable and take ownership of a TPM), which would, in turn, make the installation procedure much less user friendly.  Taking this all into account, and also the fact that the disk backend that we have in Dom0 doesn’t seem like a likely attack vector, after all, we concluded not to implement it, at least not anytime soon”
  --Joanna Rutkowska---see full discussion [here](https://groups.google.com/forum/#!searchin/qubes-devel/storage$20domain/qubes-devel/B1mAjAAl6G4/vpt4zSAXbu0J).
