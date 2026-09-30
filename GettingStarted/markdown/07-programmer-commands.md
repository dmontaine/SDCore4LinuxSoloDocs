Title: Development and file commands
Subtitle: Compiling, editing, and the verbs that maintain files, indexes and records in bulk.

**Your account has all of these, from the moment it is installed.** SD Core
used to hold them back from a *standard* account; that split is gone, and Solo
has one account — see [The account and its passwords](05-account-types.html).
This page is a reference for what each of them does, not a list of what you have
been given.

**Two things, named as they come up below, still need more than the verb**:
cataloguing globally is refused to everyone, and editing the VOC directly needs
`ADMIN` — see [What actually gates these](#what-actually-gates-these) at the foot
of this page. The full-screen editors need nothing extra.

## Compile, catalogue and run

| | |
|---|---|
| **`basic`** | compile SD BASIC source |
| **`catalog`** · **`catalogue`** | add to the catalogue |
| **`delete.catalog`** · **`delete.catalogue`** | remove from it |
| **`compile.dict`** | compile dictionary items |
| **`run`** | run a compiled program |
| **`map`** | show a program's map |
| **`generate`** | generate source |
| **`phantom`** | start a background process |

> **Nobody catalogues globally.** `catalog ... global`, and `delete.catalog` of a
> global entry, are refused whoever asks and whether or not `ADMIN` is unlocked.
> On a managed computer the global catalogue is the SD Core for Linux server's
> — see [Managed mode](15-managed-mode.html). Private and local cataloguing
> work as they always did: catalogue your own programs with `catalog ... local`.

## Edit and debug

| | |
|---|---|
| **`ed`** | the line editor. Needs nothing installed. `edit` is an alias for it |
| **`nano`** | a **full-screen** editor — opens the record in `nano` |
| **`micro`** | a **full-screen** editor — opens the record in `micro` |
| **`debug`** | the BASIC debugger |
| **`pstat`** · **`pdebug`** · **`pdump`** · **`dump`** | process introspection |

### Editors

**`edit` is not a full-screen editor here** — a real difference from SD Core for
Windows, where it opens Microsoft Edit. There is no Linux equivalent to alias it
to, so `edit` runs the **line** editor, `ed`, instead. The two full-screen
editors are named for the program each runs:

| | |
|---|---|
| **`nano`** | ships with most distributions already |
| **`micro`** | installed by the SD installer as an ordinary package |

```
nano  bp myprog
micro bp myprog
nano  dict customers name
```

Either verb writes the record to a working copy, opens the editor on it, reads
it back, and asks whether to save. For a `bp` record it then offers the compile
and the catalogue.

**Both are terminal editors**, so both work over ssh as well as at the console.

**`micro` highlights SD BASIC.** The verb copies SD's syntax file into your
`~/.config/micro/syntax` the first time it is used. **`nano` does not yet**: the
multiuser product installs a system-wide `sdbasic.nanorc`, and Solo, which
writes nothing outside your home directory, does not. **It applies to a `bp`
record and to nothing else.** SD names the working copy so the editor can
recognise the language — a record edited out of any other file is treated as
plain text, which is correct for a VOC entry or a data record.

**`ed` is unaffected and is still there.**

### What the editors are good for, and what they are not

**They are text editors**, so they suit a record whose content is lines of text:

| | |
|---|---|
| **BASIC source** in a `bp` file | what they are for |
| **VOC records** | fine — a VOC record is a few short fields (editing the VOC needs `ADMIN`) |
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

**A compiled dictionary record is truncated to its first 15 fields** while you
edit it, and recompiled with `cd` when you save.

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
than reading it through a pipe. **If an editor misbehaves over ssh and not at
the keyboard, that is worth reporting** with the terminal you connected from.

**A session with no terminal is refused**: an API session or a piped script has
nowhere to draw a full screen.

The removed full-screen editors are a different matter: `sed`, `update.record`
and `modify` are gone and are not coming back. See
[Not in SD Core](14-not-in-sd-core.html).

## Files

| | |
|---|---|
| **`create.file`** · **`delete.file`** · **`clear.file`** | the life of a file |
| **`configure.file`** | change a file's configuration |
| **`analyse.file`** · **`analyze.file`** | report on a file's internals |
| **`fstat`** | file statistics |
| **`hsm`** | hashed-file statistics monitoring |
| **`set.trigger`** | attach a trigger |
| **`cd`** | change directory |

## Indexes

**`create.index`** · **`delete.index`** · **`build.index`** · **`make.index`** · **`list.index`**

## Bulk record editing

| | |
|---|---|
| **`copy`** · **`copyp`** | copy records |
| **`delete`** | delete records |
| **`rename`** | rename records |
| **`reformat`** · **`sreformat`** | reformat |
| **`sort.item`** | sort |
| **`cname`** | change a record's name |
| **`delete.common`** | clear a common block |

## What actually gates these

| | |
|---|---|
| Changing the VOC directly | `ADMIN` first: `ed voc`, a program's `write` or `delete` to the VOC, `copy` into it, `.s` and `.d`. Everything SD writes to the VOC itself, as a side effect, is not gated |
| The global catalogue | nobody, `ADMIN` or not — `catalog global`, `delete.catalog` of a global entry, a write to `global.bp.out` or `gcat` |
| The deny list (managed mode) | `ADMIN` or the global password unlocks what the server denied |
| File permissions | ordinary Linux file permissions on your own directory — see [Security](12-security.html) |
| Reaching the operating system | none, for `sh`, `OS.EXECUTE`, or either editor |

## Two things to know when you compile

**`basic` no longer creates an object file it can never open again.** Compiling
into a reused file name previously produced an object SD could not subsequently
open.

**Object code and the catalogue are replaced on upgrade.** The compiled
programs, the messages, include records and VOC templates are all overwritten by
a new release — see [Upgrading and uninstalling](01a-upgrading-and-uninstalling.html).
**Anything in your own account — your `bp` file and everything compiled from it
— survives an upgrade untouched.**
