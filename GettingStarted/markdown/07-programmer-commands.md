Title: Development and file commands
Subtitle: Compiling, editing, and the verbs that maintain files, indexes and records in bulk.

**Your account has all of these, from the moment it is installed.** SD Core
used to hold them back from a *standard* account; that split is gone, and Solo
has one account — see [The account and its passwords](05-account-types.html).
This page is a reference for what each of them does, not a list of what you have
been given.

**Two things, named as they come up below, still need more than the verb**:
cataloging globally is refused to everyone, and editing the VOC directly needs
`admin` — see [What actually gates these](#what-actually-gates-these) at the foot
of this page. The full-screen editors need nothing extra.

## Compile, catalog and run

| | |
|---|---|
| **`basic`** | compile SD BASIC source |
| **`catalog`** | add to the catalog |
| **`delete.catalog`** | remove from it |
| **`compile.dict`** | compile dictionary items |
| **`run`** | run a compiled program |
| **`map`** | show a program's map |
| **`generate`** | generate source |
| **`phantom`** | start a background process |

> **Nobody catalogs globally.** `catalog ... global`, a name beginning `!`,
> `_` or `$`, and `delete.catalog` of a global entry, are refused whoever asks and
> whether or not `admin` is unlocked: *The global catalog holds the SD Core
> server's programs from global.bp.out and is changed only by
> sync.global.catalog*. On a managed computer the global catalog is the SD Core for Linux server's
> — see [Managed mode](15-managed-mode.html). Private and local cataloging
> work as they always did: catalog your own programs with `catalog ... local`.

## Edit and debug

| | |
|---|---|
| **`ed`** | the line editor. Needs nothing installed. `edit` is an alias for it |
| **`sed`** | a **full-screen** editor that runs inside SD, with drop-down menus and color for SD BASIC and five other languages |
| **`micro`** | a **full-screen** editor — opens the record in `micro` |
| **`ced`** | a full-screen editor for a record that is a **data collection** (JSON), one level at a time with the type of every element shown |
| **`ide`** | a full-screen workbench: the files and records, the program and its compile messages on one screen |
| **`debug`** | the BASIC debugger |
| **`pstat`** · **`pdebug`** · **`pdump`** · **`dump`** | process introspection |

### Editors

**`edit` is not a full-screen editor here** — a real difference from SD Core for
Windows, where it opens Microsoft Edit. There is no Linux equivalent to alias it
to, so `edit` runs the **line** editor, `ed`, instead. There are two full-screen
editors. **`sed` is SD's own**: a program written in SD BASIC that runs inside SD,
so it needs no installing and no operating-system permission (its menus, keys and
syntax color are described in *The SED Screen Editor*, in the User documentation).
The other is the program `micro`, which the SD installer installs as an ordinary
package:

```
sed   bp myprog
micro bp myprog
micro dict customers name
```

`micro` writes the record to a working copy, opens the editor on it, reads it
back, and asks whether to save. For a `bp` record it then offers the compile and
the catalog. `sed` edits the record itself, locks it while it is open, and
compiles from its own Tools menu.

**`micro` runs outside SD, so it needs the `os` switch**, which is off as
delivered (without it it says it is not available to the account; the
administrator turns it on with `set.os.access os on`, and `ed` and `sed` need
nothing of the kind: `sed` asks for the `os` switch only when it is given an
operating-system path outside the account's own directory). See
[Operating system access](06b-operating-system-access.html).

**Both are terminal editors**, so they work over ssh as well as at the console.

**`sed`, `ced` and `ide` run inside SD** and draw the screen themselves, so they need no
operating-system permission and no editor installed; they need a terminal window of at
least 140 columns by 43 lines (the debugger and the home screen use the same frame).

**Both highlight SD BASIC.** `micro`'s highlighting is per-user: the `micro` verb
copies SD's syntax file into your `~/.config/micro/syntax` the first time it is
used. **It applies to a `bp` record and to nothing else.** SD names the working
copy so the editor can recognize the language — a record edited out of any other
file is treated as plain text, which is correct for a VOC entry or a data record.

**`ed` is unaffected and is still there.**

### What the editors are good for, and what they are not

**They are text editors**, so they suit a record whose content is lines of text:

| | |
|---|---|
| **BASIC source** in a `bp` file | what they are for |
| **VOC records** | fine — a VOC record is a few short fields (editing the VOC needs `admin`) |
| **Dictionary records** | fine for a simple one; see the limit below |
| **Data records with multivalues** | fine — see the tokens below |
| **Data records with subvalues** | fine — see the tokens below |

**A field is a line and that part needs no explanation.** SD writes the working
copy with one field per line, so moving between fields is moving between lines.

**A value mark is not a line, and neither is a subvalue mark.** Both are control
characters an editor cannot show, so each has a token you can type:

| Type | To get |
|---|---|
| `~~` | a **value** mark |
| `` ~` `` | a **subvalue** mark |

SD converts marks to tokens on the way into the editor and tokens back to marks
on the way out, so multivalues and subvalues are both ordinary text while you
are editing.

```
SMITH~~JONES~~BROWN
```

is a three-value field, and

```
RED~`BLUE~~GREEN
```

is two values, the first of which has two subvalues.

**A record that cannot be written this way is refused, not mangled.** Some
records would come back different from how they went in — one that already
contains `~~` as data, for instance, or one with a `~` sitting immediately
before a mark, where the tilde and the token run together. Before opening the
editor, SD converts the record and converts it back; **if the result is not what
it started with, the verb refuses and names `ed`**, which needs none of this.

**Text marks are not converted**, and are covered by the same refusal rather
than being left to surprise you.

**A compiled dictionary record is truncated to its first 15 fields** while
you edit it, and recompiled with `compile.dict` when you save.

### An editor is not a hole, but it is worth thinking about

**An editor can write anywhere its user can write.** It opens the record you
named, but nothing stops you then opening any other file your Linux user may
open. **That is not a gap in SD; it is what an editor is** — and it is exactly
the reach your own shell already has. See
[Security and the operating system](12a-security-and-the-operating-system.html).

Neither editor can run a command from inside SD's session, so neither is a
shell. **What they are is read and write access to the filesystem, with your own
Linux permissions.**

### Over ssh

**A terminal editor is the point of an ssh session.** An ssh session reaches SD
through a terminal like any other, and SD hands the editor that terminal rather
than reading it through a pipe.

**If an editor misbehaves over ssh and not at the console, that is worth
reporting** with the terminal you connected from.

**A session with no terminal is refused**: an API session or a piped script has
nowhere to draw a full screen.

The removed full-screen editors are a different matter: `update.record` and
`modify` are gone and are not coming back. See
[Not in SD Core](14-not-in-sd-core.html).

## Files

| | |
|---|---|
| **`create.file`** · **`delete.file`** · **`clear.file`** | the life of a file |
| **`configure.file`** | change a file's configuration |
| **`analyze.file`** | report on a file's internals |
| **`fstat`** | file statistics |
| **`hsm`** | hashed-file statistics monitoring |
| **`set.trigger`** | attach a trigger |
| **`compile.dict`** | compile the I-types of a dictionary |

`create.file` writes the file's entry into the VOC for you; that side effect needs no `admin`.

## Indexes

**`create.index`** · **`delete.index`** · **`build.index`** · **`list.index`**

## Bulk record editing

| | |
|---|---|
| **`copy`** | copy records |
| **`delete`** | delete records |
| **`reformat`** · **`sreformat`** | reformat |
| **`sort.item`** | sort |
| **`cname`** | rename a file or change a record's name |
| **`delete.common`** | clear a common block |

## What actually gates these

| | |
|---|---|
| Changing the VOC directly | `admin` first: `ed voc`, a program's `write` or `delete` to the VOC, `copy` into it, `.s` and `.d`. Everything SD writes to the VOC itself, as a side effect, is not gated |
| The global catalog | nobody, `admin` or not — `catalog global`, `delete.catalog` of a global entry, a write to `global.bp.out` or `gcat` |
| The deny list (managed mode) | `admin` or the global password unlocks what the server denied |
| File permissions | ordinary Linux file permissions on your own directory — see [Security](12-security.html) |
| Reaching the operating system | two switches, **off as delivered**: `sh` (the `sh` verb) and `os` (`OS.EXECUTE`, a path outside the account, the embedded Python, `micro`, and a path outside the account that `sed` is given); the administrator turns each on with `set.os.access` (`admin` first) — see [Operating system access](06b-operating-system-access.html) |

## Two things to know when you compile

**`basic` no longer creates an object file it can never open again.** Compiling
into a reused file name previously produced an object SD could not subsequently
open.

**Object code and the catalog are replaced on upgrade.** The compiled
programs, the messages, include records and VOC templates are all overwritten by
a new release — see [Upgrading and uninstalling](01a-upgrading-and-uninstalling.html).
**Anything in your own account — your `bp` file and everything compiled from it
— survives an upgrade untouched.**
