Title: SD TCL - Programs and the Catalog
Subtitle: Compiling SD BASIC, running it, and giving it a name you can type.

Getting from source to something you can type is three steps: **compile it, put
it in a catalog, then call it by name.** Only the first is compulsory.

SD folds case, so a command may be typed in either case. Commands are shown here
in lower case. In the tables, *italics* mark something you supply and **bold**
marks a word typed as it stands; braces mark an optional part.

> **The listings on this page** use a two-line program in an account's `bp` file.

## Compiling

```
basic {file} record
```

The file defaults to `bp`. What comes back names the file and record, then the
error count:

```
basic bp zzprog
```

```
Compiling bp zzprog
WARNING: Final END statement is missing
0 error(s)
Compiled 1 program(s) with no errors
```

**A warning is not an error and the program still runs.** The line above is a
warning, the count says `0 error(s)`, and that program ran. **Read the count,
not the presence of output.**

`@system.return.code` is the number of programs compiled, and negative on a
fatal error.

**The object code goes into `bp.out`**, a separate file beside `bp`, one record
per program. **Its name is lower case even when the source record is not** —
a source record `ZZPROG` compiles to an object record `zzprog`. That is this
port's naming, and it is worth knowing before you go looking for the object by
the name you typed.

**`basic ... no.catalog` compiles and catalogs nothing.** A `$catalog` line in
the source or in a file it includes, or a `CATALOG` line in the `$BASIC.OPTIONS`
record, makes the compile catalog the program as well; with `no.catalog` none
of them is acted on, and the compile says so
(`zzprog compiled; its catalog directive was not run (no.catalog)`).

## Running without cataloging

```
run {file} record
```

```
run bp zzprog
```

```
ZZPROG OK
```

`run` needs the file and record name every time. It is what you use while you
are still writing the thing.

## The catalog is what gives a program a name

```
catalog {file {call.name}} {program | *} {local | global | pcode} {no.xref}
```

Catalog it and the name becomes a command:

```
catalog bp zzprog
zzprog hello
```

```
ZZPROG added to private catalogue
ZZPROG OK
```

**The program sees the whole command line.** `@sentence` in the program above
read `zzprog hello`, so arguments arrive as typed and the program parses them
itself.

**A program run by its catalog name takes `lptr` and `no.page` as `run`
does.** The words stay in the sentence for the program to see. A word in
quotation marks is not a keyword. This applies to a catalog name only, not to
the name of a VOC verb.

### Three catalogs, and they are not the same

| | |
|---|---|
| **private** | the account's own. The default when no keyword is given |
| **local** | a VOC entry in the account, so the name works only there |
| **global** | `gcat` in the system account — **every account sees it** |

**Nobody catalogs globally on Solo — `admin` or not.** `catalog ... global` is
refused with *"The global catalog holds the SD Core server's programs from
global.bp.out and is changed only by sync.global.catalog"*. **The same refusal
applies to an implicit global catalog** — one chosen by a `!`, `_` or `$`
prefix on the call name rather than by the `global` keyword (the `*` and `-`
prefixes were removed in 2.0-0) — so the prefix is not a way round it. On a managed computer the global catalog is the SD Core
for Linux server's, and only it can change it — see *Managed mode* in the
GettingStarted set. Catalog your own programs `private` or `local`.

Private and local cataloging need none of this and work in a programmer's own
account.

**`catalog ... local` asks before it overwrites.** When the VOC already holds a
local catalog pointer under that name and it points at a different object
file, you are asked first (Return is no). `no.query` goes ahead without asking,
and cataloging the same file again does not ask.

### Seeing and removing

```
map {all} {lptr {n}} {file {name}}
```

```
System catalogue map at 23:12:07 on 26 Aug 2026                    Page 1
  Catalogue name..................  Compiled.  Time....  ...Obj  ..Xref  ..Size
  ZZPROG                            26 Aug 26  23:12:07     189       6     195
                                                                Total:      195
```

`Obj` and `Xref` are the object-code and cross-reference sizes in bytes, and
`Size` is the total. An empty one says so:

```
Catalogue is empty
```

```
delete.catalog name... {global | local}
```

```
ZZPROG deleted from the private catalogue
```

> **`map` also takes a `file` option**, which writes the map to a file instead
> of the screen. **It asks before overwriting** — *"File xx will be cleared and
> overwritten. Continue (Y/N)?"* — so it is not usable from a script driving SD
> down a pipe.

### Removing a local entry and its object code: `decatalog`

```
decatalog {file} name ...
```

`delete.catalog` *name* `local` removes the VOC pointer that `catalog ... local`
made, and **leaves the object file where it was**. `decatalog` removes the
pointer **and the object code it points to**. The file name, which other systems
need, is accepted and ignored: the pointer holds the pathname of the object code.

```
decatalog p3
```

```
p3 deleted from the local catalogue
```

After `delete.catalog p3 local` the compiled program was still in the object
file (`bp.out`); after `decatalog p3` it was gone. A name that is not a local
entry is *p9 is not in the local catalog* (the name as you typed it), and no
name at all is *Catalog name required*. It acts on the local catalog only;
the private and the global catalog are `delete.catalog`'s.

## Continued in

[Programs: find.program, Dictionaries and the
IDE](24a-sd-tcl-find-program-dictionaries-and-the-ide.html) — find.program,
dictionaries and the opening of the IDE.
