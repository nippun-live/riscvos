# Custom UNIX-Style RISC-V Operating System

Developed a UNIX-style operating system kernel and a robust journaling filesystem from scratch for the RISC-V architecture. This project demonstrates deep, practical skills in low-level systems programming and OS design.

## 📝 Important Note on Source Code

The source code for this project is not included in this public repository. This is to maintain academic integrity as the project was developed as part of a university curriculum. Recruiters or other interested parties are welcome to contact me directly to request a private viewing of the source code.

## 🚀 Running the OS

This project can be run on any Linux machine with the required prerequisites installed.

### Prerequisites

You will need the following tools installed on your system:

- **QEMU**: The system emulator. Specifically, you need `qemu-system-riscv64`.
- **screen**: A terminal multiplexer used to connect to the OS shell.

You can typically install these on a Debian/Ubuntu system with:

```bash
sudo apt-get update
sudo apt-get install qemu-system-riscv64 screen
````

### Instructions

Follow these steps to boot the operating system.

#### 1. Clone the Repository

```bash
git https://github.com/nippun-live/riscvos.git
cd riscvos
```

#### 2. Launch the VM

This command starts the QEMU virtual machine in the background. It will create a dedicated pseudo-terminal (pty) for the OS shell.

```bash
qemu-system-riscv64 \
-global virtio-mmio.force-legacy=false \
-machine virt -bios none -nographic \
-object rng-random,filename=/dev/urandom,id=rng0 \
-device virtio-rng-device,rng=rng0 \
-drive file=ktfs.raw,id=blk0,if=none,format=raw,readonly=false \
-device virtio-blk-device,drive=blk0 \
-serial mon:stdio -serial pty \
-m 128M -kernel ./kernel.elf &
```

After running, QEMU will print the name of the character device it created, for example: `char device redirected to /dev/pts/0`. Take note of this device name.

#### 3. Connect to the Shell

In a separate terminal window, use the `screen` command to connect to the device name from the previous step.

```bash
# Replace /dev/pts/0 with the device name you noted
screen /dev/pts/0
```

You should now have a direct, interactive terminal session with the running operating system.


## ✨ Technical Highlights

This operating system implements several key features from the ground up, providing a practical understanding of OS design and implementation.

### UNIX-style Kernel in C

Implemented core OS functionalities including a bootloader, trap handling, Sv39 virtual memory with demand paging, and process abstraction. Supports essential user-mode syscalls (`open`, `close`, `read`, `write`, `ioctl`, `exec`, `fork`, `wait`, `usleep`, `fscreate`, `fsdelete`). Created cooperative and preemptive threading models using condition variables and timer (`mtime`) interrupts.

### Custom Filesystem & Efficient Block Cache

Engineered a block-based filesystem with a write-ahead journal for metadata consistency and robust crash recovery. Features support for create, read, write, delete, and flush operations with multi-level indirection. It is mountable via a VirtIO block device. An implemented write-back cache with configurable associativity significantly reduces I/O latency for both filesystem and block operations while ensuring data consistency through the journaling mechanism.

### Preemptive Multitasking

The kernel supports preemptive multitasking, allowing multiple threads to run concurrently.

- A simple, preemptive scheduler manages context switching between threads.
- Low-level context switching is handled in RISC-V assembly.
- The system uses timer interrupts to preempt running threads.
- Synchronization primitives, such as condition variables and locks, are provided for thread synchronization.

### Memory Management

A virtual memory system is in place, with both physical and virtual memory management.

- The system uses Sv39 paging to provide each process with its own virtual address space via a three-level page table structure.
- A buddy system-like allocator manages physical memory.
- A simple heap allocator is implemented for dynamic memory allocation within the kernel.

### Device Drivers & MMIO

Developed drivers for UART (polling & interrupt-driven), Real-Time Clock (RTC), Platform-Level Interrupt Controller (PLIC), VirtIO block & RNG devices (with custom ISR integration), and GPIO/SPI interfaces for embedded peripherals.

### Games & Application Support

Built a unified I/O interface to load and execute ELF binaries from the filesystem. This has been validated by running classic games like Star Trek, Doom, Rogue, and Zork on the QEMU RISC-V target, with automated tests for correct loading, execution, and system-call handling.


