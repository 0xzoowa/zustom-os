# Booting the Operating System: A Learning Journey

## Motivation

The operating system has always fascinated me, the idea that one thing
controls every other thing. That concept has been captivating since I
first encountered it.

Over the past year, I knew I wanted to understand OS development, but
the concepts felt dauntingly complex. Rather than dive in unprepared, I
took a deliberate approach: I built foundational knowledge through
lower-level tutorials like Dr. Jonas Birch's courses and projects such
as nand2tetris from the Hebrew University of Jerusalem.

These resources gave me a solid grounding in computer architecture and
low-level systems. They were my training wheels. Eventually, though,
they became unexciting. I recognize now that this lack of excitement was
my brain's way of signaling readiness for the real thing. Those
foundational blocks deserve credit, they built mental models of the
landscape I was exploring, making the terrain feel familiar rather than
frightening when I finally started real OS development.

I chose the OSDev Wiki because it offered structure without revealing
solutions, a balance of struggle and guidance. I needed exactly that.
While it took time to reach this point, I now feel confident in my
ability to explore this terrain. Not because I know everything required,
but because through years as a professional software engineer and
countless personal explorations, I've observed my ability to figure
things out, from the simplest to the most complex. It might take time,
struggle, and frustration, but I always do figure it out.

---

## The First Obstacle in the OSDev Wiki

### Prerequisites

Before attempting to boot a simple kernel, several tools and
technologies are required:

- An assembler (e.g., NASM) to write low-level startup code.
- A cross-compiler (e.g., GCC targeting `i686-elf`) to compile kernel
  C code without leaking host-specific details.
- A linker (LD) and a custom linker script to control memory layout.
- A bootloader (e.g., GRUB) compliant with the Multiboot
  specification.
- An emulator or virtual machine (e.g., QEMU) to test the OS safely.
- A basic understanding of x86 architecture, memory layout, and the
  boot process.

#### A Quick Detour

My learning style has always been different from those around me. I
sometimes joke that I have the memory of a goldfish if there isn't depth
or connection to what I'm learning. Surface-level skimming, cramming, or
merely "using the tool" doesn't work for me. I need to break things down
to their most basic form and build from there. That's the only way I
retain information. It may be slow, but I prefer it because I've always
prioritized depth.

#### Ok, Back on Track

The first milestone in the OSDev Wiki journey was to boot a relatively
simple kernel that outputs:

> "Hello, Kernel World!"

to the screen.

This is done using VGA (Video Graphics Array) text mode. VGA is a
hardware interface that allows software to write characters directly to
a specific region of memory (commonly at address `0xB8000` in text
mode). Each character cell consists of two bytes: one for the ASCII
character and one for its color attributes. Writing directly to this
memory buffer causes text to appear on the screen without any operating
system support, just raw hardware interaction.

---

## The Boot Process (In Logical Order)

### 1. Power On

When the computer is turned on, the CPU begins execution at a predefined
memory address. At this stage, there is no operating system, just
firmware.

### 2. POST (Power-On Self-Test)

The firmware performs a series of diagnostic checks known as POST. It
verifies that essential hardware components---RAM, CPU, keyboard, and
storage---are functional.

### 3. BIOS Initialization

On legacy systems, the firmware is the BIOS (Basic Input/Output System).
After POST, the BIOS searches for bootable devices in a predefined order
(hard drive, CD-ROM, USB, etc.).

### 4. Locating a Bootable Device

For a hard drive to be considered bootable, its first sector (512 bytes)
must end with a specific "magic number":

- Byte 510: `0x55`
- Byte 511: `0xAA`

If this signature is present, the BIOS considers the device bootable. If
no bootable device is found, an error is displayed.

### 5. Loading the Master Boot Record (MBR)

If a valid bootable device is found, the BIOS loads its first 512-byte
sector---the Master Boot Record (MBR)---into memory at address `0x7C00`
and transfers control to it.

At this point, the bootloader begins executing.

### 6. Bootloader and Multiboot

In my setup, I use a Multiboot-compliant bootloader (e.g., GRUB).

The bootloader searches the kernel binary for a Multiboot header within
the first 8 KB of the file, as required by the specification. It looks
for a magic value `0x1BADB002`, checks the flags, verifies the checksum, and ensures
the kernel is valid.

If everything checks out, the bootloader loads the kernel into memory
and jumps to its entry point.

### 7. Kernel Entry (`_start`)

The entry point in my case is the `_start` function written in NASM
assembly.

This function performs minimal setup---such as configuring the
stack---and then calls a C function defined in my kernel source.

### 8. Writing to VGA Memory

The C function writes directly to the VGA text buffer at `0xB8000`. Each
character of "Hello, Kernel World!" is written into memory along with
its color attribute.

The screen updates immediately because VGA hardware continuously reads
from that memory region to render text.

What appears to be a simple string output is actually the culmination of
firmware execution, bootloader validation, memory setup, CPU mode
transitions, and direct hardware interaction.

For someone who had previously been oblivious to this entire process, it
feels like stepping into a hidden world, one filled with wonder and
excitement.

There's much more to explore, and I intend to document my journey. I've
historically been bad at documentation, but I'm realizing now that it's
not optional, it's necessary.

---

## Testing: Using a Virtual Machine

My development machine runs macOS. I use VirtualBox to run Ubuntu as a guest operating system,
which serves as my development environment. Inside Ubuntu, I compile my kernel using a
cross-compiler toolchain and test it using QEMU.
QEMU boots the generated ISO image and emulates a bare-metal x86 machine,
allowing me to test the kernel in isolation without touching real hardware or affecting my development environment.

The process looks like this:

1.  Compile the kernel using a cross-compiler.
2.  Link it with a custom linker script to control memory layout.
3.  Package the kernel into a bootable ISO image using GRUB.
4.  Launch the ISO image.

For someone who values deep exploration and careful understanding, this
environment is perfect. It allows me to slow down, observe, and truly
understand what is happening beneath the surface.

---

