Title: Changes in LS2.0-0
Subtitle: What was added, changed and removed in SD Core Solo for Linux since LS1.1-3 (9 October 2026), by section and in alphabetical order.
Columns: 23 11 66

**This is the list of what changed between LS1.1-3 (9 October 2026) and LS2.0-0 of SD Core Solo for Linux.** It is kept
up to date as the release is built. It was started on 24 October 2026 from SD Core for Linux's list of changes
(`Changes in L2.0-0`, built from the feature spreadsheet and the changelog) and Solo's own changelog (`sdsys/changelog`,
section "Next release"), which has the full story of each change. Solo takes the part of SD Core's work that does not need a
second user; the rows below say what Solo has, and the section at the end says what is Solo's own.

## How to read it

- Every row is one thing that was **added**, **changed** or **removed**. A fix is a change.
- The rows are grouped into sections, and **within each section they are in alphabetical order**.
- A row is one or two lines. The User documentation and the Getting Started pages have the detail; so does the changelog.
- Names are in lower case as SD writes them. `$mode` words, options and BASIC keywords are written the same way.
- Solo has one account and one administrator, whoever types `admin`. Everything that lets something reach beyond the account,
  or leave the machine, is **off as delivered**, and the administrator turns it on.
- A comment at the end of a row names the spreadsheet items the row covers. It is for whoever maintains this document and
  does not show on the page.
- **What Solo does not have**, because it has one account and one user: other accounts and their Linux users, the
  per-account gates beyond its own four (`sh`, `os`, `net`, `ai`), the edit gates, the shared account, the master files
  (`tool.entries`), `run.in`, the administrator's scheduled jobs, the `sdsys` login over ssh, the files route of the API and
  the root audit daemon. SD Core has them because it has more than one user.

## Read this first: changes that alter what an existing system does

| Name | Change | What |
|---|---|---|
| A refused sign-on | Changed | ended with exit status 0, as a session that signed on did; now **1**, so a script, a job or an ssh command can tell refused from done |
| A `select` or `list` with `by` | Changed | records with equal sort keys came out in an order that depended on where they sat in the file; now in record-id order, as `sort` and `sselect` always gave them (not with `saving unique`) |
| A silent remote server | Changed | could be waited for without limit; now it is given up on after `netreply` seconds |
| A sort that uses work files | Changed | gave equal keys in an order that depended on `sortmem` and `sortmrg`; it now gives the order the sort gives in memory (equal keys newest first, the first duplicate kept) |
| AccuTerm and SDTerm | Removed | were supported; use terminal type `xterm` or `vt100` |
| An index after many deletions | Changed | in a large index (three levels) deleting records could take a whole branch out of the index, so lookups missed records that were still there; fixed, and an index damaged earlier is put right by `build.index` |
| An index and a record id | Changed | an index kept an upper-case copy of the id; now it follows the id **as the file holds it**, so rebuild an index that was built on LS1.1-3 |
| Dictionary divide by zero | Changed | stopped the report; now gives zero |
| Fedora | Removed | the installer installed on Fedora; it now refuses it, and installs on Debian and Ubuntu (and their derivatives) only |
| `fileinfo(f, 1021)` | Changed | gave text such as `12 (don), 14 (ann)`; now the user numbers of the other sessions that have the file open, and the text form is key **1100** |
| `intprec` and `sortmem` | Changed | were 13 and 1024; now **11** and **4096** (set the old values in `sd.conf` if you need them) |
| Keys of the full-screen programs | Changed | F5 ran a program in the IDE, SED and the debugger; it now draws the screen again everywhere and **F9 runs**. Ctrl-T is the theme and Ctrl-K the keys on every page, so the keys that had them moved (the home screen's Terminal and Sign out, SED's kill line and toggle, the IDE's go to line and build): see [Tools and the home screen](#tools-and-the-home-screen) |
| Linger | Changed | an unattended install left it off, so SD stopped when you signed out; now every install turns it on unless told `--no-linger` or `enable-linger=no`, and SD with its API and ssh stays running while nobody is signed in |
| Message numbers | Changed | Solo's own messages 11040 to 11054 and 11060 to 11065 moved to 14974 to 14994, so a message means the same in both products; the ssh key request refuses with 14975, 14976 and 14977 |
| Removed features | Removed | words and modes that existed to imitate other products (`option` words, `$mode` words, and the second spellings and forms of statements, clauses, functions, conversions, TCL verbs and query keywords, each listed in [REMOVED](#removed)), and other things: see [REMOVED](#removed) |
| `sed` keys | Changed | were Emacs keys with Ctrl-X sequences; now the keys most editors have (Ctrl-PageDown / Ctrl-PageUp, and Alt-PageDown / Alt-PageUp for a terminal that keeps the Ctrl keys for its own tabs, switch buffers) |
| `sed`, `ide`, `!ced()` | Changed | were 120 by 36 or less; now need a window of at least **140 by 43**, and `ed` is the editor for a smaller one |
| `select file field` | Changed | a field not otherwise used was ignored; now it is `saving multivalued field`, so the list holds the values |
| Shell and OS access | Changed | every Solo account could run Linux commands and reach any file the Linux user could; now both are **off** until the administrator runs `set.os.access`, and an upgrade leaves both off |
| The default terminal | Changed | was 120 columns by 36 lines; now **140 by 43** (a LIST fills 42 lines before it pauses) |
| The group size of a new file | Changed | was 2 KB (1 KB when `sd.conf` did not say); now **4 KB** on a new installation. An upgraded installation keeps its `sd.conf`, and so its 2, and a file keeps the size it has |
| The spelling of what SD prints | Changed | was British (catalogue, colour, analyse, initialised, unrecognised, licence); now **US** (catalog, color, analyze, initialized, unrecognized, license). A program or script that matches SD's messages by their words should be checked. Names are unchanged: verbs, keywords, options and files (`analyse.index`, the `catalogued()` function) |
| The ssh key request | Changed | the API request that lets a managed computer's server install its ssh key was number **49**; it is now **72**, and 49 asks the server for its revision, as in SD Core. A program of your own that sent 49 to install a key must send 72 |
| `when field` with no operator | Changed | kept the null values; now keeps the values that are **not** null |
| Where a session starts | Changed | was the command prompt; a new installation starts at the **home screen** of tiles unless the installer's question is answered with the prompt; **an upgrade from LS1.1-3 keeps the prompt**; `set.start.default` changes it later |

## Sections

**Features of openQM that SD now has:** [ADMIN](#admin) · [API](#api) ·
[BASIC](#basic) · [DATA](#data) · [FILES](#files) · [QUERY](#query) · [SYSTEM](#system) · [TCL](#tcl) · [TOOLS](#tools).

**[REMOVED](#removed)**: what SD no longer has, and what to write instead.

**[New Features Unique to SD Core Solo](#new-features-unique-to-sd-core-solo)**, not part of openQM:
[Administration](#administration) · [AI](#ai) · [API and client library](#api-and-client-library) ·
[Commands](#commands) · [Data formats](#data-formats) · [Encryption](#encryption) · [Files](#files-1) ·
[Installer](#installer) · [Screen library](#screen-library) · [SDNet](#sdnet) · [Security](#security) · [System](#system-1) ·
[Tools and the home screen](#tools-and-the-home-screen).

**[NO CHANGE](#no-change)**: openQM features that needed nothing.

## ADMIN

Administrator commands from the openQM list.

| Name | Change | What |
|---|---|---|
| `config()` name limit | Changed | a parameter name may be up to sixteen characters (was eight), so `config('SCREENTHEME')` finds the setting; a longer name is not found |
| `inhibit.login` | Added | administrator only: stops new sessions while maintenance is done; a killed backup no longer locks everybody out <!--s 387 g46--> |
| `lock.tracer` | Added | administrator only: logs every lock taken and released, with program and line, to the file `locktrace` <!--s g57--> |
| `logout` | Changed | `logout n group` also ends the phantoms n started; `logout` of a session whose process has gone clears the entry and its locks <!--s 386--> |
| `newvoc.mods` | Changed | `deletesdsolo.sh --keep-data` keeps it, and a reload puts it back before the VOC is refreshed <!--s 501--> |
| `SCREENTHEME` | Changed | readable with `config()`; the shipped `sd.conf` says `SCREENTHEME=paper`, and an upgrade adds that line to a kept `sd.conf` that has none (a line that is there is kept); a terminal with no color stays `mono` |

## API

The server side of the API, the client library and the language headers.

| Name | Change | What |
|---|---|---|
| api sign-in: audit | Added | the first entry to an account writes `api login user= account=` to the audit trail |
| `autologout` | Changed | reaches an API session; a client hang found while testing it is fixed |
| client library: call name | Changed | a subroutine called from the client may have a name as long as the lower of 128 and `maxidlen` (it was 63), as on the server's own `call` |
| client library: collection files | Changed | a record read from a data collection file reaches the client as its JSON text, as `ct` shows it |
| client library headers | Added | C# (`sdclient.cs`) and PHP (`sdclient.php`) beside the six already there; neither has been run against a runtime |
| client library: connections | Changed | the number of simultaneous connections is no longer limited |
| client library: detailed error | Changed | the BASIC class `sdclient` returns the detailed status when it cannot connect |
| client library: dictionary | Added | `sdevaluate`, `sdevalconv`, `sdiconv`, `sdoconv`, `sdiconvs`, `sdoconvs` |
| client library: files | Added | `sdopenseq`, `sdreadseq`, `sdreadblk`, `sdwriteseq`, `sdwriteblk`, `sdseek`, `sdweofseq` |
| client library: fixes | Changed | `sdlogto`, `sdchange`, `sddcount` and a second `sdconnect` were checked and found correct, with tests |
| client library: more functions | Added | `sdrevision`, `sdgetvar`, `sdsystem`, `sdindices`, `sdtrans`, `sdrtrans`, `sdtxn` |
| client library: python | Added | a Python wrapper for the library |
| client library: `sdcallx` | Added | `sdcallx` and `sdgetarg`: calls with arguments of any size from the C client library |
| client library: threads | Added | each thread has its own session, transfer buffer and arguments; several threads may use the library |
| client library: tls | Changed | the network session is always encrypted (TLS), so SD meets the QMClient encryption items; Solo's API port is 4249 |
| `master.login` | Changed | the paragraph runs for API connections as for interactive and phantom sessions |

## BASIC

Statements, functions, `$mode` words and compiler directives.

| Name | Change | What |
|---|---|---|
| `~=` | Added | comparison that ignores case <!--s 430--> |
| `account.path()`, `vocpath()` | Added | the pathname of an account and of a file named in the VOC, told only where the account may open it <!--s g107 g193 86--> |
| `arg.present()`, `reusing()` | Added | whether the caller passed an argument, and whether with `reuse()` <!--s g108 g169--> |
| `$basic.options` | Changed | a `mode` line takes a list and a minus turns a mode off; an `include` line names records or files that every program includes <!--s 406 407 290--> |
| `bool()` and friends | Added | `bool()`, `is.null()`, `numeric()`: Boolean, SQL-null and numeric conversions <!--s g109 g144 g161 451--> |
| `@build.time`, `@line` | Added | the time the program was compiled and the source line it stands on <!--s 506 211--> |
| `casing on` and `off` | Added | case sensitivity of string comparisons for the rest of the program <!--s g111 g274--> |
| `ceil()` and `floor()` | Added | `ceil()`, `floor()`, `roundup()`, `rounddown()`: whole numbers above and below, and rounding in steps <!--s g112 g129 g174 g175--> |
| `child()`, `signal()` | Added | whether a user number is a running phantom of this process; raises a signal in another process <!--s g113 g182 301--> |
| `class ... inherits as` | Added | the inherited class's object variable can be named; an empty `()` is accepted after a method name <!--s 374 237--> |
| `class` initial values | Added | a `public` or `private` variable may start with a value; `inherits` passes constants to the inherited class <!--s 356 565--> |
| `class` lookup | Changed | a class with 32 names or more is searched through an index, many times faster <!--s 13--> |
| `class`: `restricted` | Added | a name for the class and the classes that inherit it, and no one else <!--s g168--> |
| `class`: `run` | Added | `run` of a class module or its catalog name makes the object, runs its `main` and lets it go <!--s 484--> |
| `clearcommon name` | Added | sets the variables of the named common block to zero <!--s 52--> |
| `compare()` | Changed | an `i` after `l` or `r` in the justification compares ignoring case; `compares()` does it over two arrays <!--s 307 g114--> |
| `convert()` | Changed | `mcn-`, `mc/n-`, `mcs`, `mcp` with a quoted replacement character, `mt` with `m`, `md` options in any order <!--s 23 183 198 298 488--> |
| `dir()` | Changed | includes the size of each file; lists a plain file as `F` and a directory as `D` only, so a socket is no longer a file and a block device no longer a directory <!--s 366--> |
| `fix()` | Added | a number with a fixed number of decimals, rounded or cut off |
| exceptions | Added | `try`, `catch`, `throw`, `caught()` and `@exception`; many run time errors are caught under openQM's `sys.` names; `dumping` and `saving.stack` <!--s 353 354 355 295 309--> |
| `execute ... capturing` and `os.execute ... capturing` | Changed | output whose final newline stands alone (one empty line, or a newline just past a full 16 KB block) gives the right value; it used to end the session ("double free or corruption") or read freed memory |
| `execute ... silent` and `os.execute ... silent` | Added | the command's output is thrown away; `silent` used to compile as a variable and stop the program at run time |
| `execute ... stacklist` | Added | keeps the default select list and puts it back when the command returns <!--s 61--> |
| `execute.list` | Added | runs commands for each id of list 0 in turn <!--s g38--> |
| `fcontrol()` | Added | the openQM function: next sequential key, valid id, minimum modulus, line terminator modes, forced writes <!--s 452 102--> |
| `field()` and `vslice()` | Changed | take an asterisk for the number of elements and a subvalue slice <!--s 212 132--> |
| `flush.dh.cache` | Added | makes every process close the dynamic files it keeps cached <!--s 83 123--> |
| `for` with a list | Added | a loop over a list of values, and over the elements of a delimited string, with a delimiter <!--s 84 300--> |
| `get var from device` | Added | reads from a FIFO, port, character device or `openseq` file a character at a time, with a timeout <!--s 583--> |
| `getlocks()`, `testlock()` | Added | information about file and record locks; the owner of a task lock <!--s 508 42--> |
| `getpu()` | Added | the `pu$pathname` key (`pu$form.queue` was added and removed again, see REMOVED) <!--s 18 214--> |
| `groupstore` and `revremove` | Added | `groupstore`, `revremove`, `setrem`, `rmvd()`: statements and functions over delimited strings <!--s g136 g170 g172--> |
| `$ifdef`, `$ifndef` | Changed | accept `and` and `or` between token names <!--s 219--> |
| `inmat()` | Changed | after `matches` or `matchfield()` it is the number of the pattern that matched; after `matbuild` and `matwrite` it is the element count; a collection array is accepted <!--s 315 454 591--> |
| `input ... fill` | Added | shows a fill character in every empty cell of the field <!--s 389--> |
| `input ... no.echo` | Added | shows nothing as the user types <!--s 168 363--> |
| `input ... raw` | Added | reads exactly the number of characters given, as they arrive <!--s 548--> |
| `inputif` | Added | reads a line only if one is already there, else takes `else` <!--s 584--> |
| `inputnull`, `inputtrap` | Added | a single character typed alone makes the answer null, or sends the program to a label <!--s 408--> |
| `last()` and more | Added | `last()`, `lasts()`, `nv()`, `ns()`: the final element of a delimited string; value and subvalue position numbers (`left()` and `right()` were added and removed again, see REMOVED) |
| `locate ... by` | Added | `c` forces a case-sensitive search and `n` a case-insensitive one <!--s 419--> |
| `matchess` and `trimws` | Added | `matchess`, `matchfields`, `trimws`, `trimw()`: `matches`, `matchfield` and `trimw` over a dynamic array; `trimw` trims white space <!--s 302 g152 g153 g189--> |
| matrix arguments | Changed | a matrix passed as an argument needs no `dim` in the receiving subroutine; `if mat a = mat b` compares two matrices; the compiler no longer reports an undimensioned argument <!--s 44 111 519--> |
| `matstr()` | Added | converts a two dimensional matrix to a dynamic array <!--s 509--> |
| `matwritecsv` | Added | one CSV line of every element of a matrix <!--s g154--> |
| `digest()` | Added | the md5, sha1, sha256, sha384 or sha512 digest of a string or a file (`md5()` was added and removed again, see REMOVED) |
| `mergelist()` | Added | merges two lists by a mode; a mode for lists not in order; fixed for an empty list <!--s 420 539 566 g155--> |
| `$mode debug.equates` | Added | keeps `equate` and `$define` tokens in the object so the debugger can show, set and watch them <!--s g218 246 239--> |
| `$mode explicit` | Added | a variable must be named in a `dim` statement before it is used <!--s g220 234--> |
| `$mode` lists | Changed | `$mode` takes several names separated by commas, each with an optional minus <!--s 105--> |
| `$mode non.durable.txn` | Added | a committed nested transaction writes nothing until the outer one commits <!--s g221 442--> |
| `$mode partial.expressions` | Added | `a and b` and `a or b` stop evaluating as soon as the value is known <!--s 417--> |
| `$mode strict.equate` | Added | warns when a name given to `equate` or `$define` is also a statement or function name <!--s 582--> |
| `$mode trap.unused` | Changed | warns only of a variable set by a plain `x = ...` and never read; a variable a statement fills (`readu`, `locate ... setting`) is not reported |
| `$mode trap.unused.main` | Added | `trap.unused` for the program's own variables <!--s g227--> |
| `no.query` on cataloging | Added | the `basic` command, the `$catalogue` directive and the `catalogue` line of `$basic.options` take `no.query`, which stops the compiler asking before it removes an older entry |
| `oconv()` date `i` | Changed | the `i` code of a `d` conversion (the ISO week number) gives the week; without `yi` beside it, it gave whatever number was left in memory |
| `open ... non.transactional` | Added | a write through that file variable is not undone by a rollback <!--s 64--> |
| `open.socket()` | Changed | takes a connect timeout <!--s 174--> |
| `openseq` | Added | `creating` makes the file at once; `shared` lets several processes append; a block device can be opened <!--s 485 397 112--> |
| `openseq creating` | Changed | makes a name that is free exclusively: a name taken in the meantime, or a link that leads nowhere, is refused with `status()` 6032 and nothing is made (`shared` refuses a link that leads nowhere the same way, and otherwise still opens or creates); the `openseq file, record` form does the same, and an `openseq` of such a link without `creating` gives a file that cannot be written; the file is made only once the lock on its name is held, so a name another session holds leaves no empty file behind |
| `opentemp` | Added | makes a temporary dynamic file that is deleted when its variable is closed; in the temporary directory when no path is given <!--s 431 440--> |
| operating system files | Added | `oscopy`, `osread`, `oswrite`, `osdelete`, `!oscopy`, `!ioscopy`: copy, read, write and delete operating system files by name, held to the account's OS gate <!--s 369 517 g163 g164 g165--> |
| `option()` | Added | reads and sets the state of an option <!--s 224 473--> |
| `ospath()` | Added | keys `os$fullpathx`, `os$info`, `os$parent`, `os$uid`, `os$gid`, `os$modes`, `os$is.file` and `os$is.dir` <!--s 293 432 233--> |
| `parse()` and `ttyget()` | Added | `parse()`, `sort.compare()`, `ttyget()`, `ttyset`: split by a pattern; compare by a sort rule; the terminal modes as an array <!--s g166 g183 g187 g190 g191--> |
| `phantom` statement | Added | starts a phantom from a program; the log name may be a variable <!--s 390 351--> |
| `precision inherit` | Added | takes the precision of the calling program <!--s 201--> |
| `@pstat` | Added | text a program leaves for `pstat` to show <!--s 492--> |
| `quotes()` family | Added | `quotes()`, `squotes()`: enclose every element of a dynamic array in quotes <!--s 11--> |
| `read` etc. as a condition | Added | `read`, `readnext`, `open` and the like may be the condition of `while` or `until` <!--s 206--> |
| `recordlocku` | Changed | fixed: it did not always trap an illegal record id <!--s 528--> |
| `redim` | Added | resizes a matrix passed to a subroutine as an argument <!--s 215--> |
| `regex()` | Added | regular expression matching <!--s 510--> |
| `seek` | Changed | fixed: a negative offset from the end of a sequential file was refused <!--s 259--> |
| select list variables | Added | `formlistv`, `getlistv`, `writelist`, `movelist`, `movelistv`: select list variables and numbered lists <!--s 291 g134 g135 g195 g156--> |
| `select.socket()` | Added | waits on the sockets held in a matrix <!--s 161 g176--> |
| `selectindexv` family | Added | `selectindexv`, `selectleftv`, `selectrightv`: index scans into a select list variable <!--s g177--> |
| `send` | Added | `writeseq`, or with a colon `writeblk`, to a file opened with `openseq` <!--s g178--> |
| `sendmail` | Changed | fails (`er$ioe`, the mail command's status in `os.error`) when the mail command cannot run or fails; it reported success |
| `set.socket.mode` trace | Added | `skt$info.trace` logs the data sent and received on a socket to an owner-only file <!--s 162--> |
| `socket.info()` | Changed | fixed: `skt$info.open` on a number, a string or an unassigned variable stopped with a fatal error <!--s 249--> |
| `!sort` | Changed | sorts a select list variable; the internal sort is open to programs <!--s 274 371--> |
| `sselectv`, `sselectn` | Added | sorted selects into a select list variable; `right.aligned` and `descending` <!--s 209 243 367--> |
| `status()` after a transaction | Added | 0 if it committed, 1021 if it rolled back <!--s 19--> |
| string arithmetic | Added | `sadd`, `ssub`, `smul`, `sdiv`, `scmp`: exact string arithmetic on numbers of any length and any decimals <!--s 391 468--> |
| substring assignment | Added | `s[3] = "xy"` overwrites the last three characters <!--s 41 325--> |
| `trans()` family | Changed | `trans()`, `rtrans()`: accept `a`, `s` and `c` items and an `l` link; a file that will not open takes the error action of the call <!--s 43 182 7--> |
| variable names | Changed | a name with two or more underscores together (`a__b`) is valid <!--s 336--> |
| `watch` | Changed | the debugger's `watch variable operator value` stops only when the condition is met <!--s 45--> |

## DATA

Data collections, JSON, XML, CSV and the verbs that import them.

| Name | Change | What |
|---|---|---|
| `ced` | Added | full-screen editor for a data collection, or for JSON text in an ordinary file |
| `!ced()` | Added | full-screen editor for a collection held in a variable, with the keys every page shares (Ctrl-T, Ctrl-K, F5); the editor is also reached from `ide` and `sed` F12 and by the `ced` command |
| collection arrays | Added | `mat()`, `c{list, n}`, `ins ... as`, `del`, `inmat()` on an array |
| collection file verbs | Added | `ed`, `copy`, `compare.files`: work on a collection file; the post-write trigger no longer crashes the process |
| collection files | Added | `create.file name collection`; records are collections stored as JSON; `read` and `write` them whole; `ct` shows JSON <!--s 304--> |
| `collection()` and `c{name}` | Added | a collection is a set of named values read and written by name; `enumerate()`, `element.exists()` <!--s 311 518 529 339--> |
| `csv.mode` | Added | how `printcsv`, `writecsv` and `matwritecsv` quote, with a delimiter, quote and escape character <!--s 173 200--> |
| `csvdq()`, `dparse.csv` | Changed | `csvdq()` takes a quote and an escape character as a third and a fourth argument; fixed: a return code was corrupted when used with `execute ... returning` <!--s 526--> |
| `element` keyword in queries | Added | names an element of a collection file by its path, in a display list, `with`, `by`, `sum` and the like |
| `expand()` and links | Added | a link `"file:id"` is followed; `trans()` of an `e` item |
| file variable in a collection | Changed | a collection holding a file variable is written to a collection file as a null |
| `hu` conversion | Added | `oconv` and `iconv` escape and unescape |
| `indices` on `e` items | Added | a data collection element can be indexed; a query with `with` on it uses the index |
| `jboolean()` and `jtype()` | Added | `jboolean()`, `jnull()`, `jtype()`: true, false and null in collections, and the type of a value; a Boolean type <!--s 312--> |
| `jbuild()` | Changed | the pretty-print mode names an indent of up to 16 spaces or tabs; marks in a string are written as hex Unicode <!--s 453--> |
| `jparse()`, `jbuild()` | Added | JSON text into a collection and back |
| `mcdx` conversion | Changed | fixed: gives hexadecimal both ways, as `mx` does (`oconv` returned the number unchanged); kept as a second name of `mx` for programs moved from openQM |
| operators on an element | Added | `+=`, `-=`, `*=`, `/=`, `:=` change an element in place |
| the debugger and collections | Added | `/c`, `/c{path}` and `dump c{sub} pathname` show and write a collection; diving in with `/*` no longer crashes <!--s 530--> |
| `xml` keyword for queries | Added | `list ... xml` writes the report as XML, with an optional DTD or schema <!--s g211 g202--> |

## FILES

File handling: opening, indexes, triggers, print files, sorting, sequential files.

| Name | Change | What |
|---|---|---|
| `account:file` file names | Changed | always available, with no `filerule` setting: a name in no VOC entry with one colon is read as a Q-pointer to that account's file and meets every Q-pointer check; in Solo it reaches only the one account's own files |
| `analyse.file` | Changed | `statistics` shows a histogram of group utilization <!--s 360--> |
| `analyse.index` | Added | every index with its number of values and spread <!--s g9--> |
| `catalog -name` | Added | a hyphen is a global catalog prefix; `delete.catalog -name` removes it <!--s 217--> |
| `clear.file` | Changed | fixed: a `no.resize` file keeps its modulus <!--s 590--> |
| `configure.file ... reporting` | Added | shows how many records have been converted <!--s 172--> |
| `create.file` | Added | `comment`, `multifile` and a relative `pathname` kept relative in the VOC <!--s 383 522 278--> |
| `create.index` | Added | `no.case`: keys that differ only in case are one key; `deleting` replaces an index; a default location in `$index.path` |
| `dir_dict` | Added | three items for a directory file's dictionary: `days`, `dtm` and `owner` <!--s 181--> |
| directory file line ends | Added | line terminator modes (`l`, `c`, `b`, `x`) and triggers <!--s 177--> |
| `directory files` ids | Changed | the record ids may hold bytes 128 to 247 <!--s 34--> |
| `disable.index` | Added | stops an index being kept up to date for a large load; `build.index` switches it on <!--s g28--> |
| exclusive access | Changed | a command that needs a file to itself names the user holding it when it cannot get it <!--s 107--> |
| `file.stat` | Added | one line per file: type, modulus, records, bytes and flags; limited to named files; with dictionaries <!--s 109 152 g40--> |
| `find.file`, `find.program` | Added | what a file is; the catalog entries of a program <!--s g42 g43--> |
| `fixdir` | Added | takes the carriage returns out of directory file items that came from a Windows system <!--s 458--> |
| hashed files | Changed | a split that finds the disk nearly full stops before the file's size is changed; `sdfix -z` clears dead space <!--s 329 33--> |
| `hold.file.logger` | Added | a cataloged subroutine called when a hold file print unit opens and closes <!--s 56--> |
| `l-type` dictionary links | Added | usable in an `i`-type; links may follow links, up to ten, and reach `a`, `s` and `e` items <!--s 7 204 544--> |
| `list.common detail` | Added | shows where each common block was created <!--s 362--> |
| `list.index` | Changed | the report may go to a printer <!--s 73--> |
| `list.triggers` | Added | every trigger subroutine used by the files of an account, multifiles, dictionaries and directory files included <!--s 74 187 321--> |
| `listf`, `listfl`, `listfr` | Changed | list a multifile line by line, and take `with` and `without trigger`, `encryption` and `replication` <!--s 3 185--> |
| print unit mode 3 | Changed | takes an update lock on the `$hold` item while it is written <!--s 53--> |
| `rebuild.all.indices` | Added | rebuilds the indices of every file in the account <!--s 58--> |
| `select` of a directory file | Changed | fixed: at the open-file limit it closes the oldest cached file and goes on <!--s 279--> |
| `set.file` | Changed | the target may be a `q`-pointer; a `q`-pointer with no file name points at the VOC <!--s 32 435--> |
| `set.mode` | Added | the permission bits of records of a directory file <!--s g81--> |
| `set.trigger` | Changed | fixed: refused every function name; takes `all`; no longer needs exclusive access <!--s 388 157--> |
| sort of long records | Changed | the disk sort handles records and keys over 32 KB <!--s 455--> |
| trigger on a read | Changed | fixed: a locally cataloged read trigger could end the process <!--s 317--> |
| `vocname` letters | Added | `r` opens a file read-only, `n` non-transactionally, `s` with forced writes, from field 6 of its VOC entry <!--s 197 380 102--> |

## QUERY

The query processor and the dictionary.

| Name | Change | What |
|---|---|---|
| `absent.ignore` | Added | do not print the `not found` list for ids that are not in the file <!--s 25--> |
| `break.on "'n'"` | Added | starts a new page and numbers it 1 again <!--s 16--> |
| `by.exp` | Changed | more than one `by.exp` in the same association is sorted <!--s 572--> |
| `csv ... as file id` | Added | writes the report as a record of a directory file; a multivalued field is split over lines <!--s 159--> |
| `csv 3` | Added | every item quoted, numbers too <!--s 26--> |
| dictionary expressions | Changed | divide by zero is zero (an `if` with a part missing, a format after a value and hyphens in names were added and removed again, see REMOVED) <!--s 36 48 270 271 423 547 564--> |
| `for`, `from none` | Added | `search file for "text"`; `from none` ignores an active list 0 <!--s 190 378--> |
| `in`, `not.in` | Added | test a value against a saved list or a file record <!--s 94--> |
| `median`, `mode`, `range` | Added | figures at the end of a report and each breakpoint <!--s 231 g209--> |
| `$mode` in an `i`-type | Added | an `i`-type may begin with `$mode` for a few of the compiler modes <!--s 489--> |
| `no.page` | Changed | accepted on any query command <!--s 379--> |
| options | Added | (`query.sort.no.case`, `clear.select`, `query.string.comp`, `query.str.comp.all` and `default.mv` were added and removed again, see REMOVED) <!--s 140 299 412 205 403--> |
| `$query.defaults` | Changed | field 3 holds default words for `select`, `sselect` and `search`; field 4 the format of the id column <!--s 122 405--> |
| `reformat ... @record` | Added | appends the whole source record <!--s 546--> |
| `reporting` | Added | `n record(s) examined` for every thousand records <!--s 416--> |
| `rpt` | Added | repeats a single valued field on each line of a report on a multivalued one <!--s g210--> |
| `scan`, `search ... display` | Added | show the lines of a record that hold the strings; `search all` searches the id too; strings may come from a file <!--s 437 447 221 141--> |
| `select file field` | Changed | a field no clause uses is `saving multivalued field` <!--s 39--> |
| `show` | Changed | `b` goes to the bottom; `sort.sup`; a missing record is listed and the command ends <!--s 477 495 524--> |
| `sorted.sample n` | Added | counts the records after the sort <!--s 438--> |
| `vertically` | Changed | `fmt` for the name and data format; `$vlist` sets them for the file; multivalued data and a wrapped line are told apart <!--s 160 579 580--> |
| `when field` | Changed | with no operator keeps the values that are not null <!--s 543 343--> |
| `x` breakpoint control | Added | captures the previous value of a break field in `@bpv` <!--s 232--> |

## SYSTEM

The kernel, `sd.conf`, and the numbered keys.

| Name | Change | What |
|---|---|---|
| `cleanup=n` | Added | seconds between the checks that clear sessions that ended without logging out <!--s 98 g283--> |
| `config` | Changed | lists the new settings; `config()` reads them |
| `deadlock=2` | Added | saves the lock tables when a deadlock is found <!--s 500--> |
| `fileinfo()` | Added | the keys 1022 to 1040 as the openQM manual numbers them; `fl$seqtype`, `fl$locks`; 1021 now answers the user numbers <!--s 62 113 498 337--> |
| `intprec` | Changed | default 11 <!--s 6--> |
| lock table | Changed | the time each lock was taken and each wait began is noted; more than 65,535 locks in a file work; a program that filled the table can be logged out <!--s 487 344 532--> |
| `numlocks` | Changed | range checked, 1 to 32767 |
| `phantoms=u,n` | Added | keeps user numbers for phantoms started with `user` <!--s 31 g296--> |
| process dump | Changed | shows the length of each string, the variables of objects and the environment <!--s 22 125 569 439--> |
| `pterm` keys | Added | `pterm erase`, `ctrl.filter`, `mark`: which key erases; control characters filtered; control keys that type the marks <!--s 155 253 254--> |
| `-quiet` | Changed | `sd -quiet` suppresses the VOC update question <!--s 533--> |
| `sd.conf` parameters | Added | `dhcache`, `lgnwait`, `maxrlock`, `pwdelay`, `userpool`: `sd.conf` parameters, each optional with the old fixed number as the default <!--s 90 46 313 106 551--> (Solo has one user: it accepts `lgnwait`, `pwdelay` and `userpool` and they do nothing) |
| sessions on a dropped connection | Changed | a session whose terminal or ssh connection goes away while it is writing the error log or the audit trail no longer waits for itself for ever, which stopped every later login |
| `sortmem` | Changed | default 4096 <!--s 535-->|
| `status` | Changed | file times are in the user's time zone; field 27 is the file pathname <!--s 96 585--> |
| `-status` | Added | `sd -status` shows the version, start time and process counts <!--s 511--> |
| `system()` keys | Added | 14, 19, 20, 21, 33, 35, 45, 1016, 1036, 1037, 1040, 1045, 1047, 1048, 1049, 1051, 1062, 1065, 1067, 1071, 1073, 1083, 1086, 1088 <!--s 65 66 114 192 225 244 206 294 307 308 326 421 433 586 587 g252 g253 g254 g255 g256 g260 g261 g265 g266 g267 g270 g271 g274 g276--> |
| terminfo | Changed | `eblink`, `eprot`, `edim`, `erev`, `ebold` capabilities; `xmc#1` in the two ADDS 4000 entries <!--s 47 545--> |
| the call name | Changed | can be as long as 128 characters (the lower of 128 and `maxidlen`) <!--s 241-->|
| the startup process | Changed | its `@tty` is `startup` <!--s 91--> |
| time zone and `epoch()` | Added | `epoch()`, `mvdate()`, `mvtime()`, `mvdate.time()`, `mvepoch()`; `set.timezone` and `config timezone` <!--s 69 124 g126 g157 g158 g159 g160 g181 g310--> |
| `usplit=1` | Added | interactive users take the lowest user number, phantoms and API sessions the highest |

## TCL

Commands and VOC verbs that every account has, and the command language.

| Name | Change | What |
|---|---|---|
| `autologout` | Changed | works in the full-screen programs (the screen library, `ide`, the home screen, the tool pages): idle time is counted across their short timed waits, and only a typed key (or the break key) starts the period again; a program that polls with very short timed `keyin`s and works between them counts as waiting |
| `block.print`, `block.term` | Added | print text, or show it on the terminal, in letters seven rows high <!--s g11 g12--> |
| `catalog ... local relative` | Added | stores the object file's path relative to the account <!--s 131--> |
| `catalog` overwrite | Changed | pointing a local catalog entry at a different object file asks first <!--s 490--> |
| `catalog` prefix | Added | a prefix separate from the call name; `$private.catalog` or `$private.catalogue` <!--s 210 93--> |
| catalog name as a command | Changed | a program run by its catalog name takes `lptr`, `no.page` and the like |
| command stack | Changed | `.c-` undoes a `.0`; the stack is kept in the account the user entered <!--s 296 396--> |
| `command.logging` | Added | a log of every command this session runs, in the account directory, mode 0600 <!--s 444 g16--> |
| `como` | Added | `append`, `in file`; `$como` ids pass the same name transform <!--s 8 361 9--> |
| `compare.files` | Added | counts the records only in one, only in the other, in both but different <!--s 534--> |
| `ct` | Changed | `id.sup`, `num.sup`; lower case file names; the object code of a compiled dictionary item is not shown <!--s 242 347 373--> |
| `decatalog` | Added | deletes a local catalog entry and its object code <!--s 576--> |
| `delete` | Changed | a record another user holds locked is reported and skipped <!--s 50--> |
| `delete.common all except` | Added | deletes every named common block but those listed <!--s 425--> |
| `ed` | Changed | `l` with no text as the first command works; marks typed in input; `ev` on a compound `i`-type; line numbers show positions in edit values mode; `spool` closes the printer; external commands; automatic ids <!--s 264 268 320 331 375 446 523 305--> |
| `fstat` | Changed | wider columns <!--s 55--> |
| `fstat.user` | Added | file activity of each session <!--s g44--> |
| `git` option | Added | a directory file keeps `.gitignore` and `.gitattributes` under their own names <!--s g238--> |
| `hot spot monitor` | Changed | counts opens, reads, writes and deletes of each program <!--s 491--> |
| `if` in a paragraph | Added | `exists`, `not.exists`, `in` and `not.in`, with `no.case` <!--s 28 230 426 476--> |
| inline prompts | Changed | largely case insensitive; may use quoted prompt strings <!--s 171 377--> |
| `k` entry in the VOC | Changed | fixed: a `k` entry over a locally cataloged program stopped with "Unable to load" <!--s 263--> |
| list merges | Changed | `merge.list` finds a key without scanning, so large lists merge in seconds <!--s 414--> |
| `list.item` | Changed | numbers lines beyond 999 with as many digits as needed <!--s 349--> |
| `list.readu`, `pstat`, `getlocks()`, `system(1083)` | Changed | another session's text (a lock's record id, file path or user name, a command, a program name, `@pstat`) shows a control character, `del`, bytes 128-159 and a mark character as `?` and cannot send a code to your terminal; the locks and `fileinfo(f, 1039)` are unchanged |
| `listu`, `who`, `pstat` | Changed | `!listu` too: show the account and the login name; `listu detail`; the blocking user and when a wait began <!--s 12 24 29 38 77 121 289 427 577--> |
| `map local`, `map no.page` | Added | list the locally cataloged programs; heading once <!--s 153 287--> |
| `move.list` | Added | the command form of `movelist` <!--s g62--> |
| `multisort` | Added | `!multisort` sorts several related lists by one of them <!--s 568--> |
| `option` words | Added | `dir.dtm`, `git` and `spool.command` (twenty more were added and removed again, see REMOVED; SD Core's `inherit.ownership` is not in Solo, which has one operating-system user, by the owner's word of 9 November 2026) <!--s 288 79 314 428 461 445 37 57 154 165 404 429 49 g230 g231 g232 g233 g237 g239 g240 g241 g242 g245 g247 g248--> |
| `oscopy` | Added | copy an operating system file; `overwrite`, `append` <!--s 369 517--> |
| `phantom` | Added | `no.msg`, `data`, `log`, `group` and `user` options; `!phlog` gives a phantom's log record; a phantom started while SD is suspended gives back its place <!--s 87 137 364 483 31 385 280--> |
| phantom and terminal lists | Added | `list.phantoms`, `list.term.types`: the phantoms and no one else; the terminal types <!--s 381 g49--> |
| `private vocabulary` | Added | `$private.voc`: a file of commands searched after all other command sources <!--s 1--> |
| `profiler` | Added | records what the programs of a session do and where <!--s g66 g203-->|
| `redo` | Added | runs a command again and again, `times` and `every` <!--s 188--> |
| saved list verbs | Changed | `save.list`, `get.list`, `delete.list`: `keep`, `deleting`, `count.sup`, a default name, a name left out, `in file` <!--s 2 384 387 401 494--> |
| `selectindex` | Added | the command form of the statement <!--s 119--> |
| `setptr` | Changed | `form.feed`, `inform`, `eject`, `as ... in file`; the digits of `next` <!--s 51 59 70 139 448--> |
| `sh` with no command | Changed | fixed: starts the interactive shell again (it said *Error 2 executing operating system command*); it still needs the `sh` switch |
| `show.list` | Added | a page at a time, starring and clearing items <!--s 449-->|
| `sort.list`, `select.dir` | Added | sort a saved list; select the files of a directory <!--s 158 465--> |
| `sp.view` | Changed | `by field`; `x` deletes the item shown; shows where output goes for every print unit mode <!--s 450 560--> |
| `term` | Changed | takes two numbers for the default printer; `term default` goes to 140 by 43 <!--s 466--> |

## TOOLS

The debugger and the editors, from the openQM list. The full-screen tools that are SD's own are under New Features Unique to SD Core.

| Name | Change | What |
|---|---|---|
| `debugger` | Changed | full-screen display on any terminal that can address a cell and clear itself; `/*` for every variable; `^`; `window n`; `set v = token`; a stop before each `case`; a blank line repeats the last command; `brk` lists the breakpoints <!--s 21 67 97 142 193 203 327 335 513 550--> |
| `debugger` phantom | Changed | an `@variable` is read from the process being debugged <!--s 251-->|
| `sed` | Changed | is the IDE's editor, started alone: no files pane, the same keys and menus; asks File name? and Record name? when not given, `?` lists the records; takes `creating.seqkey` in place of a record id, like `ed` <!--s 446-->|
| `sed` line numbers | Changed | are on when a record is opened (they were off until `lnum`); Tools > Line numbers toggles them (checked while on), and `no.lnum` in `&sed.options&` starts SED with them off |
| `sed` option `no.case.explore` | Added | in `&sed.options&`: the list of records is in order whatever the case of the ids, not capitals first |
| `sed` option `scroll1` | Added | in `&sed.options&`: page up and page down move one line less than a page, so one line of the old page stays |
| `sed` screen | Changed | looks like the toolkit pages: a key row at the bottom (the keys of the layout in use), the editor's name and the person at the right of the menu bar, and the site's screen theme's colors; no key or command changed |
| `sed` syntax color | Changed | is on when SED starts, on a terminal that can show color (it was off until asked for); `no.syntax` in `&sed.options&` or `syntax off` turns it off, and the bare `syntax` switches it |
| `sed` view | Added | Search > View lines, or `view` and a text: a new read-only buffer holding only the lines of this one that contain the text |


## REMOVED

These were in LS1.1-3 - or were added and taken out again during LS2.0-0 - and **SD no longer has them**. This is the one place where a removal is listed,
whatever the area it belonged to. Most were removed because they existed only to make SD behave like another MultiValue
product, or gave a second way to do something SD already does one way. **A program, a login paragraph or a `$basic.options` record that uses one has to
be edited**; the last column says what to write instead. A program that was compiled with a removed mode keeps what it was
compiled with until it is compiled again. More rows are added as the removal goes on.

| Name | Change | What |
|---|---|---|
| AccuTerm terminal types | Removed | the eleven `-at` terminal types and AccuTerm's escape sequences |
| `$mode` words that imitated other products | Removed | `prclose.default.0` (write `printer close on` *unit*), `unassigned.common` (a common variable starts as zero), `no.case.invert` (give `input` the `nocaseinvert` clause) and `optional.then.else` (a `read` takes a `then` or an `else`) are refused now with *Unrecognized mode name*; `conditional.statements`, `implied.stop` (end a program with `stop`) and `exception.save.stack` (write `saving.stack` on the `catch`) were added and removed within 2.0-0, so they never shipped |
| `option` words that imitated other products | Removed | `option pick` and 22 words, refused now with *Option name not recognized*: `assoc.unassoc.mv`, `pick.breakpoint`, `pick.breakpoint.u`, `pick.explode`, `pick.grand.total`, `pick.implied.eq`, `pick.null`, `pick.wildcard`, `qualified.display`, `with.implies.or`, `query.no.case`, `query.priority.and`, `ampm.upcase`, `crdb.upcase`, `no.date.wrapping`, `space.mct`, `lock.beep`, `ed.no.query.fd`, `create.file.upcase`, `select.keep.case`, `debug.rebind.keys`, `proc.a`. SD does what it did with them off: write `with city = "Reno"`, use `like` for wildcards, brackets and `or` for the order of tests, `no.case` after an operator or `option query.sort.no.case` for case; times and amounts are `am`, `pm`, `cr`, `db`; a date such as 31 February rolls into the next month; a lock wait is silent; `fd` in `ed` asks first; `create.file` names a file in lower case |
| screen save and restore | Removed | `save.screen()` and `restore.screen`: still exist and do nothing the terminal could see |
| `sed` extension language | Removed | compile and run of `proc` / `func` extensions, `&sed.extensions&`, and Tools, Run extension: a program in SD BASIC does the work of one |
| `sed` key rebinding, Emacs layout and typed commands | Removed | `&sed.bindings&`, `load.keys`, `save.keys`, the repeat count, the quote character and the words typed at Alt-X: use the menus and the keys of the IDE's editor; Alt-X runs nothing now, Build, Run a command does |
| `sed` split window | Removed | Split, Unsplit and Toggle window, and `split.up` / `split.down`: to work on two records, use two buffers of `sed` or two tabs of `ide` |
| `sdterm` terminal type | Removed | with SDTerm and SDConsole, and the `term display` line about them |
| `term 120,36` at login | Removed | `login` no longer sends it |
| second names for statements | Removed | `$insert` (write `$include`), `include` without the `$` (write `$include`), `com` (write `common`), `dimension` and `redimension` (write `dim` and `redim`), `enter` with `$mode pick.enter` (write `call`, or `chain` to hand control on for good), `perform` (write `execute`), `printerr` (write `inputerr`), `remark` (write `rem`), `rqm` (write `sleep`), `sub` (write `subroutine`, also after `local`, `private` and `public`) and `tclread` (read `@sentence`): the compiler refuses each |
| second forms of statement clauses | Removed | `execute ... setting` (write `returning`), `execute ... passlist` with no variable (it did nothing: name the variable, or leave the clause out), `input ... for` and `input ... timeout` (write `waiting`), `opentemp ... ecs` (a word that was read and ignored: it is an ordinary name now, so the compiler warns that it is never assigned a value) and the bracketed Pick form `locate(expr, array, f, v; position; order)` (write `locate expr in array<f, v> setting position`) |
| second names for functions and variables | Removed | `ereplace()` (write `change()`, which takes -1 for every occurrence), `sentence()` (read `@sentence`), `swapmarks()` (it returned its argument), `terminfo.ex()` (write `terminfo()`) and `@system.set` (write `@system.return.code`) |
| `mcd`, `mcx` and `mcxd` conversions | Removed | second spellings of `mx`: an invalid conversion code now, STATUS 2 (`mcdx` stays, as `mx`) |
| second spellings of query keywords | Removed | `col.spcs` (write `col.spaces`), `display.name` (`col.hdg`), `evaluate` (`eval`), `calculate` (`calc`), `intersection` and `difference` (`intersect` and `diff`, also in `merge.list`), `margin` (`left.margin`), `stats` (`statistics`), `supp` (`hdr.sup`) and `only` (`id.only`): **a new account does not have their VOC records**, and the next `update.accounts` removes them from existing accounts, through the deletion list (saved copy first; a record of your own with the same name is kept; `restore.voc` puts one back) |
| `only` and `id.only` before the file name | Removed | `list only customers` is refused: write `list customers id.only` |
| `=>`, `=<` and `><` | Removed | the mirrored spellings of `>=`, `<=` and `<>`; as for the keyword synonyms, a new account has no record for them and an old account keeps its own |
| eleven `option` words | Removed | `run.no.page` and `paginate.on.heading` (a run is paginated unless `run ... no.page` says not), `pick.ml.conv.mask` (a left bracket in `ml` / `mr` is the round-bracket negative style), `chained.select` (a select does not run the data queue), `default.mv` (a blank S/M flag is single valued), `query.string.comp` and `query.str.comp.all` (`=` and `ne` compare as BASIC does), `assoc.all` (the dictionary's associations are used), `query.footer.pause` (no pause after a page footer), `query.merge.print` (a report closes printer 0) and `keep.old.object` (a failed compile leaves no object code). Each behaves as it did with the option off; their numbers stay reserved |
| `dquote()`, `dquotes()`, `swap()`, `xlate()` | Removed | second names of `quote()`, `quotes()`, `change()` and `trans()`: write those |
| `caption`, `nopage`, `col.hdr.space` | Removed | `caption` and `nopage` were second names of `grand.total` and `no.page`; `col.hdr.space` (a blank line under the column headings) was added and taken out again: end a heading with a blank line |
| `listdict`, `edit.list` | Removed | write `list dict` *file*, and `ed $savedlists` *name* |
| loose forms of a dictionary expression | Removed | an `if` with no `then` or no `else` is refused again (messages 2950 and 2951), a hyphenated item name such as `cust-no` is read as a subtraction, and a format string straight after a value needs its operator |
| `$mode execute.clearlist`, `open.fail.zero.fvar`, `pick.readnext` | Removed | added and taken out again in this release: the default select list is not cleared before an `execute` (use `clearselect 0`), the file variable of a failed `open` stays unassigned, and a `readnext` that takes its `else` sets the id variable to the null string. Their mode bits stay reserved |
| three more `option` words | Removed | `chain.keep.common` (`chain` always discards the unnamed common), `backslash.not.quote` (a backslash always opens a quoted string in a command line) and `correlative.reuse` (a shorter array's missing values count as zero unless an operand says `reuse()`) |
| `inputclear` | Removed | write `clearinput`; the file type constant `FL$TYPE.SH` (a static hashed file SD never had) is gone too |
| `@am`, `@svm`, `@user`, `@user.no`, `@filename` | Removed | second names of variables that stay: write `@fm`, `@sm`, `@logname`, `@userno` and `@file.name` (a dictionary item of your own that says `@filename` stops compiling until changed) |
| `go`, `go to`, and `#` before a compiler directive | Removed | write `goto` (also in `on ... goto`) and `$ifdef`, `$define`, `$include` |
| second spellings of query words | Removed | `avg`, `enum`, `vert`, `percent`, `pct`, `%`, `header`, `footer`, `first`, `#` and `&`: write `average`, `enumerate`, `vertically`, `percentage`, `heading`, `footing`, `sample`, `ne` and `and`; a `break.on` control `l` is no longer ignored when `u` is given too; and a name that is `f` and a number is a field only if the dictionary has an item of that name |
| terminal case inversion | Removed | `pterm case invert` / `noinvert` (and the line in `pterm display`), `input ... nocaseinvert`, the `invcase` setting of `sd.conf` (a kept `sd.conf` is cleaned by the installer), `system(1001)`, `pterm(PT$INVERT)` and the `no.invert` option of `sed`; `keyinc()` is now the same as `keyin()`, and field 2 of `ttyget()` is always 0. Type in the case you want |
| `nano` | Removed | the second external editor, with its syntax file and the page about it (the shared parts moved to the `micro` page): write `micro`, `sed` or `ed` |
| `set.queue`, `sp.assign`, `getpu(pu$form.queue)`, `system(1066)` | Removed | Pick style form queues were added and taken out again in this release: use `setptr` with the settings you want |
| `copyp`, `copy.listp` | Removed | the Pick-style copy and its saved-list form (`copy.listp` was added and removed again in this release): write `copy` and `copy.list` |
| `make.index` | Removed | write `create.index` and then `build.index` (the `tool.dict` page runs both after one box) |
| `list.union`, `list.inter`, `list.diff` | Removed | the saved-list twins of `merge.list`: `get.list` the two lists and `merge.list 1 union 2 to 3` (or `intersect`, `diff`) |
| `cd` | Removed | the short name of `compile.dict`: write `compile.dict` |
| `md` | Removed | the Pick name for the VOC (a Q-pointer to `voc`): write `voc` |
| `create.file ... no.case` and `... case` | Removed | every file is case insensitive, so neither word means anything; `configure.file ... no.case` stays for converting an old file |
| `catalogue`, `$catalogue`, `no.catalogue` | Removed | the British spellings: write `catalog` (the verb, and the `$basic.options` word), `$catalog` and `no.catalog` |
| `bscan`, `root`, `key()` | Removed | added and taken out again in this release: scan an index with `selectindex`, `selectleft` / `selectright` and `setleft` / `setright` |
| `call "file name"` | Removed | the Pick style call was added and taken out again in this release: catalog the program and call it by name |
| `is.alnum()`, `is.alpha()`, `is.digit()`, `is.graph()`, `is.mark()`, `is.space()`, `left()`, `right()`, `md5()` | Removed | added and taken out again in this release: use `alpha()`, `num()` and `matches` for the character tests, `s[1, n]` and `s[n]` for the ends of a string, and `digest(s, 0, 'md5')` |
| `adds()`, `subs()`, `muls()`, `divs()` | Removed | added and taken out again in this release: the operators `+ - * /` work element by element on dynamic arrays |
| four more `option` words | Removed | `no.sel.list.query` (`no.query` alone skips the question *Use active select list?*), `catalogue.local` (`catalog` and `delete.catalog` with no word work on the private catalog; say `local`), `inherit.mfile.path` (a new multifile subfile goes in the directory named for the file unless `pathname` says otherwise) and `correlative.nocase` (a correlative is case blind through its own `A;NOCASE;` flag or an I-type's `$mode nocase.strings`) |
| `analyse.file` | Removed | the British spelling: write `analyze.file` (the American one stays) |
| `cleardata`, `clearinput`, `clearprompts`, `clearselect` | Removed | the run-together second names of `clear.data`, `clear.input`, `clear.prompts` and `clear.select` (the BASIC statements of the same names are not affected) |
| `cs` | Removed | second name of `clr`: write `clr` |
| `delete.catalogue` | Removed | the British spelling: write `delete.catalog` |
| `rename` | Removed | second name of `cname`: write `cname`. (`cname` still takes the comma form, `cname old, new`.) |
| `sp.open` and `sp.close` | Removed | write `printer keep.open` and `printer close` |
| `sd -l` | Removed | "apply new license": SD has no license, and it set a flag nothing read; `sd -l` is an unrecognized option now |
| terminal types of old hardware | Removed | the Data General, MDC Prism, Prime and Televideo 910 definitions: `dg210 dg-ansi dg-generic dg6053-old dg100 prism2 prism4 prism5 prism7 prism8 prism9 prism12 prism14 p4 p5 p7 p8 p9 p12 p14 pt100 pt200 wren fenix pt100w pt200w wrenw fenixw pt250 pt250w tvi910` (31 of the 82 names `list.term.types` showed). Use `ansi`, `vt100` or `xterm` |
| `system:account:file` file names | Removed | bit 2 of `FILERULE` and a Q-pointer's fourth field named a file on another system and opened nothing; the name form is gone and a Q-pointer with a fourth field is refused (3003) |
| eleven `sd.conf` settings, `sd -<n>`, and the `gdi` and `raw` words of `setptr` | Removed | `apilogin`, `codepage`, `creatusr`, `exclrem`, `fixusers`, `gdi`, `jnldir`, `jnlmode`, `portmap`, `ringwait` and `txchar` (the first two groups did nothing; `fixusers` and `portmap` gave a connection a fixed user number): a line naming one stops SD from starting, so the installer takes it out of a kept `sd.conf`; `config('...')` no longer answers them and `config` no longer lists them; `tool.config` has no `inert` or `retired` status left; `sd -<n>` (start with a given user number) is gone; `setptr` always uses its raw mode, and its `gdi` and `raw` words are no longer keywords |
| six `option` words and `keep.select` | Removed | `unass.warning`, `div.zero.warning` and `non.numeric.warning` (with the shortcut `sdbasic.warnings`): an unassigned variable, a divide by zero and a non-numeric value are always errors, so test the value first; `inherit`: a phantom starts with every option off; `query.sort.no.case`: a sort is by case (`sp.view ... by field no.case` still ignores it, by sorting the list itself); `clear.select` and the `keep.select` statement: a select list left active is left for the next command, and `clear.select` clears it |
| twenty-two `$mode` words | Removed | `string.locate`, `composite.readnext`, `pick.read`, `selectv`, `pick.select`, `stdfil`, `stdfil.shared`, `write.delete.then.else`, `autolock`, `pick.substr`, `pick.substr.assign`, `compatible.append`, `pick.convert`, `count.overlap`, `index.overlap`, `change.no.overlap`, `case.sensitive`, `for.store.before.test`, `pick.jump.range`, `optional.final.end`, `pick.matrix` and `heading.no.eject`: each is refused with *Unrecognized mode name*, and the program does what it did with the mode off (the `keep.select` statement went in the same way with `option clear.select`); write `heading no.eject` for the last, and name the file variable for `stdfil`. `selectv` *file* `to` *var* stays a statement. The programs that used `$mode stdfil` and `$mode selectv` had no replacement; an I-type may no longer say `$mode count.overlap`, `index.overlap`, `change.no.overlap` or `compatible.append` |
| seven more `option` words | Removed | `numeric.exponent` (`123.45E-2` is not numeric), `suppress.abort.msg` (`abort` always says where it ran), `dir.sel.omit.hidden` (a select of a directory file includes dot files), `no.phantom.notify` (a phantom always tells its parent), `force.reload`, `stacked.account` (an `execute` of `logto` stays in the account it went to) and `command.level.locks` (a file variable that goes closes its file and its locks) |
| `$mode time.ms` and `$mode default.unass.args` | Removed | write `system(1020) / 1000` for milliseconds; a default argument value needs `var.args` |
| `closeseq` and `equ` | Removed | write `close` and `equate` |
| `extract()`, `replace()`, `insert()`, `delete()` and `subr()` | Removed | write `a<f,v>`, `a<f,v> = x`, `ins x before a<f,v>`, `del a<f,v>` and `call` or `deffun` |
| `procread`, `procwrite`, `@pib`, `@sib`, `@pob`, `@sob` | Removed | leftovers of PROC, which went earlier |
| `!` (the shell verb) | Removed | write `sh command` |
| the global catalog prefixes `*` and `-` | Removed | `!` and `$` stay, and `_` names a recursive pcode program; in managed mode the SD Core server's programs are cataloged under their plain names: write `call name`, not `call *name` (see `sync.global.catalog` under New Features Unique to SD Core Solo) |
| the word forms of query operators | Removed | `equal`, `greater`, `less`, `before`, `after`, `matches`, `matching`, `not.matching` and `spoken`: write `eq`, `gt`, `lt`, `like`, `unlike` and `said` (the word `not` stays) |
| `colour`, `without`, `no.grand.total` | Removed | write `color`, `with no` and `grand.total "'L'"` |
| fourteen dead query keywords | Removed | `verify`, `verify.sup`, `exclusive`, `creating`, `inquiring`, `portrait`, `odd`, `even`, `mark`, `space`, `no.label`, `no.object`, `static` and `encrypt` were read by nothing; `group` stays |
| `setptr` print mode 5 and the `version` keyword | Removed | the terminal's auxiliary printer port; a file is made in the current format |
| old terminal definitions | Removed | `addsviewpoint`, `viewpoint`, `adds4000`, `viewpoint60`, `vp60`, `vp`, `vpa2`, `vt52`, `hpgeneric`, `hp`, `ibm-pc`, `ibm5051`, `5051`, `glasstty` and `kermit`: use `ansi`, `vt100` or `xterm` (`wyse50` and `wyse60` stay) |
| VOC record type M (menu) | Removed | the home screen is the menu system |
| `FILERULE` | Removed | the sd.conf parameter and the `path:pathname` form of file name (write an F-pointer); the `account:file` form is always on now (see FILES; in Solo it reaches only the one account's own files) |
| `listf`, `listfl`, `listfr`, `listk`, `listpa`, `listph`, `listq`, `listr`, `lists`, `listv` | Removed | `list voc with type = "F"` (and `and is.remote = ""` / `and is.remote` for local / remote files), `"K"`, `"PA"`, `"PH"`, `"Q"`, `"R"`, `"S"`, `"V"` for the other types; add `ftype desc data.name dict.name` after `voc` for the file columns |
| `sd.voclib` | Removed | nothing: the file held only the ten sentences above, and its VOC pointer is removed from every account |
| C dictionary item type (and the compiler's C-type mode, @ANS return) | Removed | write the calculation as an I-type; it may call a cataloged subroutine with SUBR(); compile.dict says message 11993 for a C item |
| A and S dictionary item types (the correlative, `a;nocase;`, an `if` in field 8) | Removed | write the item as a `d` item (a field of the record) or an `i` item (a value worked out from the record; the correlative's calculation goes here). `compile.dict` says message 11938 for an A or S item and makes no object code; `cd` refuses it; a query that names one stops with *Unexpected A type item (NAME) in query sentence*. An index made from one is no longer used, so a query on its field scans: delete it with `delete.index` |
| create.file name,part | Removed | create each part as an ordinary file (create.file part) and share the dictionary with create.file part using dict name |
| create.file ... multifile | Removed | create an ordinary file; the word is now an unexpected token |
| delete.file name,part | Removed | delete.file part (each part is an ordinary file); the multifile cancel question (6133) is gone |
| open "name,part" (BASIC) and file,part in TCL verbs | Removed | open the part by its own VOC name; a comma is part of the name, so "a,b" is only found if a VOC record is called a,b |
| VOC field 4 (multifile component list) | Removed | none; the field is ignored and the record is read as an ordinary F-pointer |
| PARSER$MFILE (!parser key 5) | Removed | PARSER$GET.TOKEN |
| error 3031 (component not found) and 3032 (no component name) | Removed | none; the numbers are retired |
| the Pick style matrix in the run-time system | Removed | nothing could make one since `$mode pick.matrix` went; the matrix descriptor type, its opcode (reserved), its flag and their branches in the kernel, the debugger and the process dump are gone, with messages 1136 and 1139, `readnext` mode 0, the wait path of a remote `write`/`delete` under `autolock`, and the option-flags field of a phantom's start-up record and its seal. No program changes |
| dictionary field numbers 9998 and 9999 | Removed | a `d`, `a` or `s` item whose location was 9998 gave the item count and one whose location was 9999 the record length in bytes; they are ordinary field numbers now. Write an I-type with `@ni` or `len(@record)` |
| Fedora support | Removed | the installer now refuses Fedora, as it already refused RHEL and its clones, before it changes anything; Debian and Ubuntu are supported. An LS1.1-3 Fedora system cannot be upgraded: install on Debian or Ubuntu |

## New Features Unique to SD Core Solo

Everything in this section is SD Core Solo's own or SD Core's own work that Solo shares. None of it is on the
openQM list, so none of it has a row in the feature spreadsheet: the administrator's pages and gates, the AI
features, SDNet, the screen library and the full-screen tools, the installer's additions, and the fixes to
SD's own code. The sections above are the openQM items that were built, with the changes they made.
### Administration

| Name | Change | What |
|---|---|---|
| `docs.install` | Added | the administrator copies a directory of PDF files in, for the `pdf` verb to show |
| `list.voc.removed` | Added | administrator only: the VOC records SD no longer ships, with the release each went in and what to write instead; given the account, what the next `update.accounts` would do in it (changes nothing) |
| `load.help` | Added | the administrator fills the file `context.help` with the text of the `[?]` help buttons; records the administrator added are kept |
| `mcp.call` | Added | the work of every call of the MCP launcher `solo-mcp.py` - the gate, the checks of every word, the work and the audit line; not typed by a person |
| `remote.api`, `remote.ssh` | Added | administrator only: `off`, `local` or `open` reports or changes whether SD listens for the API (port 4249) and for its own ssh (port 4251), through your own systemd units; with no word they report and change nothing; no firewall is changed (the command shows the line to run) |
| `restore.account` | Changed | fixed: no longer overwrites a different global program of the same name; it is left alone and listed |
| `restore.voc` | Added | administrator only: puts back the newest saved copy of a VOC record the update removed, or all of them; never overwrites a record the account holds, and audits what it puts back |
| `set.config` | Added | administrator only: changes one of 14 tuning parameters of `sd.conf`: a name on SD's list and a whole number inside the range SD enforces; the old file is kept as `sd.conf.bak` |
| `set.mcp` | Added | administrator only: opens the MCP door for an AI agent's MCP client (`set.mcp on`) and gives it the write tool (`set.mcp write on`); both off as delivered; Getting Started page 25, The MCP door |
| `set.net.access` | Added | the account's `net` gate: `set.net.access on` and `off`; off as delivered; it guards API request 29 and every remote file |
| `set.os.access` | Added | the shell (`sh`) and operating system access (`os`) are two switches on the account, both off as delivered; `set.os.access sh on`, `os on`, `off`; seen at the next sign-on |
| `set.pins` | Added | `set.pins on` lets the account add programs of its own to its home screen (`my.menu`); off as delivered |
| `set.recovery` | Added | `set.recovery on` has the IDE keep a recovery copy of unsaved text, readable only by you in `~/.local/state/sd/recovery/sduser/`; off as delivered (SD Core: `modify.account ... recovery-on`) |
| `set.start.default` | Added | `menu` or `terminal`: where a session starts |
| `set.tile` | Added | takes a home screen tile off the page or puts it back; `set.tile` alone lists what is off |
| `set.tile` standard ids | Changed | the seven development tiles and `recent` are standard ids (there is no Switch account tile in Solo); the Back tile of a sub-menu cannot be turned off |
| `set.viewer` | Added | allows one PDF reader for the `pdf` verb; none is allowed as delivered |
| shell and OS access | Changed | until now every Solo account could run Linux commands and reach any file the Linux user could; both are off until the administrator turns them on |
| `sync.global.catalog` | Changed | in managed mode it catalogs the SD Core server's programs under their plain names (`call name`; the `*` prefix is gone), takes a catalog entry with no `$ ! _` prefix for the server's, refuses a program whose name begins with `$ ! _ * -`, and sweeps away the `*name` entries an earlier release made; a program cataloged in the account under the same name is found first |
| `tool.audit` | Added | after `admin`: full-screen page that reads the audit trail from its end, a screen at a time: filters by time, event, account, user and text, go to a time, follow, the rotated files, and the lines shown saved to a record; it changes nothing else |
| `tool.backups` | Added | after `admin`: full-screen page for backups and restores: the saved directory, the archives in it with each manifest, whether another session is signed on (a backup and a restore need every other session out), and `backup.account`, `restore.account` and `set.backup.directory` behind a box |
| `tool.config` | Added | after `admin`: full-screen page that shows every `sd.conf` parameter with what SD is running with, what the file says, the default and when a change takes effect, and names a line SD does not know; `E` changes one of 14 tuning parameters through `set.config` behind a box answered Run, and shows why and the line to add by hand for the rest |
| `tool.gates` | Added | full-screen page of the account's four gates (`sh`, `os`, `net`, `ai`): on or off for this session, what the register says, how each is delivered and when a change reaches you; after `admin`, Enter turns a gate on or off through its own verb |
| `tool.menus` | Added | full-screen page that shows the home screen's settings and runs the verbs once you answer Run |
| `tool.menus` | Changed | a row for each development tile and the Recent group in the Menu box |
| `tool.remote` | Added | after `admin`: full-screen page for everything that reaches SD from outside: the API (port 4249) and ssh (port 4251) doors as the machine reports them (changed after a box, through `remote.api` and `remote.ssh`), the SDNet servers (delete, define with the terminal given to `set.server`), who is connected and this machine's TLS identity |
| `tool.sessions` | Added | full-screen page of sessions and locks; logs a session out, releases a lock, stops new logins |
| `tool.viewers` | Added | the administrator's page for the PDF reader that `pdf` may use |
| `update.accounts` deletion list | Changed | after the copy from `newvoc` it removes from the account's VOC the records on SD's deletion list (`sdsys/voc_removed`), but only the record SD shipped (not one of your own, not a `[locked]` one unless it is a verb), after saving a copy, and audits it; before `newvoc.mods` |
| `update.accounts` in the SDSYS account | Changed | the administrator's own VOC is updated from `voc_template`, not `newvoc` (no visible change in Solo today: the two sets hold the same `tool.gates`) |
| `update.accounts` that cannot write the VOC | Changed | the account is left as it was with a line that says so (its file permissions refuse the write), and the command ends with its counts, where it stopped with error 3018 |

### AI

| Name | Change | What |
|---|---|---|
| `agent-pack` | Added | a guide (examples compiled and run), a generated reference of everything the product has and a Claude Code skill, in the source and in the install |
| `ai` boundary | Added | the AI is never given SD's own files (decided by where a file is: the system directory outside the account, the helper's, a `gplsrc` directory) and a question that names SD's own code or build (`gpl.bp`, `gplsrc`, `$internal`, kernel files) is refused before anything is sent, for every provider and after `admin` too; the second is a screen, said to be one |
| `ai` documentation passages | Added | the AI line adds up to four matching sections of the installed User documentation to the request (the Getting Started pages, which hold Solo's administration, only for a session that typed `admin`: `docs/admin` is mode 700); the preview names each page and section, shows its first lines and its size, and Cancel is still the default; the `ai` verb adds none |
| `ai` follow-up | Added | every AI answer has a Follow up button (never the default; it applies nothing): the next request from the same place carries that question and answer, shown in full in its preview and sent as earlier turns; used once, the last three at most, dropped after ten minutes |
| `ai` gate | Added | `set.ai gate on` and `off`: the account's `ai` gate; off as delivered |
| `ai` in the editors | Added | the debugger, `sed` and `ced`: ask about what you are looking at; a preview of everything sent; proposed changes shown as diffs and never saved by the AI |
| `ai` preview | Changed | shows the provider, the size of all that is sent and what is yours (the request, a record's lines, a selection, earlier questions); SD's own text (its instruction, the page's actions and description, documentation passages, the standing text) is sent and counted but not shown; a preview taller than the screen scrolls with PgUp and PgDn (the bottom border says which lines are shown), and Send and Cancel stay on the screen |
| `ai` proposal box | Changed | shows every line of a proposed change, command or request in full, not cut at the box edge; a proposed change that would take more than 30 rows to show is not offered |
| `ai` row in the dictionary form | Added | the IDE's dictionary item form has an AI line: say what the item should be; the values the AI proposes are shown in full (Reject the default) and only those that pass the form's own checks go into the rows; nothing is written until the form is accepted and the IDE saves; an item of SD's own files is never sent |
| `ai` row in the home screen | Added | type what you want, see what will be sent, accept one proposed tile or command |
| `ai` row in the ide | Added | explain an error or a selection, ask about the record, accept or reject proposed edits as a diff |
| `ai` row on a toolkit page | Added | every page of the toolkit but `tool.keys` (the development pages, `my.menu` and the administrator's pages): F6 (the line is the stop after the page's last box), AI > Ask the AI or a click puts the cursor in a line above the status line, which takes one row from the list; the model is offered the page's keys that change what is shown and, on `tool.dict`, `tool.files` and `tool.catalog`, new and compile (Reject is their default; a name the AI gives is only the page's own question's starting answer), and a proposal is carried out by the page's own key; with the AI off the page is as before |
| `ai` standing text | Added | a few lines sent first with every AI request (what SD Core Solo is, its conventions and gates, where its help is, your own facts); SD ships a default; after `admin`, `tool.ai` T edits it (Save, Shipped default), `set.ai context` at the prompt; kept in your Solo directory (`ai/context`, mode 0600); invisible in the preview and your question is unchanged; plain ASCII, 4,000 characters, and a text naming SD's own code is refused |
| `ai` verb | Added | asks an AI one question; shows exactly what will be sent and sends only on `y` or `yes` |
| `set.ai` | Added | the administrator chooses the provider, model and key from the prompt, and turns the `ai` gate on or off; `set.ai models` lists a provider's own model names, `set.ai model` says when the provider does not list the name; a provider's refusal shows its own words |
| `tool.ai` | Added | the administrator's AI choice page: Anthropic, OpenAI, Google, DeepSeek or custom, each named one with a default model, with the account's gate; S, one setup dialog: the key in a password field (handed to the helper through a file, so it passes through SD) and the model chosen from the provider's own list; C, the custom connection, is one form with its six fields in view, and a refusal names the field and keeps what was typed |

### API and client library

| Name | Change | What |
|---|---|---|
| api file administration | Added | a session signed in with the global password (that of the SD Core server that manages this Solo) may run `analyse.file`, `configure.file`, `create.index`, `build.index`, `delete.index`, `disable.index`, `create.file` and `delete.file` in the account, `fcontrol` on an open file and `write ... creating.seqkey`; a session signed in with the account password is refused each, and the refusal is audited; a Solo with no global password (as delivered) serves none |
| api request 29 | Changed | fixed: it opened a file for any login once `netfiles` bit 2 was set; it now needs the account's `net` gate (`set.net.access on`), and the OS question for a pointer out of the account |
| api requests 32 and 33 | Changed | fixed: file lock and unlock acted on the last file opened, not the file named |
| api request 70, `delete.file` | Changed | fixed: a delete the verb refused (the file open in another session, say) left the file on the disk but the client's file number dead; the number now stays good and the status is the verb's |
| api server loop | Changed | fixed: a client that hung up before its first request left a server process using 100% of a processor |
| api, a refused `sdlogto` | Changed | fixed: the session went on naming the refused account, so every later open of a file by its VOC name failed; the session now stays in the account it was in |
| client class `sdclient` | Added | the BASIC class gains `getvar`, `revision`, `system`, `trans`, `rtrans`, `txn`, `evaluate`, `evalconv`, `iconv`, `oconv`, `iconvs`, `oconvs`, `openseq`, `readseq`, `readblk`, `writeseq`, `writeblk`, `seek` and `weofseq` |
| client library: restricted | Changed | fixed two faults under `sdclient 1` and `2`: a heap overrun on a refusal, and a refused call that still ran |
| client library: `sdchecksum` | Added | a checksum computed on the client |
| `creating.seqkey` over the API | Added | the server's record-id counter gives the id, over the API; on Solo, for a session signed in with the global password |

### Commands

| Name | Change | What |
|---|---|---|
| `basic ... no.catalog` | Added | compiles and runs no catalog directive: a `$catalog` line in the source or in a file it includes, or a `CATALOG` line in `$BASIC.OPTIONS`, is not acted on and the compile says so |
| `menu` | Added | opens the full-screen menu (the home screen) from any account; `home` and `exit` stay |
| `pdf` | Added | shows a document in the PDF reader the administrator has allowed, even for an account that may not run programs |
| `tool.*` pages with several boxes | Changed | the box the keyboard is in is marked `[*]` in its top right corner and the others `[-]`, as in the IDE (`tool.bulk`, `tool.query`, `tool.test`, `tool.sessions`, `tool.menus`, `tool.ai`) |
| `tool.bulk` | Added | full-screen bulk update of one data file: records chosen by conditions, a saved list or all, up to ten operations, a dry run first, apply under record locks after a box that asks the file's name typed, and undo of the last run |
| `tool.catalog` | Added | full-screen list of the account's program records (kind, compile time, size, current or stale, private / local / global catalog) with compile (basic), catalog privately or locally (catalog) and take an entry out (delete.catalog, decatalog), each the ordinary verb after a box |
| `tool.dict` | Added | full-screen dictionary of a file (items by type and what they show; make, edit, delete, compile and try an item; the file's indices: make, build, delete, list) |
| `tool.files` | Added | full-screen list of the account's files (kind, records, modulus, size, indices, trigger, case, dictionary) with make, clear and delete (each the ordinary verb, after a box) and analyze and the index and trigger lists (the terminal is handed to the verb) |
| `tool.files` | Changed | refuses a VOC keyword (such as `all`) as the name of a file to make, clear, delete or report on |
| `tool.keys` | Added | full-screen diagnostic for the terminal and the keyboard, for everyone |
| `tool.query` | Added | full-screen query and report builder: file, columns (break and total), up to six conditions, four sort levels and options make a list / sort / count / sum sentence that is shown, checked, run into a results box (2000 lines), and can be saved to and loaded from the file `&queries&`; every sentence is checked again before it runs |
| `tool.query` | Changed | before every save and delete it checks that `&queries&` is an ordinary data file in your own account, and deletes only saved queries (`tool.bulk`'s undo file has the same check) |
| `tool.query` questions | Changed | its seven questions are asked on a form; a refused answer is explained in the form, with the form's explanation, and the form stays open with the text typed |
| `tool.query` saved queries | Changed | L shows them in the toolkit's pick window with columns (name, file, kind, sentence); typing narrows it, Enter loads, Del deletes after the name is typed (X no longer does) |
| `tool.test` | Added | full-screen test runner: lists the `test.*` programs of the account's program files, runs one, all of those shown or the failed ones (after a box) with `run`, and judges each by its last line `N passed, M failed` (pass, fail, no verdict; an abort is a fail); shows what a test printed; compiles a test that has no object code through `basic` after a box |
| `tool.test` boxes | Changed | Tab and Shift-Tab move between the list and what the test under the cursor printed, as on the other pages with two boxes |
| `tool.voc` | Added | full-screen list of the account's VOC (type, what each entry points at, whether a pointer works, how it differs from the shipped VOC) with edit and delete after a box; a change to the VOC needs `admin` |
| `tool.voc` | Changed | Delete no longer offers `delete voc all` or any other VOC keyword as an entry name |

### Data formats

| Name | Change | What |
|---|---|---|
| `convert.data` | Added | converts a file between formats |
| CSV functions | Added | `csvparse()`, `csvbuild()`, `readcsv`, `matreadcsv`: read and write whole CSV texts and rows with another delimiter, quote and escape, including a field with a line break |
| `import.*` verbs | Added | `import.csv`, `import.json`, `import.xml`: read a CSV, JSON or XML file into the records of an SD file |
| `json` keyword | Added | queries can write JSON |
| JSON merge, patch, path | Added | `jmerge()`, `jpatch()`, `jpath()`: RFC 7396 merge, RFC 6902 patch and JSONPath (RFC 9535) |
| `readjson`, `writejson` | Added | read and write JSON Lines through `openseq` |
| XML functions | Added | `xparse()`, `xbuild()`, `xpath()`: XML into a collection and back, and a path query <!--s 515--> |

### Encryption

Three simple layers, with no vault and nothing on by default.

| Name | Change | What |
|---|---|---|
| `encrypt.key` | Added | asks for a key on a hidden prompt and puts it in the key ring; `set` refuses a passphrase of fewer than 12 characters (`sdkey()` takes any that is not empty) |
| key ring subroutines | Added | `!keyset`, `!keyclear`, `!keylist`: fill, empty and list the session's key ring, kept in the memory of its own process |
| `sdkey()` and friends | Added | `sdsalt()`, `sdkey()`, `sdrandkey()`: a salt, a key from a passphrase, and a random key for `sdencrypt()` and `sddecrypt()`, open to every program; every `sdencrypt()` uses a fresh random nonce <!--s 504 g116 g123--> |
| `xe` conversion | Added | a dictionary item with `xe` (or `xe;name`) shows a field decrypted for a session that holds the key and the cipher text for one that does not; `iconv` encrypts |

### Files

| Name | Change | What |
|---|---|---|
| group size of a new file | Changed | `grpsize` is 4 (kb) in the shipped `sd.conf`, and the built-in value when it is absent; it was 2 and 1. Fewer groups overflow: measured in SD Core, 5 to 11 percent of the groups at 2 kb and under 2 percent at 4 kb |
| `index` and the record id | Changed | fixed: an index followed an upper-case copy of the id; it now follows the id as the file stores it |
| index, deleting many records | Changed | fixed: in an index of three levels, removing the last value of a block took its whole branch out of the index (lookups missed records that were there, deleted values stayed listed), and a branch left with one block left its neighbors pointing at a freed block |
| index on a right justified item | Changed | a new one keeps its keys in one order, the empty key, then numbers by value, then text, so a scan and a lookup agree when the item holds both numbers and text; building it no longer loses ids under keys equal as numbers (1, 01, 001); an existing index keeps its old order until it is made again, and an older SD cannot open a file holding the new kind |
| record id counter | Changed | fixed: `write ... creating.seqkey` could give an id out twice after another write of the file header |
| `sdfix`, `sdidx`, `sdconv` | Changed | fixed several faults (see the changelog); `sdidx -m` says why it cannot make the directory |
| sort of equal or ordered keys | Changed | fixed: a long run took minutes; 100,000 equal keys took 168 seconds, now a fraction of one |
| sort that uses work files | Changed | fixed: equal keys came out in an order that depended on `sortmem` and `sortmrg`, and a `srt$unique` sort could keep a duplicate that was not the first to arrive; it now gives what the same sort gives in memory |

### Installer

| Name | Change | What |
|---|---|---|
| access questions | Changed | the API and ssh questions are SD Core's two: *Allow API access from other computers? (y/N)* and *Allow ssh access from other computers? (y/N)*, where `off/local/open` was typed before. A no leaves the API off and ssh on for this computer only, as the old defaults did; a y opens each to the network. `--api` and `--ssh` still take `off`, `local` or `open`, and `remote.api` / `remote.ssh` change them afterwards |
| agent pack | Added | the installer puts the guide and the knowledge pack for AI agents in the installation's `agent-pack` folder, when the download's pack matches the installed release; an absent or other-version pack is a note, never a stop |
| changes document and manuals | Added | the changes document (`sd-core-solo-for-linux-<version>-changes.pdf`, `.html`, `.md`) and the manuals found beside the installer are put in the installation's `pdf` folder, the manuals through `docs.install`; anything else is left alone |
| distributions | Changed | Debian and Ubuntu (and their derivatives) only: Fedora is refused, as RHEL, its clones, Arch and openSUSE were, and every refusal now comes before the network check, the questions and the first `sudo`; the `dnf` step is gone |
| documentation for the AI | Added | the installer fetches the documentation pages from `SDCore4LinuxSoloDocs` into `docs/user`, where `ai` looks for the passages it sends with your question; a fetch that fails is a note, never a stop |
| installer packages | Changed | also installs `gpm`, the console mouse daemon, so the full-screen tools have a mouse on a bare Linux console; the package starts it, the installer does not, and removing SD leaves it; if `gpm` cannot be installed (a container, say) the installer warns and goes on |
| installer source message | Changed | says which source it is downloading, the release tag or `main` |
| linger | Changed | on by default for every install, an unattended one included, so that SD stays reachable over the API and ssh when nobody is signed in; the question on a terminal defaults to yes; `--no-linger` (new) or the control file's `enable-linger=no` leaves it off. An upgrade leaves linger as it was |
| release package source | Added | a release zip carries Solo's complete source in a `source` folder with its `SHA256SUMS`; the installer (and an `--upgrade`) checks every file against the list before it asks anything, builds from that folder instead of downloading it, takes the AI line's documentation from it, and stops on a changed, missing or unlisted file. A package made from a release-candidate tag is named a pre-release at the start and the end |
| os-access question | Added | the installer asks whether to turn on the shell and OS access, each answer no unless yes; `--os-access none`, `sh`, `os` or `both` answers it without a terminal; an upgrade does not ask and keeps the two as they are (a Solo from before the switches, LS1.1-3, has neither set, so both are off; type ADMIN, then `set.os.access`, to turn them on) |
| source not installed | Changed | the install compiles SD and removes `gpl.bp`, the C source and objects and the build files; an upgrade removes what an earlier install left; the complete source is in the release zip's `source` folder |
| start question | Added | a fresh install asks whether sessions start at the home screen or at the command prompt (the home screen unless you say otherwise); `--start menu` or `--start terminal`, or `start=` in the install file, answers it without a terminal; an upgrade asks nothing and keeps what the site had, and keeps the prompt for an install from before the home screen; `set.start.default` changes it later |
| uninstaller `--keep-data` | Changed | keeps what SD Core for Linux keeps with its accounts: the passwords, the audit trail, the TLS and ssh host keys, the AI settings and keys, the SDNet logins and where sessions start; a reload puts them back and asks for no password (password files given with the reload replace them) |
| uninstaller and the systemd user units | Changed | fixed: `deletesdsolo.sh` run from a shell that cannot reach the user manager (ssh command, cron) deleted the installation and left the enabled units; it now finds the manager's socket itself, removes the unit files if `solo-service.sh remove` is refused, and stops before removing anything of the installation or your data if a unit file is still there |

### Screen library

| Name | Change | What |
|---|---|---|
| accelerator keys | Added | `accel 'alt-s'` on a control; `&` marks the Alt-letter and the letter is drawn underlined |
| bracketed paste | Added | the library asks the terminal for it: pasted text arrives as one `paste` event and runs nothing; a text area takes it as one undo step, an entry its first line |
| break key | Changed | Ctrl-C comes back after an abort inside a full-screen page (the setting a page took is kept and given back at the next prompt) |
| characters of several bytes | Changed | on a UTF-8 terminal a character of two to four bytes is followed by blanks so that it takes as many cells as it has bytes: the columns after it stay in line and a redraw leaves nothing behind |
| check and slider labels | Changed | the label goes where `place` says, on the left by default |
| `color fg,bg` | Added | on the controls of a screen: label, entry, combo, check, slider, radio, list, button, progress |
| containers and viewports | Added | `screen container`, `screen viewport`: a form bigger than its window: nesting, clipping, scrolling by keys, wheel and bars |
| focus in `mono` | Changed | the role `focus` is reverse, bold and underline (it was plain reverse, the look of the selected row), and in `paper` it is underlined too |
| key decoding | Changed | a graphical terminal's F1 to F12, arrows, Home, End and the modified keys (`ctrl-left`, `shift-up`, `ctrl-pgup` ...) are read in both cursor-key modes; an unknown control sequence is ignored, not typed; a byte of 128 or more is never a command; a cut-short mouse report is ignored, not taken as the end of the input |
| Linux console keys | Changed | on the Linux text console Shift-F1 to Shift-F8 (which the kernel sends as F13 to F20) and Shift-Tab (which it sends as Esc Tab) are read as those keys, for the terminal type `linux` only; the IDE's previous pane and the editor's find previous work there |
| Linux console mouse | Added | on a text console SD takes the mouse from the gpm daemon (terminal type `linux`, input a text console, not with `nomouse`) and gives programs the same reports an emulator gives; the installer installs `gpm`; no mouse while the daemon is stopped, and not in SED |
| redraw | Changed | the library writes only the text that changed, so a screen no longer flashes |
| resize inside a dialog | Changed | a resize that arrives while a question, a menu or the AI box is open is kept and given to the page afterwards; it used to be lost |
| `screen form fill`, `stretch` | Added | a form that takes the whole window and follows it: a text area or table with `stretch` gets the spare room and what lies beyond it moves, with no code in the program; below the form's own size one warning holds the keyboard |
| `screen memo` SED's editing keys | Added | Alt-Backspace and Alt-D delete a word, Alt-E cuts to the end of the line, Alt-T swaps two characters, Alt-U / Alt-L / Alt-C the word's case, Alt-Space closes blanks, Alt-A aligns with the line above, Alt-M swaps the cursor and the mark, Ctrl-Up / Ctrl-Down move the view |
| `screen msgbox` | Changed | a text taller than the screen scrolls with PgUp, PgDn, Home and End, the bottom border says `lines a-b of n, PgUp PgDn`, and the buttons are always on the screen; the confirm boxes of the toolkit pages are such boxes |
| `screen selector` | Added | a choice of one word among several |
| `screen set` | Changed | a `value`, `text` or `items` clause written as `''` empties the control, a clause left out changes nothing, and a written `''` on `enabled`, `wrap`, `readonly` or `visible` changes nothing; programs compiled before need compiling again |
| `screen slider` | Added | an on and off switch with words of your choice |
| `screen` statement | Added | menus, dialogs and forms from BASIC; color; tabs, text areas, tables; the mouse |
| terminal type at sign-on | Changed | taken from the terminal's `TERM`: xterm, xterm-256color, VTE, Konsole, tmux, screen, alacritty, kitty, foot, rxvt and st are `xterm`, the console is `linux`; the account's login paragraph no longer forces `TERM LINUX` |
| text area keys | Changed | `screen memo` and the IDE's editor: Ctrl-A selects all (Home is the start of the row), Ctrl-O no longer copies (Ctrl-C does), F3 finds the next and Shift-F3 the one before (Ctrl-N found the next), Ctrl-D duplicates the line (Delete joins lines) and Ctrl-K cuts the whole line into the clipboard, where it cut to the end of the row |
| text area undo | Changed | `screen memo`: Ctrl-Z gives back exactly what was there; a record holding value, subvalue or text marks came back changed |
| text from data | Changed | the library never sends a control character to the terminal: one in a record, a record id, a file name or an AI answer is drawn as a reverse letter (ESC as `[`), one cell each; UTF-8 text and line drawing are unchanged |
| theme roles | Added | six new styles, `warning`, `success`, `readonly`, `busy`, `focus` and `hotkey.sel` (the key letter on the selected entry of a menu), with a look in every theme (bold, underline and reverse only in `mono`), and `error` gets a `mono` look; programs that include `sl.h` need compiling again |
| themes | Added | `paper` (black and dark blue on light gray, menus the other way round; the delivered theme), `night` (white and yellow on black), `mono`, `classic` and `blue`; color works on the Linux console terminal type |
| `!tk` routines | Added | `!tkopen`, `!tkclose`, `!tkkeys`, `!tkframe`, `!tkmenubar`, `!tklayout`, `!tkconfirm`, `!tkform`, `!tkpickrec`, `!tkout`, `!tkwrap` and the header `tkpub.h`: a program of your own builds a full-screen page with the frame, menu bar, F2 finder, key table, confirm box, form and pick list the tools use; it runs with your own rights and gates and runs nothing itself |
| window size | Changed | SD reads the terminal's window size at the start and when it is resized; `@crtwide` and `@crthigh` follow it |

### SDNet

| Name | Change | What |
|---|---|---|
| `list.servers`, `delete.server` | Added | the administrator's list of defined servers, and removal of one <!--s g52 g26--> |
| `net` gate | Added | `set.net.access on`: one of the four things that must all be true before a remote file opens (the gate, a defined server, your login on it, a VOC pointer) |
| `netwait`, `netreply` | Added | `sd.conf` settings, 1 to 3600 seconds, for a connection and a reply; a silent server is given up on <!--s 235 g291--> |
| remote file administration | Added | for a login the server lets administer its files (on SD Core, one in its group `sdfilesadmin`): the file and index verbs, `fcontrol` and `write ... creating.seqkey` on a remote file; `create.file ... server` makes the file there and the pointer here, and `delete.file` of a pointer deletes both |
| remote files, a pointer | Added | a VOC entry whose first field is `n` points to a file on another server |
| remote files, cache | Added | a short record cache |
| remote files, index | Added | `indices`, `selectindex`, `setleft`, `setright`, `selectleft` and `selectright`; a query uses the other machine's index |
| remote files, query | Added | `list`, `count`, `sort`, `select`, `list.item`, `list dict` on a remote file; `fileinfo` type 7 |
| remote files, refusal | Changed | the server's own words are shown, not `status 0`; a lost connection is an error <!--s 256--> |
| remote files, reply checked | Changed | a reply that cannot be true is a lost connection at once |
| remote files, write | Added | for a server defined `write`: `write`, `delete`, `clearfile`, the locking reads and release |
| remote lock functions | Added | `recordlocked()`, `getlocks()`, `filelock`, `fileunlock`: work on a remote file |
| remote `select` | Changed | fixed: a server error stopped the program with an abort |
| SDNet logins | Added | `set.netlogin`, `delete.netlogin`: a person's login on a server, kept as derived values and never the password |
| `set.server` | Added | defines a server; `connect` shows its key hash for you to confirm <!--s g84 147--> |
| tls reads | Changed | fixed: a half record or a late ticket made a read wait without limit |

### Security

Fixes that close a way round a gate, and what the gates now ask.

| Name | Change | What |
|---|---|---|
| api `clearfile` | Changed | fixed: it cleared whichever file a stale counter pointed at; it clears the file named |
| audit trail rotation | Changed | fixed: `sd-solo -start` renames a trail of 1 MB or more to `audit.<date>-<time>`, a name with a one-second resolution, and the rename replaced a file already called that; a rotation onto a name that is taken is now refused, the trail stays whole and SD starts as usual |
| commands that change a file you name | Changed | fixed: `get.list ... deleting`, `save.list`, `delete.list`, `sort.list`, `copy.list`, `map file`, `format`, `fixdir`, `generate`, `compile.dict`, `set.trigger`, `reformat ... to`, `sp.view`, `como ... in`, the object file of `basic` and `create.file` of an existing file, and over the API `write`, `delete` and `clearfile` could be pointed at the VOC or a server file; each now asks first |
| deleting a directory tree | Changed | fixed: `delete.file` of a directory file and the other deletes of a whole tree followed a symbolic link inside it and deleted what it pointed at; a link is now removed as a link and its target left alone |
| `$internal` programs | Changed | fixed: a program a user compiled and cataloged locally ran with internal privilege without `admin`; only the global catalog or an `admin` session keeps it |
| named common | Changed | fixed: a program that ended while holding objects in named common freed them twice |
| named common with a `$` or `_` name | Changed | the system's own common blocks: only an internal program may declare or join one; any other program stops with message 11598 |
| operating system paths | Changed | `dir()`, `select.dir`, `oscopy`, `!oscopy`, `!ioscopy`, `deleteseq` of a path, `opentemp`, `digest` of a file, `setptr ... as pathname`, `setpu`, the embedded Python and a VOC pointer out of the account each ask the account's `os` switch |
| phantom debugger messages | Changed | `pdebug` and the phantom it debugs leave each other their messages in `.local/state/sd/pdebug` below the user's Linux home (a private directory, owner-only, found through the password database), not in `$ipc`, which any session could write; message 11948 when the store cannot be set up |
| phantom start-up records | Changed | a phantom runs the command, `DATA` lines and log file name of its start-up records in `$ipc` only if the process that started it sealed them (HMAC-SHA256 under a secret given only to the new process, in its environment); a record that does not check is refused, audited as `phantom refused` and reported to the parent; the phantom debugger's conversation is not sealed |
| print file refusal | Changed | outside the account: the message names the account instead of printing `%1` |
| `setptr ... spooler`, `setpu` | Changed | fixed: a print unit's spooler string ran as a shell command for any session; it is run only for a session with the `os` switch |
| shared memory modes | Changed | the shared segment and the semaphores are mode 0600, the user's alone; they were 0666, open to every local user, and the daemon `sdlnxd` took the directory it runs `sd-solo -cleanup` from out of that segment. It now finds the directory from its own path and starts the program without a shell. A Solo already running keeps its old segment until it is stopped and started |
| `show ... eval` | Changed | compiles its expression in memory; it no longer writes a temporary I-type into the file's dictionary (a session holding that record made `show` wait) |
| the VOC's own files | Changed | a session without the `os` switch cannot write, make or delete under the account's `voc` directory through an operating system path |
| tls close | Changed | fixed: closing a TLS connection no longer ends the process when the other side has already gone |
| `u` conversion names | Changed | `iconv` and `oconv`: fixed: a name that begins with a dollar sign called an internal subroutine from an ordinary program; it is refused now |

### System

| Name | Change | What |
|---|---|---|
| clean-up daemon restart | Added | `sdlnxd`, which clears down sessions whose process has gone, runs as its own user unit, `sd-solo-lnxd.service`, which starts it again two seconds after it fails (`kill -9`, a crash); a stop (`sd-solo -stop`, exit 0) is not a failure and is not restarted; systemd allows five starts of the unit in a minute, of any kind, and refuses the sixth until `reset-failed`; `sd-solo.service` runs `sd-solo -start -nodaemon` and wants it; a second daemon ends at once; an upgrade writes the new units over the old ones (`solo-service.sh daemon`) |
| compiled kernel | Changed | built with optimization (`-O2`, with `-fwrapv` and `-fno-strict-aliasing`) where it had none: string and `locate` work about 3.4 times as fast, a BASIC loop 1.5 times, index builds, sorts and writes 1.15 to 1.2 times (measured in SD Core); `_FORTIFY_SOURCE=3` and `-fstack-protector-strong` end a session at a buffer overrun instead of letting it corrupt memory |
| default terminal | Changed | 140 by 43 |
| message numbers | Changed | Solo's own messages 11040 to 11054 are now 14974 to 14988, and 11060 to 11065 are 14989 to 14994, with their text unchanged, so that SD Core's numbers mean the same here; the ssh key request refuses with 14975, 14976 and 14977 |
| other sessions' text | Changed | fixed: the text of another session (the id of a record it holds locked, in `getlocks`, `system(1083)` and the `pstat` dump; the command, program names and `@pstat` that `pstat`, `hsm` and `profiler display` read) is made safe to print; control characters other than a tab and new line are shown as `?`, so that text cannot move your cursor or change your screen; `kernel(K$SHOW.SAFE, text)` gives a program the same rule |
| `sd-solo -start` | Changed | a script that captures its output through a pipe (`x=$(sd-solo -start)`) gets it and goes on; the background daemon kept the pipe and the script waited until `sd-solo -stop`. A phantom and the STARTUP command give back a pipe the same way; a file or a terminal is kept |
| `sd-solo -status` | Changed | fixed an overrun on a server that recorded no start time |
| `sd-solo -stop` and `sd-solo -restart` | Changed | a session still attached ten seconds after the termination signal is ended with SIGKILL, and the stop says how many; before, it was left running with SD gone under it. A session idle at a terminal or on a pipe leaves by itself |
| session whose SD was stopped | Changed | ends at once with a message in syslog and on its terminal, instead of spinning at full processor use for ever |
| sign-on banner | Changed | `SD Core Solo for Linux, the Essential Multivalue String Database, version LS2.0-0` (it said `SD Core Solo for Linux, ...`) |
| the TLS relay's last reply | Changed | fixed: the reply of a refused API login could be lost when systemd ended the relay as sd exited; sd now lets the relay deliver it (bounded) before it leaves |
| upgrade: retired files and `sd.conf` parameters | Changed | an upgrade deletes the files this release stopped shipping and takes the retired parameters out of a kept `sd.conf` (SD would not start on them); `deletesdsolo.sh --keep-data` keeps the saved VOC copies (`voc_removed.saved`) for the next install |

### Tools and the home screen

| Name | Change | What |
|---|---|---|
| 140 by 43 layout | Changed | the collection editor, `ide`, `sed` and the debugger: 140 by 43, refused below it, survive a window made smaller, take no keys while the cursor is off the screen |
| `!ced()` data mode | Added | edits a multivalue record as a tree of fields, values and subvalues (a text or a list; a list never below the second level), so that nobody types a mark; started by `ide` F12 |
| `debugger` full-screen display | Added | on a terminal that can address a cell and clear itself (xterm, the Linux console, vt100, vt220, ansi) the debugger takes the whole screen, 140 by 43, in the frame of the other full-screen pages: the bar with the app and the audience, the source with a gutter (`>` the line about to run, `*` a breakpoint) and SD BASIC color under the blue, dark and light themes, an Autos pane (the variables the line uses, the last stop's, the watched one, the rest), the messages, the command line, a status line, a border and a function-key row; F1 help, F4 the program's screen; a smaller window gets line mode, and so does `dumb` and `unknown`; where there is no alternate screen the screen is cleared for the debugger and again when it lets go: a program's SCREEN forms are drawn again, its `PRINT` text and what was there before are not; `LINE.MODE` in `$debug.options` keeps line mode |
| `debugger` keys F2 and Ctrl-Q | Changed | F2 is Find an action and Ctrl-Q stops the program and leaves, as on every page; Abort, which was F2, is on the Run menu (F2 still aborts in line mode) |
| `debugger` menu bar and keys | Added | the top row is a menu bar (Run, View, Help; Esc on its own or F10); **F9 runs** (it was F5), F5 and Ctrl-L draw the screen again, Ctrl-T the theme, Ctrl-K the keys; on the command line, delete to the end and swap two characters are Esc K and Esc T (they were Ctrl-K and Ctrl-T) |
| home screen | Added | a session on a terminal starts in a screen of tiles (the editor, the IDE, the record editor, a terminal, the keys test, sign out); arrows, Tab, letters, mouse, F5, F10; there is no Switch account |
| home screen `ai` line | Changed | Enter alone accepts a proposal that only opens one of the delivered tools but rejects a proposal of one of your own programs or of the Terminal tile (decided for the proposal, not for the page); it cannot run a command when the account's Terminal tile is off |
| home screen at the same time | Changed | fixed: a session that started at the home screen kept the next session waiting at sign-on (the missing `start.default` was read with a lock that was never let go); `set.start.default`, `tool.menus`, `tool.viewers` and `osread` close what they read the same way |
| home screen Ctrl-Q | Changed | on a screen typed as `home` at the prompt, Ctrl-Q goes back to the prompt (it signs out on a session's own screen, as before); there Sign out is Ctrl-X, the File menu or the tile |
| home screen keys | Changed | the Terminal is Alt-T (it was Ctrl-T); Ctrl-Q is the one key that leaves, for the prompt after `home` typed there and otherwise by signing out; Ctrl-T is the theme and Ctrl-K the keys of the screen, as on every page; Go > Start the tile (Enter) |
| home screen menu bar | Changed | made from the page's key table like every other page's: each entry shows its key (new: Alt-T Terminal, Ctrl-R Recent group, Ctrl-K Keys, F6 the AI line), F10 left the foot row, and / finds a tile by any part of its name |
| home screen Recent group | Added | `View > Recent group`: the tiles started last, newest first, six at most; off as delivered, the person's own record in `&menus&`; `set.tile` with `recent off` is the administrator's wall |
| home screen refused tile | Changed | a tile whose command is refused shows its message until a key is pressed, and the home screen no longer ends |
| home screen small window | Changed | a session that starts in the home screen is signed out when the window is too small for it, even with the Terminal tile on (it used to be left at the prompt) |
| home screen sub-menus | Added | a tile can open a second page of tiles with a Back tile, Esc and the path on the status line; eight deep; the delivered page declares none yet |
| home screen version | Added | the date line in the header ends with the product and its version: `SD Core Solo for Linux LS2.0-0` |
| `ide` | Added | tree, tabs, compile errors table, find, run, catalog and debug; Ctrl-E zooms the editor to the whole screen, and Go > Edit in SED hands a record to `sed` |
| `ide` autologout countdown | Added | with `autologout` set, the status line counts down the last 60 seconds before the session is signed out and says whether unsaved changes will be lost; any key stops it; it reads the inactivity clock `autologout` itself uses |
| `ide` breakpoints | Added | Shift-F7 or a click in the gutter marks a line of a program tab (`*`); F7 runs to the first mark, Build / Debug from the first line stops at line 1 with them set; what the debugger still has comes back as marks; up to 20, only lines where a statement begins |
| `ide` collection-file records | Added | a record of a data collection file opens as the text of its JSON (one element to a line) and Ctrl-S parses it and writes the collection; text that is not a JSON object or array is refused with the record untouched; F12 hands the collection to `ced`; a missing record opens as a new `{}`; compile, SED, Save as and the include do not apply; a text is not saved into a collection file under a new name |
| `ide` command line records | Added | `ide file id1 id2 ...` opens a tab for each record named (one that is not there is a new record) and `ide file *` lists the records of the file as a select list does, the first ten in tabs; both work after `dict`; an id is a word of the command |
| `ide` dictionary item form | Added | `ide dict file item` and Return on an item in the tree open a dictionary item as a form (`/text` asks for the text; Go, Dictionary item as form, or `/form`, shows it for the open tab): the Type is an entry (D, E, I, L, PH, X) and when it is entered the rows of that type appear, each a label and an entry (Field #, Conversion Cd, Column Heading, Len & Just such as `30 L`, ...), an existing item with its values filled in; each value is checked as you leave it; Accept writes the item through the IDE's own save and Cancel leaves it as text; an item the form cannot show stays text |
| `ide` dictionary items and VOC records | Added | `ide dict file item` and `ide voc record` edit them; the tree shows a file's dictionary as a node of its own; a save to the VOC needs `admin`, as every change to the VOC does in Solo; they show marks as tokens; `creating.seqkey` takes the file's next sequential id; a record that does not exist opens as a new one; an active select list opens its first records |
| `ide` edit as fields | Added | F12 or Go, Edit as fields: the record is shown as fields, values and subvalues in a pane of the tab inside the IDE's own form (the collection editor's engine; the tree, the tab strip, the menu and the messages stay alive, each tab remembers its view, and a resize keeps it); each change is in the tab's text at once, Ctrl-S is the IDE's save, F12 shows the text again (one Ctrl-Z takes the visit back) and the commands that work on the text wait while it is up; the pane writes nothing |
| `ide` Edit menu | Added | after File: Undo, Redo, Cut, Copy, Paste, Select all, and the fields pane's Insert, Delete, Rename and Change the type, each doing exactly what its key does |
| `ide` editing commands | Added | Search, List matching lines (SED's VIEW: the lines that hold a text, in the table), Query replace (Yes, No, All, Quit; one undo step), Find options (match case, whole words), Upcase and Downcase the selection, Import a record here, Export the selection; Build, Run a command (its output in the table; a command that aborts is stopped and the IDE stays up); View, Refresh; none writes a record. In a record shown with `:@vm:` marks Replace changes the record's text, never a mark's token; a text to find is at most 256 characters |
| `ide` fields view, AI line and the kind of a new record | Added | in the fields view a request typed in the AI entry, and the AI menu's explain-the-selection (the element the table has selected), go to the model with the record as indented JSON, and what is accepted changes the fields; on the empty top level Ctrl-T asks whether the record is a collection or an array |
| `ide` filter and pages of records | Added | Go, Filter the records: a pattern or a `like` / `with` query for what the tree lists; a list over 2,000 records goes by pages |
| `ide` go to line | Changed | takes line,column, and field,value,subvalue in a record shown with `:@vm:` marks, as well as a line |
| `ide` import of an operating system file | Added | Search, Import a record here takes `path:/full/name`, from the tab that holds the file or from the file itself through the account's `os` switch |
| `ide` key map | Changed | the settled keys: Ctrl-S is the only save key (F2 went); F9 runs, F7 debugs, F8 compiles, Shift-F9 builds (compile and catalog); F6 and Shift-F6 are the next and previous pane (the tree has no key of its own: F6 / Shift-F6, the Go menu entry Files and records or the mouse reach it, because F3 is the text area's find next); Ctrl-PageDown and Ctrl-PageUp, and Alt-PageDown and Alt-PageUp for a terminal that keeps the Ctrl keys for its own tabs (GNOME Terminal), are the next and previous tab; Ctrl-N is New record, Ctrl-O Open record (a record that exists; names with a dollar sign are refused) and Ctrl-W closes the record (it asks when there are changes); Save as, Write a copy and Find in the records are menu entries with no key; the tree no longer lists the `$` entries |
| `ide` keyboard macros | Added | Alt-R starts and ends recording the editor's keys, Alt-P plays them; both on the Edit menu |
| `ide` keys every page has | Changed | F5 and Ctrl-L draw the screen again, **F9 runs** (it was F5), Shift-F9 compiles and catalogs (it was F9), Alt-G goes to a line (it was Ctrl-L), Ctrl-T the theme, Ctrl-K the keys; the fields pane's change of type is Alt-T and the editor cuts a line with Alt-K |
| `ide` language of a record | Changed | chosen as `sed` chooses it - the file name, then the record id, then the first line (`#!/bin/sh`), then the languages a site added - where it was the suffix of the id alone |
| `ide` make operating system files | Added | New, Save as, the first save of an untitled buffer, Write a copy and Export the selection take `path:/full/name` and make a file of the operating system (mode 0666 less the umask), exclusively and before anything else changes: a file with content is never replaced (an empty one is filled), the directory is never made, and text with a NUL, a byte from 248 to 255 or over 2 MB is refused first; New opens the empty file in a tab and Save as turns the record's tab into the file's |
| `ide` marks as tokens | Added | a record that holds value, subvalue or text marks is shown with `:@vm:`, `:@sm:` and `:@tm:` and read back when saved, so nobody types a mark; a field that holds the text of a token is shown as it is |
| `ide` menu bar | Changed | its right end names the IDE and the person (`ide  name@account`), as the toolkit pages and `sed` do |
| `ide` mouse and scrolling | Added | View, Mouse switches the terminal's mouse reports off and on; View, Scroll the view up or down a line (Ctrl-Up, Ctrl-Down) moves the text and not the cursor |
| `ide` name form | Added | New record (Ctrl-N), the first save of an untitled buffer, Save as and Write a copy (both in the File menu) ask for the file and the record on one form; a refusal (a VOC record without `admin`) or a record that exists is said in the form, which stays open |
| `ide` open the include | Added | Go, Open the include opens the record that the `$include` line at the cursor names, in a tab |
| `ide` operating system files | Added | `ide path:/full/name`, or `path:/full/name` in Open's form, opens a file of the operating system in a tab; it is read and written by `!ide_osfile`, which is not an internal program, so the kernel asks the account's `os` switch for it (an account without it reaches its own directory only); saved in place (same inode, owner, mode and links), after a second look at the file (the overwrite question if it changed; a file removed or renamed over is refused) and read back after the write; CR LF, a missing last newline and tabs are kept; a NUL, bytes 248 to 255, a file over 2 MB, a directory, device or pipe are refused, a file you may not write opens read-only; locked while open; no recovery copy; compile, SED, fields, Save as and the include do not apply to it (Write a copy does: it keeps the text in a record or in a new file of the operating system) |
| `ide` operating system tree | Added | an *Operating system files* node after the account's files, only with the account's `os` switch on: Return lists a directory (directories first, 1000 entries at most, names the IDE cannot show counted) and opens a file in a tab; a directory the operating system will not let the session read says so, not "no OS access" |
| `ide` options record | Added | an X record `&ide.options&` in the account's VOC (or SED's `&sed.options&`) sets the IDE's starting modes: case, whole words, auto-indent, overwrite, tab width, line numbers, syntax color, mouse, case-blind record order |
| `ide` record locks and read-only records | Added | a record open in a tab is locked until the tab is closed; one that another session holds opens read-only and cannot be saved; an `ide` left open blocks other sessions on its records, as `sed` does |
| `ide` redraw | Changed | no flash; a click on the menu bar opens the menu; the active pane is marked `[*]` |
| `ide` replace | Changed | Ctrl-R is one undo step (one Ctrl-Z gives the text back; it forgot the undo steps), and follows the find options |
| `ide` resize | Changed | moves and sizes the IDE's parts instead of building the screen again: the editor keeps its undo steps, selection, find text and view, the messages table its row, the tree its open nodes, the AI entry its text (a part that cannot be moved, and coming back from Run, Debug or SED, still build the screen again, keeping the text) |
| `ide` save | Changed | takes the record's lock and writes on error: a refusing trigger or a full disk is reported and the text kept, where it ended the IDE and lost every tab; Save as moves the editor to the new record; a refusal is shown whole in a box; Catalog says so only when `catalog` did |
| `ide` SED's editing commands | Added | on the Edit menu and the editor's keys: delete the word before, cut to the end of the line, swap two characters, the word's case, close a run of blanks, align with the line above, swap cursor and mark |
| `ide` SED's small commands | Added | Stamp a history line, Expand tabs, Run with arguments, the character code (Alt-=), switches for syntax color and line numbers, a tab width of 1 to 99 |
| `ide` select list, recall, tabs list, terminal | Added | an active select list is asked about (No clears it) and Go, Next record from the list opens the next one; Up and Down in Open and in Run a command recall this session's files, records and commands; View, List of tabs; Build, Run a command at the terminal gives the command the screen |
| `ide` source control | Added | a save calls a cataloged `source.control` before the write, with the arguments `sed` and `ed` give it; it can refuse the write or change the record |
| `ide` spool | Added | File, Spool the text prints the tab's text without saving it: `at` printer, `lnum`, `region` |
| `ide` tab indent | Added | Tab and Shift-Tab indent and outdent the line or the selected lines by 3, one undo step each, never a tab character (the `ide` editor only; `screen memo` keeps its Tab); Tab never leaves a pane: F6 and Shift-F6 move between them |
| `ide` theme | Changed | starts in the site's `SCREENTHEME` (paper when none or a name that is not a theme, mono without color); View, Theme goes through the themes in the pages' order and skips those the terminal cannot show |
| `ide` tools menu | Added | Tools, between View and Help, opens `tool.files`, `tool.dict`, `tool.voc`, `tool.query`, `tool.bulk`, `tool.catalog` and `tool.test`; the page gets the terminal as a program run from the IDE does, and the IDE comes back with its tabs and unsaved text |
| `ide` untitled buffer | Added | a bare `ide` opens an editable "Untitled 1" with the cursor in it; New record writes nothing until it is saved; the arrows stay in the editor at its first and last line |
| `ide` window lock and foot | Changed | needs 140 by 43 (was 140 by 42) and has the toolkit's foot: the status line, a border and a row of function keys on the last three rows; a window made smaller while it runs holds the keyboard behind a warning that gives both sizes and says unsaved text is lost unless the recovery copy is on, whatever the cursor is doing (the warning is worded shorter in a very small window); `sed` is the IDE's editor and `!ced()` holds the same window lock |
| immediate messages | Changed | `message ... immediate` is shown in a box on any terminal |
| `my.menu` | Changed | its list grows with the window (more rows, a wider command column); F1, `?` or a click on the `[?]` shows the page's help |
| `sed` | Changed | the full-screen editor is back with a menu bar, the keys most editors have, undo and redo, syntax color and the mouse; its keys are decoded from the terminal's terminfo entry <!--s 365--> |
| `sed` keys every page has | Changed | Ctrl-T the theme, Ctrl-K the keys, F5 and Ctrl-L draw the screen again, **F9 compiles and runs** (it was F5); kill line is Alt-K and toggle characters Alt-T (both were Ctrl keys), go to a line Alt-G; a key of Esc and one character is written Alt-x, as on every page; View and Help menus, and four more Edit items |
| `sed` open of a missing path | Changed | fixed: opening a path that does not exist (and the retry in upper case) kept the name locked until the editor was closed |
| `sed` `path:` form | Changed | opens no file now: the `path:` form of a file name went with `FILERULE`, so `sed` asks the account's `os` switch and cannot open the file; it no longer retries the name in upper case after that check (`ide path:/full/name` opens an operating system file) |
| `sed`, `ide`, `ed` input ended | Changed | fixed: they used all of a processor for ever when their input had ended |
| `sp.view` menu bar | Added | on a window of 140 by 43 or more: File, View and Help (Esc or F10), with the keys every page has (F5, Ctrl-L, Ctrl-T, Ctrl-K, Ctrl-Q); the letters still work |
| `sp.view` screen | Changed | on a window of 140 by 43 or more (used whole) the record view has the frame of the other full-screen pages: the bar with the file and record and `sp.view  name@account`, a status line, a border and a row of function keys, in the site's theme; the keys, the spool form and the list of records are unchanged, and a smaller window or no terminal keeps the old layout |
| theme of your own | Added | Ctrl-T or View > Theme on every full-screen program takes the next color theme and **SD keeps your choice** (the record theme.*login name* of `&menus&` in the account): every page, the IDE, SED, the debugger and SP.VIEW start in it from then on; the site's `SCREENTHEME` is used until you choose |
| `tool.dict` new item | Changed | N asks the id and hands a new item to the `ide`, which shows the dictionary form and writes the record; before, the page asked seven questions one at a time and wrote the record itself, and it now writes nothing |
| `tool.dict`, `tool.voc` edit | Changed | E hands the item or the entry to the `ide` (`ide dict file item`, `ide voc name`), with the page's own boxes, where it used `sed` |
| toolkit pages: '/' finds a row | Added | on a page whose letters are commands, `/` goes to the next row that holds a text you type: `tool.sessions` and `my.menu`; on `tool.audit` it opens the filter, as F does |
| toolkit pages: command output | Added | Ctrl-O (View > Output) on every page: the commands this visit ran, what each printed or why it was refused, newest first, in a window over the page (the pages with an output pane keep it) |
| toolkit pages: confirm box and a held key | Changed | the box a letter opens waits until the key is let go, so holding R can no longer answer the box with its own Run |
| toolkit pages: every action on a menu | Changed | a key is a short way to a menu entry, never the only way: the few keys that had no entry (Enter on the home screen, the box key on `tool.dict`, `tool.query` and `tool.test`) now have one; `tool.keys` and the dialogs are the exceptions |
| toolkit pages: F2 and the right-click menu | Added | on every page with a menu bar, F2 lists every menu entry (*Menu > Entry*, typing narrows it) and a right click on a box gives it the keyboard and opens its actions (Shift-F10: the box that has the keyboard); both do what the entry's key does, and an entry that cannot be used now only says why |
| toolkit pages: F6 and Shift-F6 | Added | the one pair of keys that moves the keyboard to the next and the previous box, round and round, on every full-screen page and in the IDE, with the AI line (when the AI is on) the stop after the last box; Tab and Shift-Tab never move between boxes (in a form they go from field to field, in the IDE's editor they indent) |
| toolkit pages: menu bar | Added | every `tool.*` page (the administrator pages, the development pages and `my.menu`) has a menu bar on its top row: Esc (when the page has nothing of its own to close first), F10, Alt and a letter, or a click on a title opens it; an entry does what its key does, through the page's own checks and boxes, so the bar adds no access; Esc opens the first menu that does not begin with Quit, so Esc then Enter cannot leave the page; on the home screen's top page Esc opens the bar too |
| toolkit pages: mouse | Added | View > Mouse on every page and in the IDE stops the terminal reporting the mouse, so its own selection works, and starts it again; it holds for the session |
| toolkit pages: question forms | Added | defining a remote server, the custom AI connection, the audit filters, the replace operation of a bulk update and a `my.menu` program ask all their answers on one form (Tab between them, the first refused answer named under them); `tool.dict` T picks the record from a list; none of them changes what is checked or run |
| toolkit pages: redraw | Changed | the pages and the home screen write only what changed: a cursor key in a list sends under 2 KB, not the whole screen (about 15 KB); F5 and Ctrl-L redraw the whole page |
| toolkit pages: refused names | Changed | a name a page will not put in a command says which character is wrong ("has a capital letter (A)", "has the character &") |
| toolkit pages: the whole window | Changed | the administrator pages, the development pages and the home screen use a window bigger than 140 by 43 whole: more rows, wider boxes, wrapped explanations, tiles spread over the window; they follow a resize, also while a dialog is open; at 140 by 43 they look as they did (`tool.gates` and `tool.ai` keep fixed boxes in a bigger window) |
| toolkit pages: window limit and record-box titles | Changed | a window above 500 by 300 is refused before the screen is taken (it cost minutes); the boxes of a record write or delete are titled Write this record? / Delete this record? / Update these records? / Put these records back?; "Press Enter to return" names no screen |
| `tool.sessions` output pane | Added | Ctrl-O (or View > Output) shows the commands the page ran and what they answered, newest first, in a pane of six rows; Ctrl-PgUp and Ctrl-PgDn scroll it, Esc puts it away; it is kept in memory only |

## NO CHANGE

Items on the openQM list that were looked at for this release and needed no change to SD: the defect is not in SD, the
feature was already there, or it does not apply. They are here so that nobody looks for them above.

| Name | Change | What |
|---|---|---|
| 64-bit versions | None | no change in this release <!--s 196--> |
| `between` in a `when` clause | None | no change recorded <!--s 248--> |
| `*` comment line in a paragraph | None | already works <!--s g2--> |
| configuration file location | None | `sd.conf` is already in the SD system directory <!--s 14--> |
| `$echo` | None | already there in L1.1-3 <!--s g1--> |
| `errlog` permissions | None | the installer makes `errlog` in your Solo directory, owned by you with the mode your umask gives; no other user is given write <!--s 276--> |
| `formlist` bad data | None | no change recorded <!--s 265--> |
| hangup at the break key prompt | None | already ends the process by itself <!--s 266--> |
| inherited public variable | None | beside a function of the same name: already found <!--s 342--> |
| `insert` appending a field | None | `insert`, `ins` and `<-1>`: no change recorded <!--s 475--> |
| `itype()` | None | no change recorded <!--s 328--> |
| `list.users` | None | removed in an earlier release with the other user verbs; the administrator uses `listu` and `tool.sessions` <!--s g53--> |
| `locate` insertion position | None | also a very long numeric item: no change recorded <!--s 527 556--> |
| `loop` ... `repeat` | None | already there in L1.1-3 <!--s g59--> |
| `matbuild` trailing elements | None | unassigned elements at the end: no change recorded <!--s 557--> |
| `matchfield()` `n-mx` pattern | None | at the start of the template: no change recorded <!--s 562--> |
| memory alignment in `redim` | None | no change recorded <!--s 382--> |
| negative default argument | None | for an optional argument: the defect cannot occur, because the feature it belongs to was not in SD <!--s 324--> |
| Pick wildcards in `ne` | None | nothing to fix: the Pick wildcards (`option pick.wildcard`) were removed in this release <!--s 322--> |
| `qmbasic.warnings` | None | `sdbasic.warnings` was removed in this release with the three warning options <!--s g251--> |
| `$qmcall` | None | already there as the `$sdcall` flag, which lets a restricted client session call a subroutine <!--s g212--> |
| `readv` fatal error | None | no change recorded <!--s 559--> |
| record cache and case | None | no change recorded <!--s 330--> |
| `set.date` | None | SD still has the command; nothing was withdrawn <!--s 78--> |
| the inetd network server | None | SD opens no listening port of its own; sshd does <!--s 100--> |
| the `u` format in a report | None | already wraps a long value as `t` and `l` do <!--s 531--> |
| the user limit reached | None | login prints message 1000 and exits cleanly <!--s 275--> |
| `umask` | None | the verb and the `sd.conf` parameter were already there in L1.1-3 <!--s 5 143 g311--> |
| `update.account` | None | already there in L1.1-3 <!--s g102--> |

