Title: SD TCL - Files and Records
Subtitle: Making, configuring and removing files; copying, renaming, deleting and dumping the records inside them.

This page covers the verbs that act on a whole file or on the records in it,
from the command line. What a program does with the same files is
[SD Basic - File Handling](07-sd-basic-file-handling.html); this is the part you
type.

SD folds case, so a command may be typed in either case. Commands are shown here
in lower case, which is what this port uses on disk. In the tables, *italics*
mark something you supply and **bold** marks a word typed as it stands; braces
mark an optional part.

## The two kinds of file, and what they look like on disk

**Both kinds are a Linux directory.** This surprises people who expect a
database file to be one file.

| | |
|---|---|
| **dynamic** | a directory holding components named `%0`, `%1`, … The records are hashed across them and none of it is readable from outside SD |
| **directory** | a directory holding **one ordinary file per record**, named by the record id |

`voc`, `$ipc` and `dict.dic` are dynamic. `messages`, `accounts` and `bp` are
directory files — which is why you can open a BASIC program in any text
editor and why `gpl.bp` is readable in the source tree.

**A dictionary is a separate file that travels with the data file.** Verbs that
can act on either take `dict` in front of the file name; with no `dict` they
mean the data portion.

**The shipped file names are lower case on disk in this port, and unlike SD
Core for Windows, that rename is load-bearing rather than cosmetic.**
`voc`, `bp`, `newvoc`, `accounts`, `messages` and the rest were renamed —
NTFS is case-insensitive and would have matched the old spelling anyway,
but `ext4` is case sensitive, so a mismatch here would have been a real
file-not-found rather than an invisible non-issue. What you can type is
unaffected either way (SD tries as-typed, then lower, then upper), and
the lower-case spelling is what you will see in the path text SD reports
back.

## Making and removing files

```
create.file {dict|data} voc.name directory
create.file {dict|data} voc.name dynamic {parameters}
create.file voc.name using dict filename
```

`create.file` makes the file **and** the VOC record that names it. Without
`dict` or `data` it makes both portions. `using dict` *filename* takes the
dictionary from an existing file instead of creating an empty one, and
`no.query` suppresses the confirmation.

Dynamic files take parameters, all optional:

| | |
|---|---|
| `minimum.modulus` *n* | the floor the file will not shrink below |
| `group.size` *n* | group size in 1 KB units |
| `large.record` *n* | the size above which a record is stored separately |
| `split.load` *n* · `merge.load` *n* | the load factors that trigger growth and shrink |
| `version` *n* | file format version |
| `directory` *path* | put the file at *path* instead of in the account |
| `no.case` | record ids are case-insensitive |
| `no.resize` | disable automatic resizing |

```
delete.file {dict|data} voc.name {force} {no.query}
clear.file {dict|data} file
```

`delete.file` removes the file and its VOC record. **`force` is needed if the
file is not empty.** `clear.file` empties a file and keeps it, along with its
dictionary, its indexes and its `creating.seqkey` counter (the next sequential id carries on; this is
SD's behaviour, and `configure.file` *file* `next 1` starts it again).

## Changing a file after it exists

```
configure.file {dict} voc.name {parameters}
configure.file {dict} voc.name default
```

The parameters are the `create.file` ones, plus `immediate` to do the work now
rather than lazily, and `dynamic` or `directory` to change the file's kind.
`default` puts the settings back.

**Two of them restructure the whole file.** Changing `group.size` or `version`
requires a full restructure; the rest take effect without one. Plan for the time
and the disk on a large file.

**`reporting`** shows how far the work has got: a line *1000 records converted* at every thousand
and the total at the end, when `dynamic` or `directory` converts the file, and *n groups moved* in the same way
for `immediate`. It changes nothing else, and without it nothing is shown.

`no.case` / `case` and `resize` / `no.resize` are the pairs that turn a setting
on and off. For a directory file, `binary` turns on mark mapping.

**`next` *n*** sets the next record id that `write … creating.seqkey` will give out in a
**hashed** file (see [SD Basic - File Handling](07-sd-basic-file-handling.html)); **`next query`** says
what it is (*The next sequential record id is 103*). A new file's counter is 1, `clear.file` keeps it (SD's behaviour), and rebuilding a file
with a different group size or version carries the counter over. A file made by `copy` starts its own
counter at 1; `write … creating.seqkey` skips an id that already names a record. A directory file is refused:
*NEXT applies to a hashed file only*.

## Looking at a file

```
analyse.file {dict} file {statistics} {lptr}
```

Without `statistics` it reports the shape: the account, the file name, the path,
the type, the group size, the large-record size, the minimum and current
modulus, the load factors and the size in bytes. With `statistics` it walks the
whole file and adds record counts, per-group minimum, maximum and average, and a
size distribution. **`statistics` reads every group**, so it is not free on a big
file.

`analyze.file` is the same verb under the other spelling.

```
fstat filename on            start collecting, clearing the counters
fstat filename {lptr}        show what has been collected
fstat filename off           stop collecting
fstat global {lptr}          the global counters
fstat reset                  clear the global counters
fstat                        the periodic global summary
```

**`fstat` only works on dynamic files.** A directory file named on the command
line gets a warning; one arriving through a select list is skipped silently.
Where a file name is taken, a select list or several names may be given instead.

```
hsm {on|off|display} {user n}
```

The Hot Spot Monitor. `display` is the default. It counts each program's
opens, reads, writes and deletes as well as its calls and processor time, and
the report has a column for each. **What a program asks for through the `open`
of a name (its VOC lookup), or through `readv` and `writev`, which read and
write the record themselves, is not counted against it.**

```
profiler on {user n} {event ...}
profiler off {user n}
profiler {display} {user n}
```

The application event profiler: where the Hot Spot Monitor counts per program,
this records **what** the programs did and **where**. `profiler on` starts
logging in your session, or, for the administrator, in session *n*; `profiler off`
stops it and keeps what was found; `profiler` (or `display`) shows it. The events
are `call` (a subroutine), `open`, `read` (including `readu`, `readv` and
`matread`), `write`, `delete`, `openseq`, `readseq`, `writeseq`, `execute` (the
first 32 characters of the command) and `object` (a call of a class module's
object). Name some after `on` to log only those; none means all of them.

`display` is a list of the event types with a count of each. Type the **number** of
a line to open it: the names the events were on (the file, the subroutine, the
command), then the programs that did them, then the **line numbers** in the
program. **`b`** goes back a level and **`q`** leaves. (openQM uses the cursor
keys; a number works on any terminal.) With one profile available it opens
without asking which.

**What it is and is not.** The profile is kept in memory while it is on, and
written to the `$ipc` file when it is turned off, so nothing is left on the
disk and it is gone when SD is restarted. **Only the administrator can profile
another user's session** (what a profile holds is the files and commands of
that user), and an ordinary user sees only their own. It holds 4,000 different
events: when there are more, `display` says how many were not logged. A user's
program is named by the pathname of its compiled code. Only a program's own
statements are counted: a `readv` is one `read`, not that and the reads the
language does for it. The listings, and the four levels of `display`, are under
*Where the work goes: `profiler`* in
[SD TCL - Processes and Phantoms](30-sd-tcl-processes-and-phantoms.html).

```
list.files
```

Reports how many files are open now, the peak for the session and the
configured limit, then lists them:

```
Number of files open = 2.  Peak = 6.  Limit = 80.
```

The limit is the `NUMFILES` configuration parameter. **The peak is what matters
when you are sizing it** — the current figure moves constantly and tells you
almost nothing.

### A line for every file: `file.stat`

```
file.stat {dict} {file ...} {lptr {n}} {no.page}
```

One line for each file: its type, how big it is, how many records it holds, and
a column of `Y` and `N` for the things a file can have.

```
File name                Typ  Cur Mod  Min Mod Grp  Record Cnt     Primary    Overflow  Total Size AK TR ST NC NR EN RP JN
T3                       DYN        1        1   2           5        4096        2048        6144  Y  N  N  Y  N  N  N  N
T1                       DYN        1        1   2           2        4096        2048        6144  N  N  N  Y  N  N  N  N
```

| Column | |
|---|---|
| `Typ` | `DYN` a dynamic hashed file, `DIR` a directory file, `COL` a collection file |
| `Cur Mod` · `Min Mod` · `Grp` | the current and the minimum modulus, and the group size |
| `Record Cnt` | the number of records |
| `Primary` · `Overflow` · `Total Size` | bytes in the primary subfile, in the overflow subfile, and both |
| `AK` `TR` `ST` `NC` `NR` | `Y` if the file has indices, a trigger, statistics collecting (`fstat`), case insensitive record ids, and `no.resize` |
| `EN` `RP` `JN` | encryption, replication and journalling. **Always `N`**: the columns are kept so the report keeps its layout, and none of the three is a property a file has in this system |

**A directory file shows its type and its record count and nothing else.** With
no file names the report covers the files on an active select list (list 0) or,
if there is none, **every file your VOC points at with an `F` entry**; `dict`
asks about the dictionaries instead. The report is about 122 columns wide.

### What one file is: `find.file`

```
find.file {dict} file
```

Opens the file the way any program would and says what it is. It is for the
file that is not behaving as you expect.

```
Dynamic hashed file, group size 2, modulus 1
Pathname: <the full path of the file>
File uses case insensitive record ids
File has indices
```

(The real listing gives the full path; it is left out here because it was a
sandbox's.) The lines after the pathname appear only when they are true: case insensitive record ids, indices, a trigger,
read only. A directory file is reported as one. A file that cannot be opened is
not described; the reason is given instead (*NOSUCH is not defined in your
VOC*, for a name the VOC does not have, and *Invalid file name* when there is no
name at all). **Because it opens the file as a program would, it can reach only
what your VOC and the operating system access switch allow.**

### File activity by session: `fstat.user`

```
fstat.user {top} {user n} {lptr {unit}} {no.page}
```

How many records each session has read, written and deleted since it started.
These are the counts `fstat global` keeps for the whole system, kept for each
session as well. The sessions come in user number order; **`top` puts the busiest
first** (reads + writes + deletes), which shows where a slow system is busy.

```
 User Name.......................Reads..........Writes.........Deletes
    7 sduser                      1143               0               2
```

`user` *n* shows every count of one session:

```
File activity for user 10

Opens                                  2
Reads                                 68
Writes                                 1
Deletes                                4
Clears                                 0
Selects                                0
Index reads                            0
Index writes                           0
Index deletes                          0
Splits                                 0
Merges                                 0

Counted since 21 Oct 00:07
```

**An administrator sees every session; anyone else sees the sessions of their
own login name.** A number that is not a session of yours, or not logged in,
gives *No file activity to show for user 99: not logged in, or not a session of
yours*. The report is shown once, not repeated; run it again for another look.

## Permission bits of records: `set.mode`

```
set.mode {dict} file {all | from n | id ...} {no.query} mode
```

Sets the permission bits of records of a **directory file**, as `chmod` does
for files. *mode* is three octal digits, owner then group then other (`640`).
The records are named on the line, or `all` of them, or those of select list
*n*, or the active select list if none of these is given — which it asks about
first, unless `no.query` is given. The last word on the line is the mode, so a
record cannot be named after it. It answers `3 record(s) set to mode 640`, and
`@system.return.code` is the count.

**It is behind the OS switch.** A change of permission bits is a change to the
operating system's files, so the verb refuses a session that does not have the
OS access `os.execute` has — *SET.MODE changes operating system file permissions
and needs the account's operating system access (set.os.access os on)* — and the
administrator turns the switch on with `set.os.access os on`. The administrator
(`admin` typed) needs no switch.

**It changes the rwx bits and nothing else.** No set-id and no sticky bit (four
digits are refused), only regular files, and a record that is itself a symbolic
link is not followed — it is reported and the file it points at keeps its mode.
A record that is not there is *Record 'name' not found*, and a file that is not a
directory file is *A directory file is required*.

## Reaching a file in another account

```
set.file account file.name pointer.name
```

This writes a **Q-pointer** into your VOC — an indirect pointer naming another
account and a file in it. After it, *pointer.name* is usable wherever a file name
is, and the data stays where it is.

**The account must be in the accounts register**, and the name is folded to
upper case before the register is read. If it is not there you get *Account name
'…' is not in register* and nothing is written.

> **Read what `set.file` prints, not just whether it printed.** Its refusals
> echo the names you passed in, so a script that checks its output for the
> pointer name it asked for will find that name in the refusal as well as in the
> success. Check for the success wording, and treat *not in register* and *not
> found* as disqualifiers.

## Copying, deleting and renaming records

```
copy from {dict} src.file {to {dict} tgt.file} {s1{,t1} {s2{,t2}}...}
copy from {dict} src.file {to {dict} tgt.file} all
```

With no `to` clause the copy is **within the source file**, which is how you
duplicate a record under a new id. Each *s*`,`*t* pair is a source id and the id
to give the copy; a bare *s* keeps the id. Options:

| | |
|---|---|
| `overwriting` | replace records that already exist in the target |
| `updating` | copy **only** records that already exist in the target |
| `deleting` | delete the source record after a successful copy |
| `reporting` | list what was copied |
| `binary` | force binary mode when one side is hashed and the other a directory |

`@system.return.code` comes back as the **number of records copied**, or
negative on error.

```
copyp {dict} filename id... {(options}
```

The Pick-style copy. Its options are single letters in a trailing bracket:
`d` delete source, `i` suppress the id, `n` no pagination, `o` overwrite,
`p` to the printer, `s` suppress field numbers, `t` to the terminal.

```
delete {dict} file.name {record.name} {no.query}
delete {dict} file.name all
```

`@system.return.code` is the **number of records deleted**, or negative on error.

```
cname old.file.name to new.file.name
cname {dict} file.name old.record.name to new.record.name
```

A comma may be used instead of `to`. `rename` is the same verb under the other
name. `@system.return.code` is `0` on success and `-1` if the arguments were
wrong.

**`cname` on a file renames the VOC record too**, which is the point — the
name you type and the name on disk stay together.

### Comparing two files: `compare.files`

```
compare.files {dict} file1 {dict} file2 {detail}
```

Compares the records of two files (a dictionary is named with `dict` in front of
its file, so a file and its dictionary can be compared with each other or with
another) and
counts them in four groups: only in the first, only in the second, in both but
different, and the same.

```
compare.files T1 T2
```

```
Records only in T1: 1
Records only in T2: 1
Records in both that differ: 0
Records that are the same: 1
```

`detail` lists the record ids of the first three groups under their counts:

```
Records only in T1: 1
   COUNT
Records only in T2: 1
   SORT
Records in both that differ: 0
Records that are the same: 1
```

**`@system.return.code` is 0 when the two files hold the same records and 1 when
they do not**, so a paragraph can test it. With one file name the verb answers
*Two file names required*, and a file that will not open stops it (*Cannot open*
the name). The records of a data collection file are compared as their JSON text,
so two equal collections count as the same.

### Taking the carriage returns out: `fixdir`

```
fixdir file {id ...}
```

For a directory file whose records arrived with Windows line ends. **It removes
every carriage return from each record it is given**; a record with none is not
written again. With no record ids every record of the file is done.

```
fixdir BP crtest
```

```
1 record(s) fixed in BP
```

A second run of the same command says `0 record(s) fixed in BP`. A file that is
not a directory file is refused (*T3 is not a directory file*), a file that will
not open stops it (*Cannot open*), and a file name is required. The global
catalogue's directories (`gcat` and `global.bp.out`) and `solo.policy` are
refused as they are for `ed` and `copy`: only a session signed in with the global
password may change them.

The listings in these two sections were produced by running them in a sandbox
build of SD Core for Linux Solo, as the administrator; the refusal of the global
catalogue's directories is read from the source and was not run.

## Seeing what is actually in a record

```
ct {dict} file.name {record.name... | *} {options}
dump {dict} file.name {record.name... | *} {options}
```

Both are the same program. `ct` is a field-by-field listing; `dump` reports the
same data in binary. Options: `hex` for hexadecimal, `binary` for a full-width
report that is not field-based, `no.query` to skip the prompt when a select list
is in use, and `lptr` {*n*} to send it to a print unit.

**Every account has both `ct` and `dump`** — they run the same code, reached
by two separate VOC records rather than one.

## Copying an operating system file

```
oscopy source target {overwrite | append}
```

Copies one **operating system** file, named by its path, which no record verb
can do. Without an option it **will not replace** a target that exists:

```
oscopy /home/pat/in.txt /home/pat/out.txt
oscopy /home/pat/in.txt /home/pat/out.txt
```

```
/home/pat/in.txt copied to /home/pat/out.txt
Target /home/pat/out.txt already exists: give overwrite or append
```

`overwrite` replaces it and `append` adds the source to the end of it. A source
that is not there is reported (*Source ... not found*); any other failure says
*Cannot copy* with the status. **A directory is not copied** — the SD BASIC
subroutine `!ioscopy` does a whole tree — and a program does the same as the
verb with `!oscopy`.

**It runs `cp` as you, so it has your own permissions and nobody else's**, and a
path with a single quote in it is refused. **A path outside the account's own
directory needs the OS switch** (`set.os.access os on`), the same one as
`OS.EXECUTE`; without it you get *name has no OS access, so a file or directory
outside its own account cannot be reached by pathname* and nothing is copied.

**The account's own VOC is closed to it by pathname, inside the account too.** The
VOC is a directory of the account (`voc/%0`, `voc/%1`), and a path that runs through a
directory called `voc` below the account's own directory is a change to it, as a
`write` to a record of the VOC would be. A session without the OS switch that gives such
a path as the target gets the same *has no OS access* message (`!oscopy` and `!ioscopy`
answer `-4`) and nothing is copied; the source is only read, so it is not asked. The
administrator (after `admin`) and an account with the OS switch on are not asked: the
second has a shell and reached those files as it always could. (SD Core asks the same
question for a program file or a dictionary as well; Solo has no edit gates, so the VOC
is the one place it is asked.)

## Triggers

```
set.trigger file.name trigger.name {modes}
set.trigger file.name ""
set.trigger file.name
```

Set, remove, or display a file's trigger function. An empty string in quotes
removes it; no argument at all displays it. **`all`** in place of the modes sets
every one of them: pre- and post-write, pre- and post-delete, read, and pre- and
post-clear. **The function must be catalogued** first — *Trigger function is not
catalogued* otherwise.

```
set.trigger stock zztrig all
```

```
Trigger function: zztrig
Modes: PRE.WRITE, PRE.DELETE, POST.WRITE, POST.DELETE, READ, PRE.CLEAR, POST.CLEAR
```

```
list.triggers {file.name ...} {lptr {n}}
```

`set.trigger` with no option tells you about one file; `list.triggers` does it
for all of them. It looks at the data part and the dictionary of every file your
VOC points at with an `F` entry, or only at the files you name, and lists each
one that has a trigger function with the function's name and the modes it is on
for:

```
File                 Trigger function     Modes
-------------------  -------------------  ------------------------------
t3                   p9                   PRE.WRITE, POST.WRITE

1 file(s) with a trigger function
```

With nothing to report the list is empty and the last line says `0 file(s) with
a trigger function`. The listing was produced by running it in a sandbox build of
SD Core for Linux Solo, as the administrator.

### A trigger on a directory file

A directory file can have a trigger function too (16 Oct 2026). `set.trigger` works on it as on a hashed file, and
`list.triggers` lists it. The directory has no header to hold the trigger, so it is kept in a small text file named
`%header%` in the directory itself, which SD makes when the trigger is set and removes when it is taken away:

```
trigger=zztrig
modes=127
```

`select`, `count` and `analyse.file` do not count it as a record, `clear.file` leaves it (and so the trigger), and no record id
can reach it: `%` is one of the characters SD rewrites in a directory file's record ids, so a program that writes the id
`%header%` makes an ordinary record under another name. The file is not locked away, though: anyone who can write to the
directory can change it with an editor, just as anyone who can write to it can add or remove records, so the trigger is only
as safe as the directory is. A file that is not a header in this form, or names a function whose name no function can have, is
read as no trigger.

**What calls it.** `read`, `write`, `delete` and `clearfile` (and the verbs built on them) call the function for the modes that
are set, in the same order as for a hashed file, with the same arguments, and a pre-write or pre-delete trigger can refuse the
operation with `@trigger.return.code` or change the data being written. **`writeseq`, `writeblk`, `oswrite` and `osdelete` do
not call it**, so a query report sent to a file, or a compile listing, never does; that is openQM's limit too. **A change to the
trigger takes effect when the file is next opened**, not for a program that has it open already (`set.trigger` itself needs the
file to itself, as it does for a hashed file). If the header names a function that is not catalogued, the first operation that
needs it stops with *Trigger function 'name' not loaded for the file*.

## Moving data in and out: `import.csv`, `import.json`, `import.xml` and `convert.data`

```
import.csv pathname file {header} {delimiter x} {quote x} {escape x} {id column | generate} {dict} {overwriting}
import.json pathname collection.file {id name | generate} {overwriting}
import.xml pathname file {id name | generate} {overwriting}
convert.data from.pathname to.pathname {from format} {as format} {header} {no.query}
```

**`import.csv`** reads a CSV file, named by its path, into the records of an SD
file in one line. Each row is a record: the id is the **first column** (or the
column `id` names; `generate` numbers the records 1, 2, 3 and keeps every column
as a field), and the other columns are fields 1, 2, 3 in order. `header` says the
first row names the columns and is not data; with `dict`, a name that has no item
in the dictionary of the file gets a D item pointing at the field of its column.
`delimiter`, `quote` and `escape` name a dialect, as on the
[CSV page](11-sd-basic-csv-files.html), and a quoted field may hold a line break.
Every cell is text: nothing is made a number.

**`import.json`** reads a JSON array of objects, or a JSON Lines file, into a
[collection file](20a-sd-tcl-collection-files.html), one record for each object, kept
as it was. The id is the member `id` names (a string or a number), or `generate`.
An element of the array that is not an object, or an object without that member, is
skipped and counted.

**`import.xml`** is the reverse of the `xml` keyword of the query processor
([the report as XML](21-sd-tcl-query-processor.html)): a document in that shape,
with or without `elements`, is read into the records of a file, the fields
named by the dictionary of the file (a name is the item's id with what XML cannot
have in a name made an underscore). The children of the root are the records; a
single valued field is an attribute (or a child with text), a row of an
association is a child with attributes or children of its own, and a multivalued
field in no association is a child called *name*`_MV`. The id is the attribute
or child `id` names, or the record's number. A value that has no field of that name in the
dictionary is left out and counted (message 11862).

All three leave alone a record that is **already there** and count it as skipped,
unless `overwriting`, and end with *file: n record(s) written, m skipped*. Nothing is
changed by a file that cannot be read (*is not valid JSON/XML ... near character n*) or by a
file that is not there.

**`convert.data`** writes a CSV, JSON, JSON Lines or XML file as another of them. The
format is what each name ends in (`.csv`, `.json`, `.jsonl` or `.ndjson`, `.xml`), or what
`from` (for the file read) and `as` (for the file written) say. A **table** — a CSV file
with `header`, an array of JSON objects, a JSON Lines file, the children of the root of an
XML document — goes to any of the four; to CSV the columns are the names in order of first
appearance, and a value that is not text (a nested array or object) cannot be a CSV cell
and is refused. To XML a table is `<table>` with a `<row>` for each row and a child for
each column. A CSV file without `header` is an array of arrays (columns `c1`, `c2` ... in
XML). A JSON object that is not a table goes to JSON as it is, to XML only when it is an
XML element as `xparse()` gives it, and is refused for CSV and JSON Lines. A file that is
there is asked about (*Overwrite(y/<n>)?*, Enter is no) unless `no.query`.

**All four go through the same gates as `oswrite` and `write`.** The pathnames need the
account's OS-access switch (`os-on`) for anything outside the account's own directory
(*has no OS access ... cannot be reached by pathname*, and nothing is read or written).
Solo has no edit gates (SD Core also asks whether the account may change a program file, VOC or
dictionary), so nothing more is asked.

## Who has these verbs

**Every account has all of them now** — there is no longer a withheld set.
`ct`, `list.files`, `set.file`, `analyse.file`, `analyze.file`, `clear.file`,
`cname`, `compare.files`, `configure.file`, `convert.data`, `copy`, `copyp`,
`create.file`, `delete`, `delete.file`, `dump`, `file.stat`, `find.file`,
`fixdir`, `fstat`, `fstat.user`, `hsm`, `import.csv`, `import.json`,
`import.xml`, `list.triggers`, `oscopy`, `rename`, `set.mode`, `set.trigger`
are all in every account's VOC. (`set.mode` is there but refuses a session
without the OS switch, as above.)

## What is not here

**Indexes are their own subject** and are not on this page. For what an index
is and what it does to a file, see [SD Basic - Alternate Key
Indexes](09-sd-basic-alternate-key-indexes.html).

**Selecting records is the query processor's job**, not these verbs'. `copy`,
`delete` and `ct` all accept a select list built by `select` or `sselect`, and
that is how they are usually driven over more than a handful of ids.

**`reformat` and `sreformat`** rewrite records through a dictionary and are run
by the query processor, so they are documented with it.

**There is no verb that repairs a damaged file.** `analyse.file statistics` will
tell you a file is badly overflowed; `configure.file` is what acts on that.

## See also

[SD TCL - The Command Processor](19-sd-tcl-command-processor.html) ·
[SD Basic - File Handling](07-sd-basic-file-handling.html) ·
[SD Basic - Select Lists](08-sd-basic-select-lists.html).
