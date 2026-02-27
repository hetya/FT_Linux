# A Custom Gnu/Linux OS

**FT_linux** is a custom GNU/Linux operating system built entirely from scratch using [LFS 12.4](https://www.linuxfromscratch.org/lfs/view/12.4/index.html). This is a deep dive in understanding of how a Linux system is constructed — from toolchain compilation to compiling a base system in a controlled environment, kernel configuration and creating a bootable operating system.

Rather than relying on a prebuilt distribution, every component is compiled and configured manually to provide a complete understanding of the GNU/Linux stack.
---

## Built With

- GNU toolchain (GCC, Binutils, Glibc)
- Linux Kernel
- Core GNU utilities
- Some other convenient addons like openssh


This is my config with tools I think where usefull, here all packages are built from source and managed manually — no external package manager is used.
Every configuration choice reflects a deliberate design decision focused on minimalism, control, and learning.
