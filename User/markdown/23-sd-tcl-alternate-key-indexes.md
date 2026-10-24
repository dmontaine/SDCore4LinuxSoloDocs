Title: SD TCL - Alternate Key Indexes
Subtitle: Indexing a field so records can be found by its contents, and the step that is easy to skip.

An alternate key index lets SD find records by what is in a field instead of by
record id. Five verbs make, populate, inspect and remove them.

SD folds case, so a command may be typed in either case. Commands are shown here
in lower case. In the tables, *italics* mark something you supply and **bold**
marks a word typed as it stands; braces mark an optional part.

> **Every listing on this page was produced by running it**, on SD Core for
> Linux, against a six-record file whose field 1 held five `V` and one
> `K`.

## Use `make.index`

**That is the whole of this page's advice.** `create.index` and `build.index`
are two halves of what `make.index` does in one command, and **the half that is
easy to miss is the second one**:

```
make.index zzak f2
```

```
Added index for F2
Building index 'F2'...
6 records processed
Populating index...
```

Use the two-step form only when you have a reason to separate them.

## The five verbs

| | |
|---|---|
| **`create.index`** *file* *field*… | define an index. **Does not populate it** |
| **`build.index`** *file* *field*… \| **all** | populate it from the records already there |
| **`make.index`** *file* *field*… | both, in one command |
| **`list.index`** *file* *field*… | what indexes the file has, and whether they work |
| **`delete.index`** *file* *field*… \| **all** | remove them |

`create.index` and `make.index` also take:

| | |
|---|---|
| **`no.nulls`** | do not index records whose field is empty |
| **`no.case`** | case insensitive: keys that differ only in case ('Apple', 'apple', 'APPLE') are one key, and a lookup of any of them finds all three records |
| **`pathname`** *path* | put the index somewhere other than beside the file |

## Why the two-step form catches people

`create.index` records the definition and **walks nothing**:

```
create.index zzak f1
```

```
Added index for F1
```

That reports success, and the index it made matches no records at all. The
proof is in `list.index`:

```
list.index zzak f1
```

```
Alternate key indices for file zzak
Number of indices = 1
Index name...... En Type Nulls S/M Fmt Field/Expression
F1                N  D    Yes   S   L  1
```

**The `En` column is the one to read. `N` means the index is not enabled** —
it exists, it is listed, and it will not find anything. After `build.index` the
same line reads `Y`:

```
F1                Y  D    Yes   S   L  1
```

**So `list.index` is the check**, and *"the index is there"* is not the same
question as *"the index works"*.

## `build.index` needs the file to itself

```
build.index zzak f1
```

```
Cannot gain exclusive access to file
```

**That is what you get If anything has the file open — including the command
you just typed.** `create.index` followed by `build.index` **in the same
session** fails this way every time, because `create.index` leaves the file
open. From a fresh session it succeeds:

```
Building index 'F1'...
6 records processed
Populating index...
```

**`make.index` does not have this problem**, which is the practical reason to
prefer it.

> **A session that was killed rather than logged out also holds the file.**
> Its entry stays in the user table, `listu` still lists it, and `build.index`
> is refused with the message above until it is cleared. If a build is refused
> and you are certain nothing is using the file, run `listu` before looking for
> anything more exotic.

## Asking what a file has

```
list.index zzak f1 f2
```

```
Alternate key indices for file zzak
Number of indices = 2
Index name...... En Type Nulls S/M Fmt Field/Expression
F1                Y  D    Yes   S   L  1
F2                Y  D    Yes   S   L  2
```

| column | |
|---|---|
| `En` | enabled — **`Y` only after a successful build** |
| `Type` | `D` for a field, `I` for a computed expression |
| `Nulls` | whether empty values are indexed — `no.nulls` at creation is what changes it |
| `S/M` | single- or multi-valued |
| `Fmt` | the key's format code, as a dictionary entry carries it |
| `Field/Expression` | the field number, or the I-type expression |

A file with none says so plainly:

```
File has no indices
```

## Using an index

**Nothing changes in how you write a query.** An index makes a selection on that
field faster; it does not add syntax and it does not change the answer:

```
count zzak with f1 = "V"
```

```
5 record(s) counted
```

That is the same command, and the same result, as before the index existed.
**Index the fields you actually select on** — each index makes every write to
the file a little more expensive, and one that is never used in a `with` clause
costs and returns nothing.

Once built, an index maintains itself: ordinary writes and deletes keep it
current with no further action. Reading an index from a program — `selectindex`,
`indices()` — is in
[SD Basic - Alternate Key Indexes](09-sd-basic-alternate-key-indexes.html),
which also has the detail on why **an already-open file variable never learns
about a new index**.

## Removing an index

```
delete.index zzak all
```

```
Deleted index F1
Deleted index F2
```

`all` removes every index on the file. Naming them individually also works —
with one catch:

> **`delete.index` matches the index name exactly, and index names are held in
> upper case.** `delete.index zzak f1` answers *"Unrecognised index name
> (f1)"* — on a file where `list.index zzak f1` had just found the index and
> printed it. **`list.index` folds case and `delete.index` does not**, so a name
> that lists perfectly well may still refuse to delete. Use **`all`**, or type
> the name exactly as `list.index` prints it.

Deleting the file removes its indexes with it.

## Four more verbs

`analyse.index` says how an index is doing, `disable.index` switches it off for a
bulk load, `rebuild.all.indices` rebuilds every index the account has, and
`selectindex` reads an index from the command line. The listings in this
section were produced by running them in a sandbox build of SD Core for Linux
Solo, as the administrator, against a file `T3` of five records whose field 2
holds a city (`Oslo` twice, `Rome` twice, `Bern` once) with an index `CITY` on
it.

### How an index is doing: `analyse.index`

```
analyse.index file {index.name {statistics} {detail}} {lptr {n}} {no.page}
```

With the file name alone, every index of the file with its number of indexed
values and the most records any one value points at:

```
analyse.index T3
```

```
Analysis of the indices of file T3

Index name...... Indexed values  Most records for one value
CITY                           3                     2
```

With an index name it reports on that index. `statistics` is the default:

```
analyse.index T3 CITY
```

```
Index CITY of file T3

Index entries      Key values     Min Recs       Avg Recs       Max Recs
            5               3            1         1.6667              2

Records for a value        Values
             1         1
        2 to 3         2
```

*Index entries* is the number of (value, record) pairs, *Key values* the number of
different values, and the last table says how the values spread over the number
of records they point at. `detail` lists every indexed value with its number of
records, and both may be given:

```
analyse.index T3 CITY detail
```

```
Index CITY of file T3

Records...  Key value......................................................
         1  Bern
         2  Oslo
         2  Rome
```

`lptr` sends the report to a print unit and `no.page` shows it without pausing
at each page. **The node settings and the sizes of the nodes are not reported:**
SD's indices are not read at that level from BASIC, and `list.index detail` or
`statistics` is the place for the rest. An index name that is not there is
*Unrecognised index name (NOSUCH)*, and no file name at all is *File name
required*.

### Switching an index off: `disable.index`

```
disable.index file index ...
disable.index file all
```

For a large load of records, or a batch run that would change most of them. The
index is **not kept up as records are written, and the query processor does not
use it**.

```
disable.index T3 CITY
```

```
Disabled index CITY
```

`list.index` then shows it with `N` in the `En` column. A query on the field still
answers, from the records (`list T3 with CITY = "Oslo"` gave the same two records
with the index disabled as with it on). **`build.index` builds the index again and
switches it on**; until then its contents are out of date. `selectindex` reads
whatever the index holds, enabled or not, so on a disabled index it may be wrong.

Without an index name `disable.index` asks for one (*Index name:*); a name that is
not there is *Unrecognised index name (NOSUCH)* and a file that is not there is
*File not found*. **Like `delete.index` it needs the file to itself**: another
session that had the file open would go on using, and keeping up, an index this
one had switched off.

### Rebuilding every index: `rebuild.all.indices`

```
rebuild.all.indices {no.query}
```

Rebuilds the indices of **every file your VOC points at with an `F` entry and that
has indices**, as `build.index` *file* `all` would, file by file. A file that cannot
be opened, is not a dynamic file or has no indices is passed over. It asks first,
and the answer is no unless you type `y`:

```
Rebuild the indices of all files (y/<n>)?
```

`no.query` goes ahead without asking:

```
rebuild.all.indices no.query
```

```
Indices of t3 rebuilt
1 file(s) had their indices rebuilt
```

**Read the answer with care.** `build.index` needs a file to itself (see above), and
`rebuild.all.indices` **does not look at what `build.index` said** — a file that
another session has open is refused by `build.index` and is still reported as
rebuilt. Make sure nothing else has the files open, and check a file you doubt with
`list.index`.

### Reading an index from the command line: `selectindex`

```
selectindex {dict} file index {value} {to n}
```

The command form of the `selectindex` statement in
[SD Basic - Alternate Key Indexes](09-sd-basic-alternate-key-indexes.html). It
builds list 0 (or list *n* with `to`) from an index of a dynamic file.

| | The list holds |
|---|---|
| no *value* | **every value in the index**, one to a list entry |
| a *value* | the **ids of the records** that have that value |

```
selectindex T3 CITY Oslo
```

```
2 record(s) selected to select list 0
```

That list is `a1` and `a3`, the two records whose city is `Oslo`. Without the
value:

```
selectindex T3 CITY
```

```
3 record(s) selected to select list 0
```

holds `Bern`, `Oslo` and `Rome`: the values, which are not record ids, so a
`list T3` straight after it answers `'Bern' not found` and the like. **The value
is matched as it is stored in the index (its internal form), as the statement
does.** The index name is looked up as typed, then in lower case, then in upper
case, so `selectindex T3 city` works. A name that is not there is
*Unrecognised index name (NOSUCH)*, and no index name at all is *Index name
required*.

## Who has these verbs

**Every account has all nine now.** An index changes how a query runs, not
how data is written, so there was never a reason to withhold them even
under the old model. The five on the table above, and `analyse.index`,
`disable.index`, `rebuild.all.indices` and `selectindex`.

## See also

[SD Basic - Alternate Key Indexes](09-sd-basic-alternate-key-indexes.html) ·
[SD TCL - Files and Records](20-sd-tcl-files-and-records.html) ·
[SD TCL - The Query Processor](21-sd-tcl-query-processor.html).
