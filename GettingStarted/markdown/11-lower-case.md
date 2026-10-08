Title: Lower case
Subtitle: Commands, file names and record ids in SD's own files are lower case — and nothing you type has to change.

**Everything that can be lower case is lower case, completely — no name exists
in two casings.** SD used to be inconsistent about it: BASIC source is free-form
and usually written lower case, while file names and field names were forced up.

**What you type does not change.** `LIST`, `list` and `List` all run the same
verb, and so does every keyword — `with`, `by`, `no.page` and the rest.

## The lookup rule

SD tries a name **as you typed it, then in lower case, then in upper case**.

```
as typed  →  lower  →  upper
```

A name that matches exactly still wins, so nothing that works today changes. A
name that exists in **no** case is still reported as not found.

This order is used everywhere: the parser, the query processor, `run`,
multifile resolution, the `login` paragraph lookup, and `set.file`'s default
`qfile` pointer.

## What is spelled in lower case, and that is the whole of it

| | |
|---|---|
| Commands in the VOC | **`list`**, `count`, **`select`**, **`create.file`**, **`setptr`** … all 414 `newvoc` records and all 422 `voc_template` records — and that is how they appear in `list voc`, `listv` and `ct voc` |
| BASIC source and include records | 215 `gpl.bp` records, 16 `syscom` includes — all of them |
| System files on disk | `accounts`, `bp`, `bp.out`, `gpl.bp`, `messages`, `newvoc`, `pcode.out`, `syscom`, `voc`, `voc.dic`, `voc_template` and the rest — 0 of 15 directory names still upper case |
| Account names on disk | `user_accounts/sduser`, `$cred/sduser` |
| Files in a new account | created with lower-case names on disk |

**This is a completed migration, not an additive fallback that leaves old
spellings behind it** — every name in every ruled category is lower case,
checked directly against the source tree (`gplbld/verify-nocase.py`, part of this
project's own build checks), not merely claimed.

## Global catalogue names are the exception

**The names of programs in the global catalogue are upper case** — `$LOGIN`,
`!set.password`, and on a managed computer the server's programs, catalogued as
`*NAME` by `sync.global.catalog`. `catalog` upper-cases a global name, and the
loader looks it up upper-cased, so a program that `CALL *zzgsub` finds the entry
`*ZZGSUB`. You type them in either case; nothing you write changes.

## Record ids in your own data files are untouched, deliberately

**`ext4` is case sensitive — a real difference from NTFS, which is not.** On a
Windows computer `SUE` and `sue` are already the same file, so lower-casing an
application's own data cost nothing there. On Linux they are two different
files, so forcing an application's own record ids to lower case would silently
change its data. **This is ruled out of scope on purpose**: the lower-case
migration covers SD's own system names — commands, VOC entries, source — never
the records an application stores in its own files. What you put in your own data
keeps whatever case you gave it.

**`create.file`** also takes a `no.case` option, which creates a file whose
record ids are treated as case insensitive regardless: SD writes records
preserving the casing given by whatever performs the write, and reads locate
records regardless of casing. That is an opt-in choice for a file where you want
it, not the system default.

## One correction worth reading

**`list` and `ct` used to disagree about the same name.** `list voc $hold`
answered *"'$HOLD' not found"* on the very record `ct voc $hold` had just shown
you. `list`, `sort`, `select` and the rest of the query language now use the same
as-typed → lower → upper order as everything else.

## Two related refusals that no longer depend on case

**`delete.file`**'s refusal to delete `voc` and `$acc` no longer depends on the
case you type. It could not be got round before, because those names were upper
case — **it could have been, once they are not.**
