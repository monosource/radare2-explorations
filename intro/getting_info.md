# Getting Information

Before going deep into analyzing a file with radare2, you first need some key pieces of information.

## Beauty is in the `i` of the beholder

r2 can give us quite a bit of information via the `i`-prefixed commands.

```
[0x004048c5]> i?
|Usage: i Get info from opened file
| Output mode:       
| '*'                Output in radare commands
| 'j'                Output in json
| 'q'                Simple quiet output
| Actions:           
| i|ij               Show info of current file (in JSON)
| iA                 List archs
| ia                 Show all info (imports, exports, sections..)
| ib                 Reload the current buffer for setting of the bin (use once only)
| ic                 List classes, methods and fields
| iC                 Show signature info (entitlements, ...)
| id                 Debug information (source lines)
| iD lang sym        demangle symbolname for given language
| ie                 Entrypoint
| iE                 Exports (global symbols)
| ih                 Headers
| ii                 Imports
| iI                 Binary info
| ik [query]         Key-value database from RBinObject
| il                 Libraries
| iL                 List all RBin plugins loaded
| im                 Show info about predefined memory allocation
| iM                 Show main address
| io [file]          Load info from file (or last opened) use bin.baddr
| ir|iR              Relocs
| is                 Symbols
| iS [entropy,sha1]  Sections (choose which hash algorithm to use)
| iV                 Display file version info
| iz                 Strings in data sections
| izz                Search for Strings in the whole binary
```

Information acquired this way is usually displayed in columns, which are easily greppable.

```
[0x004048c5]> iI
havecode true
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

[0x004048c5]> iI~pic
pic      false
[0x004048c5]> iI~canary
canary   true
[0x004048c5]> iI~nx
nx       true
[0x004048c5]> iI~lang
lang     c
[0x004048c5]> iI~stripped
stripped true
```

## Running commands in command line

You can use r2 to get precise information without actually needing to start it. You can feed r2 some commands to execute and then quit.

Example:
```
$ r2 -A -q -c 'iI~pic,canary,nx' /bin/ls
pic      false
canary   true
nx       true
```
