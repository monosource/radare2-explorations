# Tutorial 3 - ESIL

This section will probably be confusing at first, but I will try to make it as simple and as practical as possible. Afterward, you can probably go and read the ESIL [section](https://radare.gitbooks.io/radare2book/content/esil.html) in the radare2 book and read pancake's [presentation](http://rada.re/get/lacon2k15-esil.pdf).

ESIL is an intermediate language based on evaluable strings, with a Polish-like order of evaluation; it is a representation of various architecture-specific instructions in a more general, simplified form. ESIL can also be viewed as a virtual machine with its own stack, registers and instruction set.

ESIL can be a common ground between ARM, x86, MIPS and all other architectures supported by radare2.

## What is the purpose of ESIL?

Having a controlled environment is crucial when dealing with, say, live malware. Sometimes, setting up such an environment can lead to risks of its own.

Some architectures are quite obscure and inaccessible, and you have to reverse engineer a binary the hard way, by studying opcodes and trying to understand what the program does.

A solution to these problems (and many others) lies in emulation. Since ESIL is a translation of various instructions from different architectures, it can be used for the purpose of emulating non-native, or native but dangerous code.

ESIL can also be used to study an architecture by examining the effects different instructions have on registers, stack and memory.

## A few examples

So how does ESIL look like?

`mov ecx, ebx` -> `ebx,ecx,=`  

`add ebx, edi` -> `edi,ebx,+=,$o,of,=,$s,sf,=,$z,zf,=,$c31,cf,=,$p,pf,=`

Okay, so it isn't very pretty or easy to read at first, but it's very easy to parse and process.

## ESIL commands

All ESIL-related commands are prefixed by `ae`.

```
[0x08048460]> ae?
|Usage: ae[idesr?] [arg]ESIL code emulation
| ae?                show this help
| ae??               show ESIL help
| aei                initialize ESIL VM state (aei- to deinitialize)
| aeim               initialize ESIL VM stack (aeim- remove)
| aeip               initialize ESIL program counter to curseek
| ae [expr]          evaluate ESIL expression
| aex [hex]          evaluate opcode expression
| ae[aA][f] [count]  analyse esil accesses (regs, mem..)
| aep [addr]         change esil PC to this address
| aef [addr]         emulate function
| aek [query]        perform sdb query on ESIL.info
| aek-               resets the ESIL.info sdb instance
| aec                continue until ^C
| aecs [sn]          continue until syscall number
| aecu [addr]        continue until address
| aecue [esil]       continue until esil expression match
| aetr[esil]         Convert an ESIL Expression to REIL
| aes                perform emulated debugger step
| aeso               step over
| aesu [addr]        step until given address
| aesue [esil]       step until esil expression match
| aer [..]           handle ESIL registers like 'ar' or 'dr' does
```

You can see all the ESIL instructions (27 at the time of writing) with `ae??`. These are explained in slightly more detail in the radare2 book.

## ESIL in practice

Let's load up our tutorial binary in radare2:

`r2 -A ./esil` (notice that we are not running it in debug mode)

We'll first see what the `main` function does via `pdf @ main`. It seems that it reads an integer via `scanf`, sleeps, and then calls some function which receives our number.

Let's inspect that function.

```
[0x08048460]> pdf
            ;-- check:
(fcn) mystery 47
; arg int arg_8h @ ebp+0x8
; CALL XREF from 0x080484e0 (main)
0x08048460      55             push ebp
0x08048461      89e5           mov ebp, esp
0x08048463      8b4508         mov eax, dword [ebp + arg_8h] ; [0x8:4]=0
0x08048466      bb37130000     mov ebx, 0x1337
0x0804846b      89d9           mov ecx, ebx
0x0804846d      31d3           xor ebx, edx
0x0804846f      01fb           add ebx, edi
0x08048471      21f7           and edi, esi
0x08048473      09df           or edi, ebx
0x08048475      83c320         add ebx, 0x20
0x08048478      01f7           add edi, esi
0x0804847a      89cb           mov ebx, ecx
0x0804847c      29d8           sub eax, ebx
0x0804847e      83ef31         sub edi, 0x31
0x08048481      29fb           sub ebx, edi
0x08048483      31f7           xor edi, esi
0x08048485      81e6000000ff   and esi, 0xff000000
0x0804848b      89cb           mov ebx, ecx
0x0804848d      c9             leave
0x0804848e      c3             ret
```

I have renamed it to `mystery`. It seems to perform a lot of operations using all the registers. We can use ESIL to get some valuable information.

> **Note**: You can cycle between the representations of the instructions displayed in visual mode by pressing `O`. You can also enable emulation comments on the right hand side via `e asm.emu=true`.

The instructions prefixed with `aea` will show us which registers are being read, written to or not used at all within the next instructions, next bytes or the entire function.

```
[0x08048460]> aeaf
A: esp ebp eax ebx ecx edx zf pf sf cf of edi esi eip
R: esp ebp ebx edx edi esi ecx eax
W: esp ebp eax ebx ecx zf pf sf cf of edi esi eip
N: edx
```

Interesting; it seems the `edx` register is untouched by the function.

Let's set our seek to `0x08048466`, which is after the function's argument, our number, is being read from the stack into `eax`. We want to feed `eax` some values and then emulate the function from this point on.

> **Note**: In the following examples, ESIL will need to write in memory, but we've opened the binary in read-only mode. To bypass this, use `e io.cache = true`.

Now we can initialize the ESIL VM state and set the VM program counter (PC or EIP) to point to our seek.

```
[0x08048466]> aei
[0x08048466]> aeip
[0x08048466]> aer
oeax = 0x00000000
eax = 0x00000000
ebx = 0x00000000
ecx = 0x00000000
edx = 0x00000000
esi = 0x00000000
edi = 0x00000000
esp = 0xfffffd10
ebp = 0x00000000
eip = 0x08048466
eflags = 0x00000000
```

Notice that indeed `eip` is equal to our seek.

We can change any register value using `aer <register>=`. Let's set `eax`, which theoretically stores our input number, to some arbitrary value.

```
[0x08048466]> aer eax=0x1234
[0x08048466]> aer
oeax = 0x00000000
eax = 0x00001234
ebx = 0x00000000
ecx = 0x00000000
edx = 0x00000000
esi = 0x00000000
edi = 0x00000000
esp = 0xfffffd10
ebp = 0x00000000
eip = 0x08048466
eflags = 0x00000000
```

This is where ESIL comes in quite handy. Althought this is a didactic exercise, you can imagine a more complex example in which it is very hard to determine what is happening to our input.

The ESIL VM can be used like a debugger. You can step and continue as usual, but you can also continue until a given ESIL expression is true.

Let's continue execution until the value of `eax` is greater than its initial one.

```
[0x08048466]> "aecue eax,0x1234,>"
ESIL BREAK!
[0x08048466]> aer
oeax = 0x00000000
eax = 0xfffffefd
ebx = 0x00001337
ecx = 0x00001337
edx = 0x00000000
esi = 0x00000000
edi = 0x00003974
esp = 0x00000008
ebp = 0x464c457f
eip = 0x0804847e
eflags = 0x00000081
```

> **Note**: Mind the quotes surrounding the `aecue` expression. These are to ensure that r2 interprets it as a single command, not a sequence of commands.

Notice that the condition has been reached. Let's seek to the location at which the VM stopped and print the preceding instruction.

```
[0x08048466]> sr eip
[0x0804847e]> pd -1
│           0x0804847c      29d8           sub eax, ebx
```

It seems that `eax` has been changed by subtracting `ebx` from it. Notice that `ebx` is still at `0x1337`, which means that this is the expected value in order for `eax` to become 0.

We can test this by resetting `eip` to the initial position, setting `eax` to 0x1337 and continuing emulation until `eax` reaches 0.

This was only an introductory tutorial to what can be accomplished by using ESIL.
