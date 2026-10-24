Title: SD VOC - Using and Managing the VOC
Subtitle: How the command processor reads the VOC, the verbs that manage it, and what account creation puts there.

This page continues [SD VOC - Structure and
Usage](32-sd-voc-structure-and-usage.html).

## How the command processor uses the VOC

The command processor is a BASIC program, `$CPROC`, and it runs a loop:
show any waiting messages, print the prompt, read a line, dispatch it,
repeat. The first word of the line is looked up in the account's `voc`
file.

### Name resolution

A name is tried **as typed, then lower case, then upper case** (and then
with each hyphen changed to a dot, in upper case and in lower case). This is
the change that lets a lower-case `voc`, `bp` and `newvoc` on disk
coexist with code and habits that still type names in upper case.

**Account names are the deliberate exception.** They are still folded
upward, which is what makes signing in case-insensitive.

### If the VOC has nothing, the private vocabulary is tried

If the account's VOC has a record `$private.voc` (type `X`, the name of a
file in field 2), a name that is in no VOC record is looked for in that file
next, as typed, then in lower case, then in upper case. The VOC always wins,
so the private vocabulary can add a command and never replace one the VOC
holds. See [SD VOC - A Private Vocabulary and Your Own Record
Types](32b-sd-voc-private-vocabulary-and-parser.html), which also describes
`$voc.parser`, the record that hands a VOC record of a type SD does not know
to a program of your own.

### If neither has it, the catalogue is tried

A name that is not in the VOC or the private vocabulary is looked for in the **private catalogue**
of the account and then in the **global catalogue**, `gcat`. A hit is
treated exactly as though the VOC had held `V` / `CA` / *name*. This is
why a program you have catalogued can be run by typing its name with no
VOC record of its own.

If neither has it, the command is refused by name.

## Verbs that manage the VOC

### set.file — write a Q-pointer

```
set.file account file.name pointer.name
```

Writes a Q-pointer into the VOC — field 1 `Q`, field 2 the account name,
field 3 the file name. After it, *pointer.name* is usable wherever a file
name is, and the data stays where it is.

**The account must be in the accounts register.** The name is folded to
upper case before the register is read. If it is not there you get
*Account name '...' is not in register* and nothing is written.

### update.accounts — refresh from NEWVOC

```
update.accounts
```

Re-runs `LOGIN`'s `update.voc` paragraph, which copies any changed
records from the system `NEWVOC` into the account's VOC. **Since 17 Aug
2026 this is incremental** — it re-copies only the records whose stamp has
changed, not the whole file. Every ordinary account gets the same `NEWVOC`
now, so `update.accounts` is only ever bringing an existing account's copy
up to date after a release changes something — there is no tier list left
for it to consult.

### copy — copy records into the VOC

```
copy from voc messages,myname
copy from src.file to dict tgt.file record.id
```

`copy` can write into the VOC the way it writes into any file. The
`from voc` form copies a VOC record under a new name, which is how an
alias is made without `alias`.

### delete — remove a VOC record

```
delete voc name
```

Removes the record. **`.d` *name* at the command prompt asks first**;
`delete voc name` does not.

### .s and .r — save and recall

```
.s name s e
.r name
```

`.s` writes stack lines *s* to *e* into the VOC as *name* — a sentence
when the range is one line, a paragraph when it is more. `.r` loads a
sentence or paragraph from the VOC into the stack. **The record type
follows from what you saved** rather than from anything you say.

### The LIST family

Ten verbs list VOC entries by type. They are all R-records into
`SD.VOCLIB`:

| Verb | Lists |
|---|---|
| `listf` | all files (F-records) |
| `listfl` | local files only |
| `listfr` | remote files only |
| `listk` | all keywords (K-records) |
| `listpa` | all paragraphs (PA-records) |
| `listph` | all phrases (PH-records) |
| `listq` | all Q-pointers |
| `listr` | all remote references (R-records) |
| `lists` | all sentences (S-records) |
| `listv` | all verbs (V-records) |

`listf`, `listfl` and `listfr` show the VOC name, the **file type**, the
description and the data and dictionary pathnames. The type is `DH` for a
dynamic hashed file (followed by `I` for alternate key indices, `S` for
statistics and `T` for a trigger function), `Dir` for a directory file, `Err`
and the error number for a file that will not open. **A multifile has a line
for each subfile**: the type column holds one value for each subfile, so that
each type is on the line of that subfile's pathname, and the first begins with
`Mult` (`Mult DH` on the first line, `Dir` on the second for a multifile of a
hashed and a directory subfile).

The three can take a `with` or `without` clause on **`trigger`** or
**`encryption`**, as `listf with trigger` for the files that have a trigger
function. SD has no file encryption, so `with encryption` selects nothing and
`without encryption` selects every file. Any other query processor words may be
added after them, as for every sentence. (SD Core for Linux also has a
`replication` word here; this product has no dictionary item for it yet, so
leave it out.)

### count — count the records

```
count voc
count voc with v = "V"
```

`count` over the VOC gives the total record count. With a selection
clause it counts by type or by any other field.

### ct — display a record

```
ct voc name
```

Displays the record field by field. This is the way to see what a VOC
entry actually contains without an editor.

## The VOC and the account

The installer copies the account's VOC from `newvoc` in the system directory —
**475 records**, counted directly on 21 Oct 2026. (SDSYS's own VOC, which nobody
signs in to, is built from `voc_template` instead — **480 records**, the same verbs plus a few
system file pointers.) `update.accounts`, which needs
`admin`, and an upgrade, add to the account's VOC whatever a new release ships;
neither takes anything away.

**Administration is in the verb, not in the VOC.** Your account has `config`,
`set.date`, `clean.account`, `list.readu`, `list.locks`, `lock`, `unlock`,
`listu`, `update.accounts` and the rest — and each one refuses with *Command
requires administrator privileges* until `admin`. There is no `create.account`,
`delete.account`, `modify.account`, `remote.ssh` or `remote.api` anywhere: an
unknown name is simply not recognised. See *Administrator commands* in the
GettingStarted set.

## Case on disk

The VOC ids are stored in lower case — `list`, `create.file` and so on. This changes nothing about what you type —
SD tries a name as typed, then lower, then upper. What it changes is what
SD prints back: `CT VOC LIST` answers `VOC list`.

The file-pointer entries — `bp`, `bp.out`, `gpl.bp`, `gpl.bp.out` — moved
to lower case on 19 Aug 2026. **The migration is complete, not partial**: an
account's own `voc`, and the system directory's `newvoc`, `accounts`,
`messages` and `syscom`, are all lower case on disk too, confirmed directly
— there is nothing left in upper case to migrate. `qfile` was never a path
of its own; it is a VOC pointer to `@SDSYS/$ipc`, the session IPC area, and
has no on-disk name to fold.

## What is not here

**There is no verb that edits the VOC directly.** `ED` edits any file,
and `ED VOC`*name* is how you edit a VOC record by hand. But there is no
verb whose purpose is to create or modify VOC entries — `set.file` writes
one kind, `.s` writes two, and everything else is done with `ED` or with
`copy from voc`. **On Solo every direct change to the VOC needs `admin`** —
`ed voc`, a program's `write` or `delete` to it, `copy` into it, `.s` and `.d`
with a name — and is refused with *The VOC can only be changed after admin*
without it. What SD writes to the VOC itself — `create.file`'s entry, the command
stack — is not gated.

**`PROC` is removed.** A `PQ`-type record is refused by name. Nothing
that ships is type `PQ`, so this can only be met in a VOC record somebody
wrote.

**`menu` records are dispatched but no menu ships.** Type `M` is handled
and there is no shipped example to look at.

**`EDIT` runs.** It was once exactly the kind of VOC entry described
above — present but undispatchable, its type field reading
`Verb - Full screen editor` rather than `V`. That was reversed 26 Aug
2026: it is a real `V` record now, a working full-screen editor. See [SD
VOC - Structure and Usage](32-sd-voc-structure-and-usage.html#what-the-shipped-voc-holds).

## See also

[SD TCL - The Command Processor](19-sd-tcl-command-processor.html) ·
[SD TCL - Files and Records](20-sd-tcl-files-and-records.html) ·
[SD Basic - File Handling](07-sd-basic-file-handling.html) ·
[SD Basic - Dynamic Arrays](05-sd-basic-dynamic-arrays.html).
