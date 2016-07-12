# Navigation

Let's get back to our example:

```
r2 /bin/ls
 -- Find hexpairs with '/x a0 cc 33'
[0x004048c5]>
```

We'll start by fully analyzing the binary using `aaa`. Radare2 will automatically delimit and name functions for us.

```
[0x004048c5]> aaa
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze len bytes of instructions for references (aar)
[x] Analyze function calls (aac)
[ ] [*] Use -AA or aaaa to perform additional experimental analysis.
[x] Constructing a function name for fcn.* and sym.func.* functions (aan))
```

## Flags

Whatever radare2 finds and considers to be interesting (strings, functions, sections, relocs and so on) a corresponding "flag" will be added for it. A flag is nothing more than a bookmark at an offset within the file, kept as a string.

Flags are grouped up in flagspaces. A flagspace is a namespace for flags. (i.e. all flags marking strings will be grouped up under the 'strings' flagspace).

Flags are useful because you can name them, navigate to them, iterate over them, group them into custom flagspaces.

```
[0x004028a0]> f my_special_flag 20 @ main + 15
[0x004028a0]> pd 1 @ main + 15
|           ;-- my_special_flag:
│           0x004028af      4881ec980300.  sub rsp, 0x398
│           0x004028b6      488b3e         mov rdi, qword [rsi]
[0x004028a0]> s my_special_flag 
[0x004028af]> pd 1
|           ;-- my_special_flag:
│           0x004028af      4881ec980300.  sub rsp, 0x398
```


You can list all the flags with the command `f`. Flags generally have a prefix in their name, such as `str.`, `sym.`, `sub.`, `fcn.` etc. These are very useful since you can grep for them and find something of interest.

## Seeking

You can seek to any virtual address within the binary using `s`. This is where flags come in handy, because you can seek to them.

```
[0x004048c5]> afl~main # List function flags and grep for 'main'
0x00402300    2 16   -> 48   sym.imp.textdomain
0x00402340    2 16   -> 48   sym.imp.bindtextdomain
0x004024f0    2 16   -> 48   sym.imp.__libc_start_main
0x004028a0  277 7780 -> 5801 main
[0x004048c5]> s main # seek to main
[0x004028a0]> 
```

Some commands in radare2 will add new flags, such as the search command.

```
[0x004028a0]> / ASCII
Searching 5 bytes from 0x00400000 to 0x0061d480: 41 53 43 49 49 
Searching 5 bytes in [0x400000-0x61d480]
hits: 1
0x00418cbc hit0_0 "ASCII"
[0x004028a0]> s hit0_0 
[0x00418cbc]> 
```

Notice that radare2 automatically flags each "hit" of a search for you to use.
This is also useful for iteration via `@@` and regex. You can execute a command for every result of a search, such as printing, xoring with a value, or even more complex operations

```
[0x00418cbc]> /a jmp rax
Searching 2 bytes in [0x400000-0x61d480]
hits: 2
0x00404915 hit1_0 ffe0
0x00404963 hit1_1 ffe0
[0x00418cbc]> pd 2 @@ hit1_*
|           ;-- hit1_0:
│           0x00404915      ffe0           jmp rax
            0x00404917      660f1f840000.  nop word [rax + rax]
            ;-- hit1_1:
            0x00404963      ffe0           jmp rax
            0x00404965      0f1f00         nop dword [rax]
```
