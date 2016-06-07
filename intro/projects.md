# Project Management

When reverse engineering, it is a good practice to save often, since most tools do not feature undo capabilities. Also, it is somewhat unlikely, in typical use cases, that you will accomplish everything you need in one sitting.

Fortunately, radare2 has some basic project management capabilities.

```
[0x00406260]> P?
|Usage: P[?osi] [file]Project management
| Pc [file]    show project script to console
| Pd [file]    delete project
| Pi [file]    show project information
| Pl           list all projects
| Pn[j]        show project notes (Pnj for json)
| Pn [base64]  set notes text
| Pn -         edit notes with cfg.editor
| Po [file]    open project
| Ps [file]    save project
| PS [file]    save script file
| NOTE:        See 'e file.project'
| NOTE:        project files are stored in ~/.config/radare2/projects
```

A project will keep track of the current seek, flags and comments. It will not remember marks, however.

I've opened `/bin/ls` in radare2 and taken the liberty of renaming some functions

```
[0x00404d30]> afl~mystery
0x004048f0    4 50   -> 41   mystery_func1
0x00404a20   44 778  -> 499  mystery_func2
0x00404d30    6 71   -> 69   mystery_func3
```

Very inspiring, I know.

We can save our project via `Ps` and providing a project name.

```
[0x00404d30]> Ps myls
myls
```

We can add some notes in our configured editor (more on that later) via `Pn -`.

```
[0x00404d30]> Pn -
[0x00404d30]> Pn
I have labeled three functions accordingly.

The first one does bla.
The second one does blabla.
The third one is susceptible to a buffer overflow.
```

We can now save the project and quit radare. We can start r2 with no file input with `r2 -` and reload the project to resume from where we left off.

```
$ r2 -
[0x00000000]> Po myls
Close current session? (Y/n)
[0x00404d30]> Pn
I have labeled three functions accordingly.

The first one does bla.
The second one does blabla.
The third one is susceptible to a buffer overflow.

[0x00404d30]> afl~mystery
0x004048f0    4 50   -> 41   mystery_func1
0x00404a20   44 778  -> 499  mystery_func2
0x00404d30    6 71   -> 69   mystery_func3
[0x00404d30]> 
```
