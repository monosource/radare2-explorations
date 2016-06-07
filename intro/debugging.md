# Debugging

Let's load up `/bin/ls` in debug mode. There are multiple ways to do this.

One way is to load it up directly in debug mode via the `d` flag.

`r2 -Ad /bin/ls`

If `/bin/ls` is already opened in read-only mode, you can reopen it via `ood`, or the alias `doo`. Any flags you set will be preserved.

## Commands

All debugging-related commands are prefixed with `d`, which is easy to remember and quite handy.

```
[0x7f5c795e8190]> d?
|Usage: d # Debug commands
| db[?]                   Breakpoints commands
| dbt                     Display backtrace based on dbg.btdepth and dbg.btalgo
| dc[?]                   Continue execution
| dd[?]                   File descriptors (!fd in r1)
| de[-sc] [rwx] [rm] [e]  Debug with ESIL (see de?)
| dg <file>               Generate a core-file (WIP)
| dh [handler]            List or set debugger handler
| dH [handler]            Transplant process to a new handler
| di                      Show debugger backend information (See dh)
| dk[?]                   List, send, get, set, signal handlers of child
| dm[?]                   Show memory maps
| do                      Open process (reload, alias for 'oo')
| doo[args]               Reopen in debugger mode with args (alias for 'ood')
| dp[?]                   List, attach to process or thread id
| dr[?]                   Cpu registers
| ds[?]                   Step, over, source line
| dt[?]                   Display instruction traces (dtr=reset)
| dw <pid>                Block prompt until pid dies
| dx[?]                   Inject and run code on target process (See gs)
```

You can set breakpoints using `db <address/flag>`. `db` will simply list all breakpoints.

`ds <n>` will step into `n` instructions, while `dso <n>` will step over them (i.e. not following calls)

You should experiment as much as possible with each debugging command.

## Useful tips and tricks

### Continue until address/flag

Instead of setting a breakpoint at an address and then continuing execution with `dc`, you can instead enter `dcu <address>` and execution will continue until that address or flag.

Example:
```
[0x7fb12b928190]> dcu main
Continue until 0x004028a0 using 1 bpsize
hit breakpoint at: 4028a0
attach 21109 1
[0x004028a0]> 
```

### System call tracing

You can continue execution until a specific system call via `dcs <syscall name/number>`. You can trace all syscalls with `dcs*`.

Example:
```
[0x7f9e72ede190]> dcs mmap
Running child until syscalls:9 
hit breakpoint at: 7f9e72ede193
attach 21117 1
--> SN 0x7f9e72ef2e8c syscall 12 brk (0x0)
hit breakpoint at: 7f9e72ef2e92
--> SN 0x7f9e72ef4207 syscall 21 access (0x7f9e72ef7556 0x0)
hit breakpoint at: 7f9e72ef420d
--> SN 0x7f9e72ef42da syscall 9 mmap (0x0 0x2000 0x3 0x22 0xffffffff 0x0)
```

### Telescoping and references

Something else which you might be interested in when debugging is to find out what the registers and stack values point to (cross-references).

These can be achieved via `drr` and `pxr @ rsp`, respectively.

```
[0x7f0ac9d09190]> dcu main
Continue until 0x004028a0 using 1 bpsize
hit breakpoint at: 4028a0
attach 21122 1
[0x004028a0]> drr
  orax 0xffffffffffffffff  orax
   rax 0x00000000004028a0  (.text) (/bin/ls) rip main program R X 'push r15' 'ls'
   rbx 0x0000000000000000  rbp
   rcx 0x0000000000000000  rbp
   rdx 0x00007ffd390e4c18  rdx stack R W 0x7ffd390e5594 --> stack R W 0x524e54565f474458 (XDG_VTNR=7) --> ascii
    r8 0x00007f0ac98d5c60  (/lib/x86_64-linux-gnu/libc-2.19.so) r8 library R W 0x0 --> rbp
    r9 0x00007f0ac9d16de0  (/lib/x86_64-linux-gnu/ld-2.19.so) r9 library R X 'push rbp' 'ld-2.19.so'
   r10 0x00007ffd390e49b0  r10 stack R W 0x0 --> rbp
   r11 0x00007f0ac9550a50  (/lib/x86_64-linux-gnu/libc-2.19.so) r11 library R X 'push r14' 'libc-2.19.so'
   r12 0x00000000004048c5  (.text) (/bin/ls) r12 entry0 program R X 'xor ebp, ebp' 'ls'
   r13 0x00007ffd390e4c00  r13 stack R W 0x1 --> (.shstrtab) rdi
   r14 0x0000000000000000  rbp
   r15 0x0000000000000000  rbp
   rsi 0x00007ffd390e4c08  rsi stack R W 0x7ffd390e558c --> stack R W 0x736c2f6e69622f (/bin/ls) --> ascii
   rdi 0x0000000000000001  (.shstrtab) rdi
   rsp 0x00007ffd390e4b28  rsp stack R W 0x7f0ac9550b45 --> (/lib/x86_64-linux-gnu/libc-2.19.so) library R X 'mov edi, eax' 'libc-2.19.so'
   rbp 0x0000000000000000  rbp
   rip 0x00000000004028a0  (.text) (/bin/ls) rip main program R X 'push r15' 'ls'
rflags 0x0000000000000246  rflags
[0x004028a0]> pxr @ rsp!32
0x7ffd390e4b28  0x00007f0ac9550b45   E.U..... (/lib/x86_64-linux-gnu/libc-2.19.so) library R X 'mov edi, eax' 'libc-2.19.so'
0x7ffd390e4b30  0x0000000000000000   ........ rbp
0x7ffd390e4b38  0x00007ffd390e4c08   .L.9.... rsi stack R W 0x7ffd390e558c --> stack R W 0x736c2f6e69622f (/bin/ls) --> ascii
0x7ffd390e4b40  0x0000000100000000   ........
```

### Command at breakpoint hit

You can set radare2 to run a command automatically when hitting a breakpoint via `dbc`. This can be any sort of command, simple or complex.
Each breakpoint can have its own command!

Example:

```
[0x7f710cdd2190]> db main
[0x7f710cdd2190]> db entry0
[0x7f710cdd2190]> dbc main drr
[0x7f710cdd2190]> dbc entry0 pd 10
[0x7f710cdd2190]> dc
hit breakpoint at: 4048c5
  ;--         mov rdi, rsp
        call 0x7f710cdd5710
        mov r12, rax
        mov eax, dword [rip + 0x21fc57]
        pop rdx
        lea rsp, [rsp + rax*8]
        sub edx, eax
        push rdx
        mov rsi, rdx
        mov r13, rsp
[0x004048c5]> dc
hit breakpoint at: 4048c7
hit breakpoint at: 4028a0
  orax 0xffffffffffffffff  orax
   rax 0x00000000004028a0  (.text) (/bin/ls) section..text main program R X 'push r15' 'ls'
   rbx 0x0000000000000000  rbp
   rcx 0x0000000000000000  rbp
   rdx 0x00007ffefebe0af8  stack R W 0x7ffefebe2594 --> stack R W 0x524e54565f474458 (XDG_VTNR=7) --> ascii
    r8 0x00007f710c99ec60  (/lib/x86_64-linux-gnu/libc-2.19.so) library R W 0x0 --> rbp
    r9 0x00007f710cddfde0  (/lib/x86_64-linux-gnu/ld-2.19.so) rdx library R X 'push rbp' 'ld-2.19.so'
   r10 0x00007ffefebe0890  stack R W 0x0 --> rbp
   r11 0x00007f710c619a50  (/lib/x86_64-linux-gnu/libc-2.19.so) library R X 'push r14' 'libc-2.19.so'
   r12 0x00000000004048c5  (.text) (/bin/ls) rip entry0 program R X 'xor ebp, ebp' 'ls'
   r13 0x00007ffefebe0ae0  r13 stack R W 0x1 --> (.shstrtab) rsi
   r14 0x0000000000000000  rbp
   r15 0x0000000000000000  rbp
   rsi 0x00007ffefebe0ae8  stack R W 0x7ffefebe258c --> stack R W 0x736c2f6e69622f (/bin/ls) --> ascii
   rdi 0x0000000000000001  (.shstrtab) rsi
   rsp 0x00007ffefebe0a08  stack R W 0x7f710c619b45 --> (/lib/x86_64-linux-gnu/libc-2.19.so) library R X 'mov edi, eax' 'libc-2.19.so'
   rbp 0x0000000000000000  rbp
   rip 0x00000000004028a0  (.text) (/bin/ls) section..text main program R X 'push r15' 'ls'
rflags 0x0000000000000246 
```

This can be very useful when you have a breakpoint within a loop which changes a register or an area of memory. You can keep hitting the breakpoint and see how the register or memory region gets updated.
