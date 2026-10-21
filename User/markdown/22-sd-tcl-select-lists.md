Title: SD TCL - Select Lists
Subtitle: Choosing a set of records once and then using it, saving it, and combining it with another.

A select list is a list of record ids that SD is holding for you. You build one
with `select`, and **the next command you type works on those records instead of
the whole file**. That one sentence is most of this page.

SD folds case, so a command may be typed in either case. Commands and keywords
are shown here in lower case. In the tables, *italics* mark something you supply
and **bold** marks a word typed as it stands; braces mark an optional part.

> **Every listing on this page was produced by running it**, on SD Core for
> Linux, in an administrator account. The file being queried is `voc`,
> because it is the one file every account has.

## The one thing to understand first

```
select voc with dispatch = "OS"
```

```
2 record(s) selected to list 0
```

Now the file has 418 records and the list has 2. **The next command sees 2:**

```
count voc
```

```
2 record(s) counted
```

**That is the point of select lists and it is also the trap.** A list you
built and forgot about silently narrows the next thing you do, and nothing
about the answer says so — `2 record(s) counted` is not obviously wrong. **A
surprisingly small answer is very often a leftover list.**

**The list is consumed by the command that uses it.** Run `count voc` again and
you get 418, because the list is gone.

### The prompt tells you

While a list is active the command prompt gains a second colon:

```
:select voc with dispatch = "OS"
2 record(s) selected to list 0
::
```

**`::` means a list is waiting.** `:` means there is not one. This is the
quickest way to answer *why did that only find two records* before you go
looking for a bug.

`clear.select` throws lists away without using them:

| | |
|---|---|
| **`clear.select`** {*n*} | discard list *n* |
| **`clear.select all`** | discard all of them |

```
clear.select all
```

```
Cleared all numbered select lists
```

`clearselect` is the same verb, spelled without the dot.

## Building a list

| | |
|---|---|
| **`select`** *file* {*selection*} | the records that match |
| **`sselect`** *file* {*selection*} | the same, sorted |
| **`show`** *file* {*selection*} | choose records interactively |
| **`qselect`** {**dict**} *file* {**saving** *n*} | a list of **field values** rather than record ids |
| **`nselect`** *file* | keep only the ids of the active list that are **not** in *file* |
| **`form.list`** {**dict**} *file* *record* | take the list from a record's contents |

`select` and `sselect` take the whole selection language of the query processor
— `with`, the operators, `by` — so anything you can list you can select. See
[SD TCL - The Query Processor](21-sd-tcl-query-processor.html).

**`qselect` reads a field out of each record and makes a list of those values.**
It is how you follow a reference from one file into another: select the orders,
`qselect` the customer number, and the list you are left with is customers.

> **`qselect` does not tell you which list it filled.** Its completion message
> ends after the words *select list* with no number:
> `14 record(s) selected to select list`. **The list is there** — it is list 0
> unless you gave `to` *n*. `select` and `nselect` both print the number
> correctly, so do not read this as the list being missing.

## Numbered lists

There are **eleven**, numbered **0 to 10**. List 0 is the default and the one
every verb uses when you do not say otherwise. `to` *n* puts the result
somewhere else, which is how you keep two lists at once:

```
select voc with dispatch = "OS" to 3
select voc with dispatch = "IN" to 4
```

```
2 record(s) selected to list 3
45 record(s) selected to list 4
```

**A list in 1 to 10 does not affect the next command.** Only list 0 does. That
is what makes the numbered lists a safe scratch area, and it is why the prompt
above stayed `:` rather than becoming `::`.

## Saving a list by name

Numbered lists last as long as your session. A **saved** list is a record in the
account's `$savedlists` file and outlives it.

| | |
|---|---|
| **`save.list`** {*name*} {**from** *n*} | save list *n* as *name* |
| **`get.list`** {*name*} {**to** *n*} | load it back into list *n* |
| **`delete.list`** *name* | delete the saved list |
| **`copy.list`** *name* {`,`*new*} {**from** *file*} {**to** *file*} | copy one, or `*` for all |

Both default to list 0, and `save.list` defaults the name to the list number.

```
select voc with dispatch = "OS"
save.list zzos
```

```
2 record(s) selected to list 0
2 records saved to select list 'zzos'
```

```
get.list zzos
delete.list zzos
```

```
2 record(s) selected to select list 0
Deleted saved select list 'zzos'
```

**`copy.list` moves a saved list between accounts**, because `from` and `to` name
the `$savedlists` file to read and write. Within one account it renames or
duplicates.

## Combining two lists

**There are two sets of verbs and they take different things. this is the
easiest mistake on the page.**

| | takes | |
|---|---|---|
| **`list.union`** · **`list.inter`** · **`list.diff`** | **saved list names** | reads and writes `$savedlists` |
| **`merge.list`** | **list numbers** | works on the numbered lists in your session |

Giving a number to `list.union` does not do the numbered thing — it looks for a
saved list with that name and fails:

```
list.union 3 4 5
```

```
First source list does not exist
```

### By name

```
list.union zzos zzin zzboth
list.inter zzos zzin zzover
```

```
47 records saved to select list 'zzboth'
0 records saved to select list 'zzover'
```

Both write a **saved** list, not a numbered one, so `get.list` is how you pick
the result up. The two answers above check each other: the sources hold 2 and 45
records with nothing in common, so the union is 47 and the intersection is 0.

### By number

```
merge.list 3 union 4 to 5
merge.list 3 intersect 4 to 6
```

```
47 record(s) selected to select list 5
0 record(s) selected to select list 6
```

The operator words are **`union`**, **`intersect`** (or `intersection`) and
**`diff`** (or `difference`). `to` *n* names the target, defaulting to list 0,
and `count.sup` suppresses the count line.

`list.diff` and `merge.list ... diff ...` give the records in the first list that
are not in the second, so **the order of the two arguments matters** where it
does not for union and intersection.

## Going through a list by hand: `show.list`

```
show.list {n}
```

Shows the items of an active numbered list (list 0 if you give no number) a page
at a time, each with its number and **an asterisk when it is selected**. All
items start selected. You clear and set the asterisks; when you quit, **the list
is made again from the items that still have one**.

```
select t3
show.list
```

```
    1 * a1
    2 * a2
    3 * a3
    4 * a4
    5 * a5
: c 3-4
    1 * a1
    2 * a2
    3   a3
    4   a4
    5 * a5
: s 3
    1 * a1
    2 * a2
    3 * a3
    4   a4
    5 * a5
: q
```

| You type | |
|---|---|
| `t` | the top of the list |
| `n` or Return | the next page |
| `p` | the previous page |
| `s` *item* · *item* | select the item (the `s` may be left out) |
| `c` *item* | clear the item |
| `q` | quit, keeping the items that have an asterisk |
| `qc` | quit, clearing every item |
| `?` | help |

An *item* is a number, a range *a*`-`*b* (no spaces round the hyphen), `visible`
(the items on this page) or `all`, and `visible` and `all` may be shortened to
`v` and `a`. Several may be given, separated by spaces or commas: `1,4,8-11`. The
space after `s` and `c` is optional.

After the `q` above, the list holds `a1`, `a2`, `a3` and `a5`, and the next
`list t3` shows those four. **`qc` leaves no list at all**, so the command after
it works on the whole file, not on an empty list. A list that is not active is
refused (`show.list 7` answers *Select list 7 is not active*).

## Sorting a saved list: `sort.list`

```
sort.list list.name {from file.name}
```

Sorts a saved list into ascending order **as strings**, in place: `10` comes
before `9`. The list is in `$savedlists` unless `from` names another file.

```
sort.list names1
```

```
5 records saved to select list 'NAMES1'
```

`@system.return.code` is the number of items. A list that is not there is
*Saved select list 'NOSUCH' not found*. The file named by `from` is held to the
same rule as for `copy`: a VOC can be changed only after `admin`, and the global
catalogue's directories only with the global password.

## Moving a list to another number: `move.list`

```
move.list src to tgt {keep}
```

*src* and *tgt* are select list numbers, 0 to 10. The list in *src* moves to
*tgt* and *src* is empty; with `keep` the source stays as it was, so it is a
copy. It prints nothing when it works. A number outside 0 to 10 is refused
(*Select list number missing or invalid*). It is the command form of the
`movelist` statement.

## Copying saved lists, Pick style: `copy.listp`

```
copy.listp {{dict} src.file} {id.list | *} {(options}
```

`copy.listp` is `copyp` (described with the other record verbs in
[SD TCL - Files and Records](20-sd-tcl-files-and-records.html)) with **the saved
lists as its source file**: unless a file is named, the source is `$savedlists`
and every word is the name of a saved list. `*` means every list. Like `copyp`
it asks where the copy is to go and shows what it did:

```
copy.listp rome
```

```
To: rome3
Copied item ROME from $savedlists to $savedlists item ROME3
1 record(s) copied.
```

The options are `copyp`'s, in a trailing bracket: `d` deletes the source (the
line then reads *Moved item ...* and the count *copied and deleted*), `i`
suppresses the id, `n` no pagination, `o` overwrites, `p` and `t` send the
listing of the list to the printer or the terminal (`copy.listp rome (t` prints
`Item ROME` and its ids), `s` suppresses field numbers. **Without `o`, a list
that already exists at the destination is not copied** (*Item NAMES1 already
exists in target file - Not copied*).

## A command for each id: `execute.list`

```
execute.list command {; command ...}
```

Runs the commands once for each id of the default select list (list 0), in
order. **A pair of empty round brackets in a command is replaced by the id.**
Commands are separated by semicolons. The ids are all taken out of the list
first, so a command that itself uses list 0 does not take ids from the loop. With
no active list nothing is run.

```
select t3 with city = "Rome"
execute.list display found () ; display again ()
```

```
found a2
again a2
found a4
again a4
```

A line with nothing after the verb is refused (*A command is needed after
EXECUTE.LIST*).

The listings in the sections from `show.list` to here were produced by running
them in a sandbox build of SD Core for Linux Solo, as the administrator. What is
read from the source and was not run: the guard on the file `sort.list` is
given with `from`, and the refusal of an `execute.list` with no command.

## Who has these verbs

**All of them are in every account.** Building, saving and combining lists
is reading, and nothing on this page changes a record. `show.list`,
`sort.list`, `move.list`, `copy.listp` and `execute.list` are in every account
too; `execute.list` changes records only if the commands it runs do.

## See also

[SD TCL - The Query Processor](21-sd-tcl-query-processor.html) ·
[SD TCL - Files and Records](20-sd-tcl-files-and-records.html) ·
[SD Basic - Select Lists](08-sd-basic-select-lists.html).
