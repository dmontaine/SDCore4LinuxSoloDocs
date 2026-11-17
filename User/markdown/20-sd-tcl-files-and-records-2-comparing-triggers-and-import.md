Title: SD TCL - Files and Records: Comparing, Triggers and Importing
Subtitle: Comparing files, copying operating system files, triggers, and moving data in and out.

This page continues [SD TCL - Files and
Records](20-sd-tcl-files-and-records.html).

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
catalog's directories (`gcat` and `global.bp.out`) and `solo.policy` are
refused as they are for `ed` and `copy`: only a session signed in with the global
password may change them.

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
`OS.EXECUTE`; without it you get *name has no OS access, so outside its own account
it can reach by pathname only a directory the administrator opened to it* and nothing is copied.

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
post-clear. **The function must be cataloged** first — *Trigger function is not
cataloged* otherwise.

```
set.trigger stock zztrig all
```

```
Trigger function: zztrig
Modes: PRE.WRITE, PRE.DELETE, POST.WRITE, POST.DELETE, READ, PRE.CLEAR, POST.CLEAR
```

```
list.triggers {file ...} {lptr {n}}
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
a trigger function`.

### A trigger on a directory file

A directory file can have a trigger function too. `set.trigger` works on it as on a hashed file, and
`list.triggers` lists it. The directory has no header to hold the trigger, so it is kept in a small text file named
`%header%` in the directory itself, which SD makes when the trigger is set and removes when it is taken away:

```
trigger=zztrig
modes=127
```

`select`, `count` and `analyze.file` do not count it as a record, `clear.file` leaves it (and so the trigger), and no record id
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
file to itself, as it does for a hashed file). If the header names a function that is not cataloged, the first operation that
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
(*has no OS access ... can reach by pathname only a directory the administrator opened to
it*, and nothing is read or written).
Solo has no edit gates (SD Core also asks whether the account may change a program file, VOC or
dictionary), so nothing more is asked.

## Who has these verbs

**Every account has all of them now** — there is no longer a withheld set.
`ct`, `list.files`, `set.file`, `analyze.file`, `clear.file`,
`cname`, `compare.files`, `configure.file`, `convert.data`, `copy`,
`create.file`, `delete`, `delete.file`, `dump`, `file.stat`, `find.file`,
`fixdir`, `fstat`, `fstat.user`, `hsm`, `import.csv`, `import.json`,
`import.xml`, `list.triggers`, `oscopy`, `set.mode`, `set.trigger`
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

**There is no verb that repairs a damaged file.** `analyze.file statistics` will
tell you a file is badly overflowed; `configure.file` is what acts on that.

## See also

[SD TCL - The Command Processor](19-sd-tcl-command-processor.html) ·
[SD Basic - File Handling](07-sd-basic-file-handling.html) ·
[SD Basic - Select Lists](08-sd-basic-select-lists.html).
