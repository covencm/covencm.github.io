---
title: "Stack Writeup"
author: coven_cm
categories: [os, linux, nvidia ]
draft: false
tags: ["linux","nvidia"]
date: 2025-08-28T11:34:14+05:30
---



## The Stack
This challenge introduces the stack as a source of data that is already available when a program begins execution.
On x86-64 Linux, the register rsp points to the top of the stack. At program startup, the value stored at [rsp] is argc, the number of command-line arguments passed to the program, including the program name itself.

For example, if the program is run with:

```bash ./program hello world```

then argc is 3:

```
./program
hello
world
```

The goal of the challenge is to read that value from the stack and use it as the program’s exit code.

Solution:


```
.intel_syntax noprefix
.section .text
.global _start

_start:
    mov rdi, [rsp]
    mov rax, 60
    syscall 
```

Explanation
The key instruction is:

``` mov rdi, [rsp] ```

This reads the 64-bit value stored at the memory location pointed to by rsp. At startup, that value is argc.

Next:

``` mov rax, 60 ```

sets rax to 60, which is the syscall number for exit on x86-64 Linux.

Finally:

``` syscall ```

invokes the system call. Since Linux expects the exit status in rdi, the program exits with the value of argc.

## Stack Offsets
Find the secret value on the stack at an offset of 128 bytes from rsp and use that value as the program’s exit code.

rsp points to the top of the stack.
Since stack values here are treated as 8-byte values, an offset of 128 bytes means:

``` 128 / 8 = 16 ```
so the secret is effectively the 16th 8-byte slot above rsp

The challenge wants us to:

load the value at [rsp+128]
place it in the correct register for exit
invoke the exit syscall

Solution:

```
.intel_syntax noprefix
.section .text
.global _start

_start:
    mov rdi, [rsp+128]
    mov rax, 60
    syscall
mov rdi, [rsp+128]
```
Loads the 8-byte value located 128 bytes above the current stack pointer into rdi.

```mov rax, 60```

```syscall```

Calls exit(rdi), so the program exits with the secret value.

## Program Arguments on the Stack

The goal of this challenge was to read the first program argument from the stack and then exit with that value.

At program start on Linux x86-64:

[rsp] contains argc
[rsp+8] contains a pointer to argv[0]
[rsp+16] contains a pointer to argv[1]
The important detail is that argv[1] is not stored directly at [rsp+16].
Instead, [rsp+16] holds an address pointing to the actual argument string.

So I needed to Read the pointer to argv[1] from the stack, Dereference that pointer to access the argument data and Exit using that value

Solution:

```
.intel_syntax noprefix
.section .text
.global _start

_start:
    mov rdi, [rsp+16]
    mov rdi, [rdi]
    mov rax, 60
    syscall
Load argv[1] pointer

mov rdi, [rsp+16]
```

This loads the pointer to the first argument (argv[1]) into rdi.

2. Dereference the pointer

```mov rdi, [rdi]```

This reads the data stored at that address. Since the argument is a string, this grabs the first 8 bytes starting from that string.

For this challenge, that works because the program only needs to exit with the first byte’s value.

3. Call exit

```
mov rax, 60
syscall
```

rdi is used as the exit status.

## Popping From the Stack

This challenge was about using the stack the way it is meant to be used with pop and Read the argument count (argc) from the stack and exit with that value.

Solution:
```
.intel_syntax noprefix
.section .text
.global _start

_start:
        pop rdi
        mov rax, 60
        syscall
```
When a program starts, the stack begins with argc at the top of the stack, pointed to by rsp.

So at _start:

[rsp] contains argc
pop rdi:
    copies the value at [rsp] into rdi
    increments rsp by 8
After that, rdi holds the argument count.

Then:

mov rax, 60 sets up the exit syscall number on x86_64 Linux
syscall calls exit(rdi)

Since the exit status is taken from rdi, the program exits with the value of argc.

Also The challenge specifically wanted pop, not just mov rdi, [rsp].

The difference is:

mov rdi, [rsp] reads the top of the stack
pop rdi reads it and advances the stack pointer
That matches the intended “stack” behavior.

The checker ran the program with different numbers of arguments, and the exit code matched the total argument count each time:

```
pop.elf arg0 arg1 → exit code 3
pop.elf arg0 ... arg7 → exit code 9
pop.elf arg0 ... arg8 → exit code 10
```
That confirms the program correctly popped argc and used it as the exit status.
