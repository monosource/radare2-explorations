# The Basics

In this section, we will simply get introduced to radare2 and explore the basics of its commands. It is not centered on any particular aspect of radare2, but will provide you with some vital background needed to more efficiently wrap your head around the overall structure and design of the framework.

## Resources

There are plenty of resources scattered around the web from which you can learn more on how you can use radare2 for various tasks.

Most of these resources can be found in [this](http://radare.today/posts/radare2-is-documented/) blog post.

What is left out is [radare.tv](http://radare.tv/), which is quite a magical place which you should check out from time to time.

There's also the more practical and focused [workshop](https://github.com/Maijin/Workshop2015) by Maijin.

## So, what is radare2?

You would not be wrong if you were to say that radare2 (or r2, in short) is a disassembler. But it is so much more.

r2 can aptly be named a reverse engineering framework, with some extra features on the side.

Here is a (non-exhaustive) list of what r2 can be:

* Disassembler
* Assembler
* Debugger
* Hex editor
* Exploit development tool
* Emulator
* Binary diffing tool
* Shellcode compiler
* Launcher with specific contexts
* And more...

It can run on all major operating systems and understand any gizmo which has as little as an oscillator in it.

## Taking the plunge

Radare2 can be started by typing **radare2** or **r2** in the console. You will be prompted with the following:

```shell
r2
Usage: r2 [-dDwntLqv] [-P patch] [-p prj] [-a arch] [-b bits] [-i file]
          [-s addr] [-B blocksize] [-c cmd] [-e k=v] file|pid|-|--|=
```

The important argument here is **file**. A process id (pid) can also be supplied when we want to attach to a process which is already running, but in most cases we will be using files.

Let's try it out on the humble and ubiquitous **/bin/ls**

```shell
r2 /bin/ls
-- Change the UID of the debugged process with child.uid (requires root)
[0x004048c5]>
```

Notice that the prompt changes. We are now exploring the memory map of **/bin/ls**. The value between parentheses is the current address position within the current file. This is the entry point of the binary (unless radare is configured differently).

## Help!

Now you may want to navigate, disassemble, search, mark and do other operations. How?

```
[0x004048c5]> help
|ERROR| Invalid command 'help' (0x68)
```

Radare2 is self-documented. For a full list of commands, a simple question mark (`?`) will suffice and is much faster than typing `help` all the time.

```
[0x004048c5]> ?
Usage: [.][times][cmd][~grep][@[@iter]addr!size][|>pipe] ; ...
Append '?' to any char command to get detailed help
Prefix with number to repeat command N times (f.ex: 3x)
|%var =valueAlias for 'env' command
| *off[=[0x]value]     Pointer read/write data/values (see ?v, wx, wv)
| (macro arg0 arg1)    Manage scripting macros
| .[-|(m)|f|!sh|cmd]   Define macro or load r2, cparse or rlang file
| = [cmd]              Run this command via rap://
| /                    Search for bytes, regexps, patterns, ..
| ! [cmd]              Run given command as in system(3)
| # [algo] [len]       Calculate hash checksum of current block
| #!lang [..]          Hashbang to run an rlang script
| a                    Perform analysis of code
| b                    Get or change block size
| c [arg]              Compare block with given data
| C                    Code metadata management
| d                    Debugger commands
| e [a[=b]]            List/get/set config evaluable vars
| f [name][sz][at]     Set flag at current address
| g [arg]              Go compile shellcodes with r_egg
| i [file]             Get info about opened file
| k [sdb-query]        Run sdb-query. see k? for help, 'k *', 'k **' ...
| m                    Mountpoints commands
| o [file] ([offset])  Open file at optional address
| p [len]              Print current block with format and length
| P                    Project management utilities
| q [ret]              Quit program with a return value
| r [len]              Resize file
| s [addr]             Seek to address (also for '0x', '0x1' == 's 0x1')
| S                    Io section manipulation information
| t                    Cparse types management
| T [-] [num|msg]      Text log utility
| u                    uname/undo seek/write
| V                    Enter visual mode (vcmds=visualvisual  keystrokes)
| w [str]              Multiple write operations
| x [len]              Alias for 'px' (print hexadecimal)
| y [len] [[[@]addr    Yank/paste bytes from/to memory
| z                    Zignatures management
| ?[??][expr]          Help or evaluate math expression
| ?$?                  Show available '$' variables and aliases
| ?@?                  Misc help for '@' (seek), '~' (grep) (see ~??)
| ?:?                  List and manage core plugins
[0x004048c5]>
```

This is understandably a daunting sight to behold, and it will not get any easier from this point on. Thankfully, most of these are self-contained and define a specific category of subcommands. For example, all analysis commands begin with `a`, all commands related to the debugger begin with `d`, all printing commands begin with `p` etc.

## Looking through commands

While learning radare2, you will iteratively consult the built-in documentation to find commands which help you accomplish your specific needs, by appending a `?` after each combination of interest. For example:

```
[0x004048c5]> a?
|Usage: a[abdefFghoprxstc] [...]
| ab [hexpairs]     analyze bytes
| aa                analyze all (fcns + bbs) (aa0 to avoid sub renaming)
| ac [cycles]       analyze which op could be executed in [cycles]
| ad                analyze data trampoline (wip)
| ad [from] [to]    analyze data pointers to (from-to)
| ae [expr]         analyze opcode eval expression (see ao)
| af[rnbcsl?+-*]    analyze Functions
| aF                same as above, but using anal.depth=1
| ag[?acgdlf]       output Graphviz code
| ah[?lba-]         analysis hints (force opcode size, ...)
| ai [addr]         address information (show perms, stack, heap, ...)
| ao[e?] [len]      analyze Opcodes (or emulate it)
| an[an-] [...]     manage no-return addresses/symbols/functions
| ar                like 'dr' but for the esil vm. (registers)
| ap                find prelude for current offset
| ax[?ld-*]         manage refs/xrefs (see also afx?)
| as [num]          analyze syscall using dbg.reg
| at[trd+-%*?] [.]  analyze execution traces
Examples:
 f ts @ `S*~text:0[3]`; f t @ section..text
 f ds @ `S*~data:0[3]`; f d @ section..data
 .ad t t+ts @ d:ds

[0x004048c5]> af?
|Usage: af
| af ([name]) ([addr])              analyze functions (start at addr or $$)
| afr ([name]) ([addr])             analyze functions recursively
| af+ addr size name [type] [diff]  hand craft a function (requires afb+)
| af- [addr]                        clean all function analysis data (or function at addr)
| afa[?] [idx] [name] ([type])      add function argument
| af[aAv?][arg]                     manipulate args, fastargs and variables in function
| afb+ fa a sz [j] [f] ([t]( [d]))  add bb to function @ fcnaddr
| afb [addr]                        List basic blocks of given function
| afB 16                            set current function as thumb (change asm.bits)
| afc@[addr]                        calculate the Cyclomatic Complexity (starting at addr)
| afC[a] type @[addr]               set calling convention for function (afC?=list cc types)
| aff                               re-adjust function boundaries to fit
| afF[1|0|]                         fold/unfold/toggle
| afg                               non-interactive ascii-art basic-block graph (See VV)
| afi [addr|fcn.name]               show function(s) information (verbose afl)
| afl[*] [fcn name]                 list functions (addr, size, bbs, name)
| afo [fcn.name]                    show address for the function named like this
| afn name [addr]                   rename name for function at address (change flag too)
| afna                              suggest automatic name for current offset
| afs [addr] [fcnsign]              get/set function signature at current address
| afx[cCd-] src dst                 add/remove code/Call/data/string reference
| afv[?] [idx] [type] [name]        add local var on current function

[0x004048c5]> afn?
Usage: afn[sa] - analyze function names
 afna       - construct a function name for the current offset
 afns       - list all strings associated with the current function
 afn [name] - rename function
```

While this "forest" of a documentation does a decent job of what each and every command does, it does not tell you anything about **how** to use them. This is what the Internet (and subsequently, this book) is for. As mentioned before, radare2 can be used in plenty of scenarios. Not everyone is interested in shellcodes or DNA sequencing, so it makes a bit of sense not to include domain-specific examples within the documentation.

## Command philosophy

```
Usage: [.][times][cmd][~grep][@[@iter]addr!size][|>pipe] ;
```

This is the command format for radare2. Although this looks cryptic, only the command itself is mandatory, and it will operate using some default values as we will see further on.

If you have some experience working with \*nix shell, Vim, sed, awk, then learning radare2's commands will be *slightly* more intuitive.

### Current seek

In general (i.e. default behavior), each command has a point of reference, which is usually the current position in memory, indicated by the prompt. Any printing, writing or analysis commands will be performed at the current offset (address) in the file. For example:

```
[0x004048c5]> pd 1
    ;-- entry0:
            0x004048c5      31ed           xor ebp, ebp
```

Disassembles one instruction at address `0x4048c5`, which is the entry point for **/bin/ls**.

### Block size

If we do not specify a number of instructions to disassemble, the default `block size` will be used instead. This can be shown or changed with the command `b`.

```
[0x004048c5]> b
0x100
[0x004048c5]> pd
    ;-- entry0:
            0x004048c5      31ed           xor ebp, ebp
            0x004048c7      4989d1         mov r9, rdx
            0x004048ca      5e             pop rsi
            0x004048cb      4889e2         mov rdx, rsp
            0x004048ce      4883e4f0       and rsp, 0xfffffffffffffff0
            0x004048d2      50             push rax
            0x004048d3      54             push rsp
            0x004048d4      49c7c0602541.  mov r8, 0x412560
            0x004048db      48c7c1f02441.  mov rcx, 0x4124f0
            0x004048e2      48c7c7a02840.  mov rdi, 0x4028a0           ; "AWAVAUATUS..H..H...." @ 0x4028a0
            0x004048e9      e802dcffff     call sym.imp.__libc_start_main
            0x004048ee      f4             hlt
            0x004048ef      90             nop
            0x004048f0      b85fc66100     mov eax, 0x61c65f           ; ".interp" @ 0x61c65f
            0x004048f5      55             push rbp
            0x004048f6      482d58c66100   sub rax, 0x61c658
            0x004048fc      4883f80e       cmp rax, 0xe
            0x00404900      4889e5         mov rbp, rsp
        ┌─< 0x00404903      761b           jbe 0x404920
        │   0x00404905      b800000000     mov eax, 0
        │   0x0040490a      4885c0         test rax, rax
       ┌──< 0x0040490d      7411           je 0x404920
       ││   0x0040490f      5d             pop rbp
       ││   0x00404910      bf58c66100     mov edi, 0x61c658           ; "strtab" @ 0x61c658
       ││   0x00404915      ffe0           jmp rax
       ││   0x00404917      660f1f840000.  nop word [rax + rax]
       └└─> 0x00404920      5d             pop rbp
            0x00404921      c3             ret
            0x00404922      66666666662e.  nop word cs:[rax + rax]
        ┌─> 0x00404930      be58c66100     mov esi, 0x61c658           ; "strtab" @ 0x61c658
        │   0x00404935      55             push rbp
        │   0x00404936      4881ee58c661.  sub rsi, 0x61c658
        │   0x0040493d      48c1fe03       sar rsi, 3
        │   0x00404941      4889e5         mov rbp, rsp
        │   0x00404944      4889f0         mov rax, rsi
        │   0x00404947      48c1e83f       shr rax, 0x3f
        │   0x0040494b      4801c6         add rsi, rax
        │   0x0040494e      48d1fe         sar rsi, 1
       ┌──< 0x00404951      7415           je 0x404968
       ││   0x00404953      b800000000     mov eax, 0
       ││   0x00404958      4885c0         test rax, rax
      ┌───< 0x0040495b      740b           je 0x404968
      │││   0x0040495d      5d             pop rbp
      │││   0x0040495e      bf58c66100     mov edi, 0x61c658           ; "strtab" @ 0x61c658
      │││   0x00404963      ffe0           jmp rax
      │││   0x00404965      0f1f00         nop dword [rax]
      └└──> 0x00404968      5d             pop rbp
        │   0x00404969      c3             ret
        │   0x0040496a      660f1f440000   nop word [rax + rax]
        │   0x00404970      803d417d2100.  cmp byte [rip + 0x217d41], 0
       ┌──< 0x00404977      7511           jne 0x40498a
       ││   0x00404979      55             push rbp
       ││   0x0040497a      4889e5         mov rbp, rsp
       ││   0x0040497d      e86effffff     call 0x4048f0
       ││   0x00404982      5d             pop rbp
       ││   0x00404983      c6052e7d2100.  mov byte [rip + 0x217d2e], 1 ; [0x61c6b8:1]=105
       └──> 0x0040498a      f3c3           ret
        │   0x0040498c      0f1f4000       nop dword [rax]
        │   0x00404990      bf00be6100     mov edi, section..jcr       ; section..jcr
        │   0x00404995      48833f00       cmp qword [rdi], 0
       ┌──< 0x00404999      7505           jne 0x4049a0
       │└─< 0x0040499b      eb93           jmp 0x404930
       │    0x0040499d      0f1f00         nop dword [rax]
       └──> 0x004049a0      b800000000     mov eax, 0
```

### @addr - Relative seek

A command can be issued relative to an offset via the use of `@`, like so:

```
[0x004048c5]> pd 10 @ main
    ;-- main:
    ;-- section_end..plt:
    ;-- section..text:
            0x004028a0      4157           push r15                    ; [12] va=0x004028a0 pa=0x000028a0 sz=64746 vsz=64746 rwx=--r-x .text
            0x004028a2      4156           push r14
            0x004028a4      4155           push r13
            0x004028a6      4154           push r12
            0x004028a8      55             push rbp
            0x004028a9      53             push rbx
            0x004028aa      89fb           mov ebx, edi
            0x004028ac      4889f5         mov rbp, rsi
            0x004028af      4881ec980300.  sub rsp, 0x398
            0x004028b6      488b3e         mov rdi, qword [rsi]
```

Addresses, symbolic names and even custom set flags can be used as offsets. This type of operation does not change the current seek.

#### !size

As we have seen, `pd` takes an argument specifying the number of instructions to disassemble. This may not be the case with other commands, which will use the default block size for their operation (particularly block writing commands). We may want to fine tune this, but without changing the block size beforehand.

One way to do this is by using `!size` after the address, as follows:

```
[0x004048c5]> p8 @ main
4157415641554154555389fb4889f54881ec98030000488b3e64488b042528000000488984248803000031c0e81fb00000be71854100bf06000000e830feffffbe3f514100bf28514100e851faffffbf28514100e807faffffbfc0a14000c705d89c210002000000e863fc000048b80000000000000080c7050fa8210000000000c605a8a821000148890551a921008b05979c210048c70550a921000000000048c7053da92100ffffffffc6059ea821000083f8020f848308000083f803742f83e8017405e8b6f8ffffbf01000000e80cf9ffff85c00f842c0e0000c705caa8210002000000c60563a8210001eb16be0500000031ffc705b0a8210000000000
[0x004048c5]> p8 @ main ! 32
4157415641554154555389fb4889f54881ec98030000488b3e64488b04252800
```

Notice that the first command will print 256 bytes, while the second one will print 32 bytes.

### Times

Like in Vim, commands can be prefixed by a number specifying the number of times you want it to execute. This is very useful when coupled with "repeatable" complex commands.

### ~grep

Radare2 features an internal `grep` which is very handy when you want to filter search results or iterate over them in a clever fashion. It can be used by appending a tilde `~` after a command.

For example, `i` prints out various info about the currently loaded binary.

```
[0x004048c5]> i
type     EXEC (Executable file)
file     /bin/ls
fd       7
size     0x1ce08
blksz    0x0
mode     -r--
block    0x100
format   elf64
pic      false
canary   true
nx       true
crypto   false
va       true
intrp    /lib64/ld-linux-x86-64.so.2
bintype  elf
class    ELF64
lang     c
arch     x86
bits     64
machine  AMD x86-64 architecture
os       linux
minopsz  1
maxopsz  16
pcalign  0
subsys   linux
endian   little
stripped true
static   false
linenum  false
lsyms    false
relocs   false
rpath    NONE
binsz    119892
```

But this is a lot to take in. Suppose we want only a few bits of information, such as position independence of code, canary, NX. We can use the internal grep to do this:

```
[0x004048c5]> i~pic
pic      false
[0x004048c5]> i~canary
canary   true
[0x004048c5]> i~nx
nx       true
```

#### :Row/[column] selection
Some commands output their result in table form. Rows and columns can be selected as follows:
```
[0x004048c5]> drr
   rax 0x0000000000000000  section_end.GNU_STACK
   rbx 0x0000000000000000  section_end.GNU_STACK
   rcx 0x0000000000000000  section_end.GNU_STACK
   rdx 0x0000000000000000  section_end.GNU_STACK
   rsi 0x0000000000000000  section_end.GNU_STACK
   rdi 0x0000000000000000  section_end.GNU_STACK
    r8 0x0000000000000000  section_end.GNU_STACK
    r9 0x0000000000000000  section_end.GNU_STACK
   r10 0x0000000000000000  section_end.GNU_STACK
   r11 0x0000000000000000  section_end.GNU_STACK
   r12 0x0000000000000000  section_end.GNU_STACK
   r13 0x0000000000000000  section_end.GNU_STACK
   r14 0x0000000000000000  section_end.GNU_STACK
   r15 0x0000000000000000  section_end.GNU_STACK
   rip 0x0000000000000000  section_end.GNU_STACK
   rbp 0x0000000000000000  section_end.GNU_STACK
rflags 0x0000000000000000  section_end.GNU_STACK
   rsp 0x0000000000000000  section_end.GNU_STACK
```

A particular column can be selected by using `[NUM]`

```
[0x004048c5]> drr~[0]
rax
rbx
rcx
rdx
rsi
rdi
r8
r9
r10
r11
r12
r13
r14
r15
rip
rbp
rflags
rsp
```

And a row can be selected by using `:NUM`

```
[0x004048c5]> drr~:5
   rdi 0x0000000000000000  section_end.GNU_STACK
```

The two can also be combined:

```
[0x004048c5]> drr~:5[0]
rdi
```

### |Pipes and >redirection

Commands can be piped over to external processing tools such as tr, awk, sed, cut, grep and so on.

```
[0x004048c5]> pd 10 | tr -s ' ' | cut -d ' ' -f 4 | tail -n +2
xor
mov
pop
mov
and
push
push
mov
mov
mov
```

The output of most commands can be redirected to a file.

```
[0x004048c5]> pcp > demo.py
```

### @@Iteration

A very powerful feature of radare2 is the ability to run a command over multiple points in a binary. This is useful when you tag a series of points which require the same patch and then patching them all in one swoop.

The simple example below prints the first 4 bytes of every function.

```
[0x004048c5]> p8 4 @@ fcn.*
```

Some commands will automatically add flags which can be iterated over. For example:

```
[0x004048c5]> / err
Searching 3 bytes from 0x00400000 to 0x0061d480: 65 72 72
Searching 3 bytes in [0x400000-0x61d480]
hits: 6
0x00401094 hit0_0 "err"
0x0040117f hit0_1 "err"
0x0040124d hit0_2 "err"
0x00416137 hit0_3 "err"
0x00417470 hit0_4 "err"
0x00417695 hit0_5 "err"
[0x004048c5]> pd 5 @@ hit0_*
```

We first look through the binary for 'err'. This results in flags being set at every corresponding 'hit' points. We can then iterate over these 'hits' and further process them.

### Other commands

Quick conversions can be performed via the use of `?`

```
[0x004048c5]> ? 1234
1234 0x4d2 02322 1.2K 0000:04d2 1234 11010010 1234.0 0.000000f 0.000000
```

Other useful commands can be found using `???`

```
[0x004048c5]> ???
|Usage: ?[?[?]] expression
| ? eip-0x804800     show hex and dec result for this math expr
| ?:                 list core cmd plugins
| ?! [cmd]           ? != 0
| ?+ [cmd]           ? > 0
| ?- [cmd]           ? < 0
| ?= eip-0x804800    hex and dec result for this math expr
| ??                 show value of operation
| ?? [cmd]           ? == 0 run command when math matches
| ?B [elem]          show range boundaries like 'e?search.in
| ?P paddr           get virtual address for given physical one
| ?S addr            return section name of given address
| ?V                 show library version of r_core
| ?X num|expr        returns the hexadecimal value numeric expr
| ?_ hudfile         load hud menu with given file
| ?b [num]           show binary value of number
| ?b64[-] [str]      encode/decode in base64
| ?d[.] opcode       describe opcode for asm.arch
| ?e string          echo string
| ?f [num] [str]     map each bit of the number as flag string index
| ?h [str]           calculate hash for given string
| ?i[ynmkp] arg      prompt for number or Yes,No,Msg,Key,Path and store in $$?
| ?ik                press any key input dialog
| ?im message        show message centered in screen
| ?in prompt         noyes input prompt
| ?iy prompt         yesno input prompt
| ?l str             returns the length of string
| ?o num             get octal value
| ?p vaddr           get physical address for given virtual address
| ?r [from] [to]     generate random number between from-to
| ?s from to step    sequence of numbers from to by steps
| ?t cmd             returns the time to run a command
| ?u num             get value in human units (KB, MB, GB, TB)
| ?v eip-0x804800    show hex value of math expr
| ?vi rsp-rbp        show decimal value of math expr
| ?x num|str|-hexst  returns the hexpair of number or string
| ?y [str]           show contents of yank buffer, or set with string
```
