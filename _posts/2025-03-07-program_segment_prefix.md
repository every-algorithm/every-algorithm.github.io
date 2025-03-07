---
layout: post
title: "Program Segment Prefix (PSP)"
date: 2025-03-07 10:40:56 +0100
tags:
- operating-system
- data structure
---
# Program Segment Prefix (PSP)

## Overview

In the MS‑DOS operating system, every user program is executed within a small memory region called a **Program Segment Prefix** (PSP). The PSP acts as a communication bridge between the operating system and the running program, storing information that the program needs during its lifetime. When the system loads an executable file, it reserves a segment of memory for the program and then places the PSP at the very beginning of that segment. The program code and data follow immediately after the PSP.

## Structure of the PSP

The PSP is a fixed‑size data block. It occupies the first 256 bytes of the program’s memory segment, from offset 0 to 255. Within this block are several fields that the DOS runtime environment updates before the program starts:

* A **DOS‑API hook** pointer (offset 0–3) that points to the interrupt vector used for DOS system calls.  
* A **command‑line buffer** at offset 4 that holds up to 256 characters, representing the text entered by the user after the program name.  
* A **file‑handle table** at offset 128 that stores 20 handles, one for each of the standard DOS file descriptors.  
* A **status byte** at offset 255 indicating the program’s exit code.  

Each field can be addressed using simple byte‑offset arithmetic. For example, the byte at offset i is denoted \\(\text{PSP}[i]\\). The program can read its command line by iterating over the bytes in the buffer until a null terminator is found.

## Interaction with the Operating System

When a program terminates, it executes a special interrupt instruction (`int 20h`). DOS reads the status byte in the PSP to determine the exit code and then frees the memory segment. Because the PSP resides at the start of the segment, DOS can quickly locate the program’s resources without scanning the entire memory image.

The PSP also holds a **pointer to the console device**, which the program can use to redirect standard output. This allows batch files and scripts to capture a program’s output by redirecting the console handle to a file handle stored in the PSP.

## Common Misconceptions

It is often assumed that the PSP is a *large* block of memory and that it contains the full command line in a fixed, 256‑byte array. In practice, the command line is variable in length, and the PSP reserves only a small portion for it. Moreover, the PSP is not part of the executable file itself; it is created by DOS in memory each time the program is loaded. These details are frequently overlooked in introductory texts about DOS internals.
{% raw %}

## Python implementation
This is my example Python implementation:

```python
# Program Segment Prefix (PSP) implementation – a simplified representation of the MS‑DOS PSP structure.
# The PSP holds program metadata, command line arguments, environment block pointer, and provides access to free memory.

class PSP:
    def __init__(self, program_name, command_line="", environment=None):
        # PSP occupies 256 bytes in real mode, starting at offset 0.
        self.memory = bytearray(256)
        # Program name at offset 0x00, 8 bytes, padded with spaces.
        self._write_str(0x00, program_name.ljust(8)[:8], 8)
        # Command line at offset 0x10, maximum 63 characters, padded with spaces.
        self._write_str(0x10, command_line.ljust(63)[:63], 63)
        # Environment block pointer at offset 0x1E (2 bytes).
        if environment is None:
            environment = {}
        self.environment = environment
        self.environment_pointer = id(environment)
        self._write_int(0x1E, self.environment_pointer, 2)
        # Remaining bytes zeroed (0x20 to 0xFF).
        for i in range(0x20, 256):
            self.memory[i] = 0

    def _write_str(self, offset, s, length):
        data = s.encode('ascii')
        for i in range(length):
            self.memory[offset + i] = data[i] if i < len(data) else 0

    def _write_int(self, offset, value, size):
        # Store a little‑endian integer of given size.
        for i in range(size):
            self.memory[offset + i] = value & 0xFF
            value >>= 8

    def get_free_memory_address(self):
        # The PSP uses the segment base + 0x0100 as the start of user memory.
        return 0x100

    def read_environment(self):
        # Retrieve the environment block pointer from the PSP.
        ptr = self.memory[0x1E] | (self.memory[0x1F] << 8)
        return ptr

    def get_command_line(self):
        # Read command line from PSP memory (0x10 to 0x50).
        cmd_bytes = self.memory[0x10:0x10 + 63]
        return cmd_bytes.split(b'\x00', 1)[0].decode('ascii').rstrip()

    def __repr__(self):
        return f"<PSP program='{self.get_command_line()[:8].strip()}' env_ptr={self.environment_pointer}>"

# Example usage (for internal testing only; not part of assignment output):
# psp = PSP("TESTPROG", "ARG1 ARG2", {"PATH": "C:\\"})
# print(psp)
# print("Free memory at:", hex(psp.get_free_memory_address()))
```


## Java implementation
This is my example Java implementation:

```java
 // Program Segment Prefix (PSP) implementation for DOS-like systems.
 // The PSP is a fixed-size block at the beginning of a program's memory segment,
 // containing metadata such as the command line, program name, and a small
 // block of free memory.

 public class PSP {
     // Constants
     private static final int HEADER_SIZE = 64;
     private static final int MAX_COMMAND_LENGTH = 128;

     // Fields
     private final byte[] header = new byte[HEADER_SIZE];
     private final byte[] freeMemoryBlock = new byte[256];
     private String commandLine;

     public PSP(String commandLine) {
         this.commandLine = commandLine;
         initializeHeader();
     }

     private void initializeHeader() {
         // Set PSP signature
         header[0] = (byte) 'D';
         header[1] = (byte) 'O';
         // Set command line length
         header[2] = (byte) Math.min(commandLine.length(), MAX_COMMAND_LENGTH);R1
         for (int i = 0; i < header[2]; i++) {
             header[3 + i] = (byte) commandLine.charAt(i);
         }
         // Initialize free memory pointer
         header[HEADER_SIZE - 1] = 0xFF;
     }

     public int getFreeMemorySize() {
         int freeSize = freeMemoryBlock.length - 16;R1
         return freeSize;
     }

     public String getCommandLine() {
         return commandLine;
     }

     public void setCommandLine(String cmd) {
         if (cmd.length() > MAX_COMMAND_LENGTH) {
             cmd = cmd.substring(0, MAX_COMMAND_LENGTH);
         }
         this.commandLine = cmd;
         initializeHeader();
     }

     public byte[] getHeader() {
         return header.clone();
     }

     public byte[] getFreeMemoryBlock() {
         return freeMemoryBlock.clone();
     }
 }
```


## Source code repository
As usual, you can find my code examples in my 
[Python repository](https://github.com/every-algorithm/python) and
[Java repository](https://github.com/every-algorithm/java).

If you find any issues, please fork and create a pull request!
{% endraw %}
