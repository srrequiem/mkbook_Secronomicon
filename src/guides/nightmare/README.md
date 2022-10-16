# Notas de Nightmare

[https://guyinatuxedo.github.io/](https://guyinatuxedo.github.io/)

# 1 Introduction

## 1.1 Assembly

### Registers

- rbp: Base Pointer, points to the bottom of the current stack frame
- rsp: Stack Pointer, points to the top of the current stack frame
- rip: Instruction Pointer, points to the instruction to be executed

**General Purpose Registers**
These can be used for a variety of different things

- rax
- rbx
- rcx
- rdx
- rsi
- rdi
- r8
- r9
- r10
- r11
- r12
- r13
- r14
- r15

In `x64` linux arguments to a function are passed via registers. The first few args are passed by these registers:

- rdi:    First Argument
- rsi:    Second Argument
- rdx:    Third Argument
- rcx:    Fourth Argument
- r8:     Fifth Argument
- r9:     Sixth Argument

| 8 Byte Register | Lower 4 Bytes | Lower 2 Bytes | Lower Byte |
|---|---|---|---|
| rbp | ebp | bp | bpl |
| rsp | esp | sp | spl |
| rip | eip |  |  |
| rax | eax | ax | al |
| rbx | ebx | bx | bl |
| rcx | ecx | cx | cl |
| rdx | edx | dx | dl |
| rsi | esi | si | sil |
| rdi | edi | di | dil |
| r8 | r8d | r8w | r8b |
| r9 | r9d | r9w | r9b |
| r10 | r10d | r10w | r10b |
| r11 | r11d | r11w | r11b |
| r12 | r12d | r12w | r12b |
| r13 | r13d | r13w | r13b |
| r14 | r14d | r14w | r14b |
| r15 | r15d | r15w | r15b |

In `x64` we will see the `8` byte registers. However in `x86` the largest sized registers we can use are the `4` byte registers like `ebp`, `esp`, `eip` etc. Now we can also use smaller registers, than the maximum sized registers for the architecture.

In `x64` there is the `rax`, `eax`, `ax`, and `al` register. The `rax` register points to the full `8`. The `eax` register is just the lower four bytes of the `rax` register. The `ax` register is the last `2` bytes of the `rax` register. Lastly the `al` register is the last byte of the `rax` register.

## 1.4 Debugginng with DBG

- `nexti` - will have you go instruction by intruction through the program, but will not step into function calls such as puts.
- `next` - which will take you through one line of code, but will step over function calls such as puts.
- `step` - which will take you through one line of code, but will step into function calls
- `stepi` - whch will take you through one instruction at a time, stepping into function calls.
- `x` - examine things.
- `x/a` - as an address.
- `x/10c` - as a number of characters.
- `x/s` - as a string.
- `x/g` - as a qword.
- `x/w` - as a dword.

## 1.5 Scripting with Python pwntools

- `p64(x)` - pack the integer as a least endian QWORD.
- `p32(x)` - pack the integer as a least endian DWORD.
- `u64(x)` - unpack a least endian QWORD and get it's integer value.
- `u32(x)` - unpack a least endian DWORD and get it's integer value.

# Cheatsheet

Visualizar ensamblador paginado:

```bash
objdump -D <binary> -M intel | less
```

## GDB

```bash
disassemble
disass
b *main+25
b *0x08048414
b *puts
info breakpoints
info b
i b
info registers
info frame
delete <breakpoint number>
del 2
d 2
x/a 0x08048451
x/10c 0x08048451
x/s 0x08048451
x/g 0x08048451
x/w 0x08048451
jump *0x08048451
j *0x08048451
```

## Exploit template

```python
from pwn import *

target = remote("github.com", 9000)
target = process("./challenge")
gdb.attach(target)
# gdb.attach(target, gdbscript='b *main')
target.send(x)
target.sendline(x)
print target.recvline()
print target.recvuntil("out")
target.interactive()
```

# References

- [Nightmare](https://guyinatuxedo.github.io/).
- [Pwn College](https://pwn.college/).
- [Exploit Education](https://exploit.education/).
- https://github.com/hugsy/gef
- https://github.com/pwndbg/pwndbg
- https://infosecwriteups.com/pwndbg-gef-peda-one-for-all-and-all-for-one-714d71bf36b8
- 