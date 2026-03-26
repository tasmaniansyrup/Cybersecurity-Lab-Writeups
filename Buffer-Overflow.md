# Buffer Overflow Exploitation
**Course:** CS 456 | **Date:** October 23, 2025

---

## Skills Demonstrated
- Stack-based buffer overflow exploitation
- Memory layout analysis and register manipulation
- Shellcode injection and NOP sled construction
- Vulnerability identification and secure code remediation

---

## Overview
This lab involved finding, exploiting, and remediating a stack-based buffer 
overflow vulnerability in a deliberately vulnerable C program. Starting from 
scratch, I identified the exact memory layout of the program's stack, crafted 
a working exploit that hijacked execution flow, and delivered a shellcode 
payload to obtain a command shell. The final step involved rewriting the 
vulnerable code to eliminate the vulnerability entirely.

---

## Exploit Development Process

### Step 1: Determining the EIP Offset
The first step was determining how many characters were required to fully 
overwrite the EIP register. Through testing, 512 characters were needed to 
fully overwrite EIP. Notably, a segmentation fault can be triggered with as 
few as 509 characters, which marks the beginning of the EIP register — 
confirming the exact boundary between the buffer and the register space.

### Step 2: Mapping the Memory Layout
With the offset established, each register (EBX, EBP, and EIP) was overwritten 
with a distinct character pattern to confirm precise control over each one 
individually. This step validated a complete understanding of the stack layout 
and confirmed that the overflow could be controlled with surgical precision.

### Step 3: Injecting the Payload
A test script (`exploit-test.py`) was used to verify that a crafted payload 
could be successfully injected into the program's memory buffer. Printing the 
buffer contents confirmed the payload landed exactly where expected.

### Step 4: Finalizing the Exploit
The final exploit script was constructed with three components: shellcode, a 
NOP sled, and the overwritten return address pointing into the NOP sled region. 
The specific memory address used was chosen to land within the NOP sled, 
allowing the CPU to slide down to the shellcode without requiring knowledge of 
its exact location.
```python
import sys

shellcode = b"\x31\xc0\x50\x68\x6e\x2f\x73\x68\x68\x2f\x2f\x62\x69\x6e\x50\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80"
eip = b"\xc8\xce\xff\xff" * 10
nop = b"\x90" * 447
buff = nop + shellcode + eip

sys.stdout.buffer.write(buff)
```

Running the finalized script caused the program to slide down the NOP sled, 
execute the shellcode, and return a command shell — confirming a fully 
successful exploit.

---

## Questions

### The Role of Security Protections
This lab required disabling several modern security protections via gcc flags, 
including `-fno-stack-protector` and `-z execstack`. The `-z execstack` flag 
enables execution of code on the stack by disabling the NX (no-execute) bit. 
Normally, instructions on the stack are not executable — this exists precisely 
to prevent attackers from writing malicious shellcode onto the stack and 
redirecting execution to it. This is exactly what was done in this lab. Without 
this flag disabled, the injected shellcode would have been blocked from 
executing entirely, and the exploit would have failed.

### The NOP Sled
The NOP sled solves the problem of precision when overwriting the return 
address. Due to natural variations in memory layout, pinpointing the exact 
address of injected shellcode is unreliable. By prepending a large sequence of 
NOP (no-operation) instructions before the shellcode, any return address that 
lands anywhere within that region will cause the CPU to slide instruction by 
instruction until it reaches and executes the shellcode.

A disadvantage of using a very large NOP sled is that it becomes detectable by 
intrusion detection systems (IDS). A long sequence of `0x90` bytes is a 
well-known exploit signature that many IDS tools actively scan for, meaning a 
large NOP sled significantly increases the likelihood of the attack being flagged 
and blocked before it succeeds.

### Secure Coding: Remediating the Vulnerability
The vulnerability in `vuln.c` stemmed from the use of `strcpy()`, which copies 
characters into a buffer with no limit on length — allowing an attacker to write 
beyond the buffer's bounds and overwrite adjacent memory. The fix is to replace 
it with `strncpy()`, which accepts a third parameter specifying the maximum 
number of characters to copy. Using `sizeof(buffer) - 1` as the limit ensures 
the copy can never exceed the buffer's capacity, with the `-1` reserving space 
for the null terminator.
```c
#include <stdio.h>
#include <string.h>

int main(int argc, char** argv) {
    char buffer[500];
    strncpy(buffer, argv[1], sizeof(buffer) - 1);
    return 0;
}
```

This single change eliminates the buffer overflow vulnerability by enforcing a 
hard boundary on how much data can be written into the buffer.
