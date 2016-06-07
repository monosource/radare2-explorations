# Modes of operation

Even though radare2 features a CLI (Command Line Interface), it can be used in a variety of modes.

## Command mode

This is the default mode in which radare2 starts, unless configured otherwise. All the available commands are accessible from this mode. These have already been discussed in the Basic introduction.

## Visual Mode

You can enter a slightly different mode of operation by pressing `V<Enter>`. Noticed that the output has changed into something similar to what `xxd` might show you. This is known as 'hex' mode. Indeed, radare2 can be used as a hex editor.

The prompt is now at the top of the screen. Notice that it looks slightly different:
```
[0x004048c5 15% 448 /bin/ls]> x @ entry0
```

The command shown after the prompt is what's being used to generate the output. If you were to return to command mode (by pressing `q`), and enter `x @ entry0`, you will see the same output as before. The only difference is that in visual mode you can interact with and update it in real time.

As before, you can obtain a list of available commands and shortcuts in visual mode by pressing `?`.

```
Visual mode help:
?        show this help or enter the userfriendly hud
&        rotate asm.bits between supported 8, 16, 32, 64
%        in cursor mode finds matching pair, otherwise toggle autoblocksz
@        set cmd.vprompt to run commands before the visual prompt
!        enter into the visual panels mode
_        enter the flag/comment/functions/.. hud (same as VF_)
=        set cmd.vprompt (top row)
|        set cmd.cprompt (right column)
.        seek to program counter
/        in cursor mode search in current block
:cmd     run radare command
;[-]cmt  add/remove comment
/*+-[]   change block size, [] = resize hex.cols
>||<     seek aligned to block size
a/A      (a)ssemble code, visual (A)ssembler
b        toggle breakpoint
c/C      toggle (c)ursor and (C)olors
d[f?]    define function, data, code, ..
D        enter visual diff mode (set diff.from/to)
e        edit eval configuration variables
f/F      set/unset or browse flags. f- to unset, F to browse, ..
gG       go seek to begin and end of file (0-$s)
hjkl     move around (or HJKL) (left-down-up-right)
i        insert hex or string (in hexdump) use tab to toggle
mK/'K    mark/go to Key (any key)
M        walk the mounted filesystems
n/N      seek next/prev function/flag/hit (scr.nkey)
o        go/seek to given offset
O        toggle asm.esil
p/P      rotate print modes (hex, disasm, debug, words, buf)
q        back to radare shell
r        browse anal info and comments
R        randomize color palette (ecr)
sS       step / step over
T        enter textlog chat console (TT)
uU       undo/redo seek
v        visual code analysis menu
V        (V)iew graph using cmd.graph (agv?)
wW       seek cursor to next/prev word
xX       show xrefs/refs of current function from/to data/code
yY       copy and paste selection
z        fold/unfold comments in disassembly
Z        toggle zoom mode
Enter    follow address of jump/call
Function Keys: (See 'e key.'), defaults to:
  F2      toggle breakpoint
  F4      run to cursor
  F7      single step
  F8      step over
  F9      continue
```

Notice that these are very different from the commands we're used to, but arguably fewer. Notable ones are `p/P` for cycling display modes, `o` for seeking, `;` for adding comments, `V` for visual ASCII graph. Of course, you can still execute any r2 command via `:`, or quitting the visual mode altogether with `q`.
Visual mode is very useful when debugging, since you can both see where the current program counter is located and seek to inspect any location you desire.
