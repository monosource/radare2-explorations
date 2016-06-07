# Configuration

If you are a hacker at heart, each tool you use is probably tailored to your needs, your terminal is semi-transparent and the text is neon green... okay, maybe that is going a bit too far.

In any case, there are some quirks which you probably would like to change about radare2.

## Evaluable variables

radare2 comes with a giant list of evaluable vars, which can be listed via `e??` (too many to list here).

```
[0x00000000]> e?
|Usage: e[?] [var[=value]]Evaluable vars
| e?asm.bytes     show description
| e??             list config vars with description
| e               list config vars
| e-              reset config vars
| e*              dump config vars in r commands
| e!a             invert the boolean value of 'a' var
| eevar           open editor to change the value of var
| er [key]        set config key as readonly. no way back
| ec [k] [color]  set color for given key (prompt, offset, ...)
| et [key]        show type of given config variable
| e a             get value of var 'a'
| e a=b           set var 'a' the 'b' value
| env [k[=v]]     get/set environment variable
```

You can go through each one via `e?<eval_var>`. Thankfully, they are neatly and intuitively (most of the time) prefixed, so it's easy to find something specific.

For example, I'm slightly annoyed that the debugger leaves me in the loader instead of the entry point of my program (thought this has some use cases).

```
[0x00000000]> e??~entry
             dbg.bep: break on entrypoint (loader, entry, constructor, main)
```

In order to see what the current value is, and change it afterwards, use `e`:

```
[0x00000000]> e dbg.bep
loader
[0x00000000]> e dbg.bep=entry
[0x00000000]> e dbg.bep
entry
```

Excellent.

## Colors

In radare2 you can change any keywords to any RGB color you wish. Since this is a timeconsuming process, radare2 already comes with some color templates for you to use or change. You can access them via `eco`.

```
[0x00000000]> eco
dark
basic
ogray
zenburn
behelit
white
xvilka
lima
matrix
rasta
pink
smyck
twilight
solarized
focus
consonance
tango
```

You should cycle through the visual mode and change the themes around to see which one suits you best.

## Making changes permanent

Config vars are limited only to the current session. They are also saved and preserved by projects.

Once you find yourself reusing some specific settings, you should commit to them. Simply add the commands as you would in radare2 in `.radare2rc` in your $HOME directory.

Example:
```
e dbg.bep = entry
e dbg.follow = false
e asm.syntax = intel
e scr.color = true
eco xvilka
ec prompt red
e scr.utf8 = true
```
