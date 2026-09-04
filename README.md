# minix3-tux35

This is a personal sandbox project tracking my exploration back into operating system fundamentals and microkernel design. 

I live in Linux every day for work, but MINIX was actually the very first OS I used back at university to see how things worked under the bonnet. With Linux turning 35, I thought it would be a fun excuse to dust off MINIX again, see what I actually remember, break a few things on purpose, and get back in touch with the basic primitives I haven't touched in years.


## A Few Minor Caveats

Before diving into the roadmap, it is perhaps worth noting a couple of practical constraints regarding this workspace:

**MINIX  is definitely showing its age.**
Honestly, that’s half the fun. Wrangling a vintage operating system and its cross-compiler to play nice with a modern Linux machine is part of the challenge. The nice thing is that its tiny codebase makes it incredibly easy to open up files, read through the source code, and trace exactly what's going on.

**AI is pretty terrible at this.**
I’ve used them here and there to bounce architectural ideas around or locate odd bits of the source tree, but old C code, microkernels, and legacy build system flags are very good at exposing confident nonsense. Most of the actual progress still comes down to reading the source files, fixing compiler errors manually, and trying things until they boot.

## Sandbox Structural Roadmap (Hobby Progress Logs)

There isn't a strict order to this repository. I'm building things out organically and jumping around depending on what subsystem catches my eye in my free time:

- **setup**

  Get the environment and toolchain set up for cross-compiling MINIX, then make a simple kernel modification to change the startup banner.

  <a href="setup">source</a>


- **syscalls**

  Play around with system calls, try to modify an existing one and create a custom new system call.

- **device driver**

  Build a small device driver from scratch and see how MINIX moves data between isolated processes.

- **ipc-pipes**

  See how MINIX manages data stream buffers when two processes talk using standard MINIX pipes.

- **memory-malloc**

  Play with the classic C `malloc` call and see how memory is organised from RAM to virtual memory. Try to implement a version that can handle page faults.

- **memory-encryption**

  Play with page tables and try encrypting user pages right before they hit swap so the raw data on disk is obfuscated.