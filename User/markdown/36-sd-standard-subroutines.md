Title: SD Standard Subroutines
Subtitle: The catalogued names beginning with !, which of them an application may use, and how each is called.

**This release ships 154 catalogued routines whose names begin with `!`** —
counted from the `$catalog` lines of `gpl.bp` on a Solo tree on 24 October 2026.
The figure grows as features are added, so read it as dated. Twenty-two are the
embedded Python interface (`!PY_CREATEDICT` and the rest — see
[Embedded Python](37a-sd-basic-python-integration.html); this page does not
repeat them) and forty-nine, all named `!SCREEN_...`, are the routines behind
the `screen` statement (see
[The Screen Statement](12a-sd-basic-the-screen-statement.html); they are not
called directly and are not listed here). This page names the rest that a
programmer is likely to meet. Solo has no need for several ones the multiuser
product lists — `!CREATE_USER`, `!SET_PASSWD` and the other account-creation
helpers went with the accounts — nor for the Windows-only `!ELEVATE`,
`!PS_SCRIPT`, `!DELETE_USER`, `!OS_GROUP` and `!PROFILE_DIR`. They are in the
global catalogue, so your account reaches them without cataloguing anything of
its own.

**Most exist to support SD itself and are not an interface for applications.**
They are listed in full because a name in a stack trace, a `map` listing or an
error message has to be findable, and because a handful are genuinely useful.
Which is which is said plainly below rather than left for you to discover.

> These are not a versioned interface. A routine here can change between
> releases in ways a documented function would not. If you call one, test what
> it returns rather than assuming its shape holds.

## Some are functions and some are subroutines

Get this right first, because calling one the wrong way does not produce a
helpful error.

A **subroutine** is reached with `call`, and its results come back in its
arguments. The name is written with its `!` and brackets, not as a quoted string
with the arguments after commas (that form does not compile: *"Expected direct
or indirect subroutine reference"*):

```
call !errtext(text, 3035)
```

A **function** is declared with `deffun` and returns a value:

```
deffun ftype(path) calling "!FTYPE"
kind = ftype(@path)
```

`SUBR()` calls a subroutine from an I-type expression in a dictionary.

Nothing here is typed at the TCL prompt. These are called from SD BASIC.

## The ones an application may reasonably use

### Paths and names

| | |
|---|---|
| `!ABSPATH` | `abspath((dir), rel)` — resolve *rel* against *dir* and return an absolute path. A null *dir* means `@path`; a *rel* that starts with `/` comes back as it is, and one that starts with `@sdsys` has that replaced by the SDSYS directory |
| `!PATHTKN` | `pathtkn((path))` — replace a leading `@sdsys`, `@usrdir`, `@tmp` or `@home` in *path* with the installation directory, the accounts folder, `config('TEMPDIR')` or the `HOME` environment variable; the token may be in either case. Anything else comes back unchanged. The token is recognised by its first characters alone, so `@homex/y` becomes the home directory followed by `x/y`. It does not split anything |
| `!FTYPE` | `ftype(path)` — what kind of file is at *path* |
| `!VALID_OS_NAME` | `valid_os_name(name)` — is *name* usable as an operating system account name |
| `!VALID_OS_PATH` | `valid_os_path(path)` — is *path* a well-formed host path |
| `!VALID_SHELL_CMD` | `valid_shell_cmd(cmd)` — is *cmd* acceptable to hand to a shell |

The three validators are the ones SD's own administrative verbs use before
handing a value to the shell or the operating system. If you are building a
command line or an account name from input, call them rather than
reimplementing the rules.

### Sessions and users

| | |
|---|---|
| `!USERNAME` | `username(userno)` — the login name of the session with that user number. Null if the number is below 1 or no session has it |
| `!USERNO` | `userno((name))` — the user numbers of the sessions signed on under *name*; upper or lower case makes no difference. **It can return more than one**: when the name is signed on in two sessions the result is two numbers separated by a field mark, and a program that assumes one number gets the pair. Null if no session has that name, or if *name* is null. A session's own number is `@userno` |

### Errors, formatting and sorting

| | |
|---|---|
| `!ERRTEXT` | `call !errtext(expansion, errno)` — the message text for a status code |
| `!FORMAT` | `call !format(in.rec, out.rec, default.file, indent.step, case.option, errors)` — the source formatter behind the `format` verb |
| `!SORT` | `call !sort(in.list, out.list, mode)` — sort a list; *in.list* may also be a select list variable, whose ids are sorted (and used up) |
| `!VOCREC` | `call !vocrec(rec, (id))` — read the VOC record *id* into *rec*, following R records. The id is tried as given, then in lower case, then in upper case. An R record names a file (field 2) and a record in it (field 3): `!VOCREC` opens that file, reads that record, and repeats while the record it has read is another R record. `status()` is 0 when a record was read. When the VOC has no such record, or an R record leads to a file that will not open or a record that is not there, *rec* comes back null and `status()` was 3007, "No VOC record", in each case tried. It builds and checks nothing |

`!ERRTEXT` is the useful one. A program that traps a status code and wants to
log something a person can read should call it rather than keep its own table
of numbers.

### Selection lists on screen

| | |
|---|---|
| `!PICK` | `call !pick(item, top.line, item.list, title, pick.pos)` — a full-width scrolling chooser |
| `!PICKLIST` | `call !picklist(value, list, return.col, index.col)` — a box drawn over the screen that lists the rows of *list* and returns one of them |

Both need a real terminal. Neither does anything useful in a phantom, a
scheduled job or an API session.

**The routine is `!PICKLIST`.** An earlier version of this page called it
`!PICKLST`, which is the name of the program that holds it; there is no
catalogue entry of that name, and `call !picklst(...)` stops at run time with
*"Unable to load '!picklst' object code"*.

`!PICKLIST` takes *list* as a field-mark delimited list of rows; a row of
several columns has its columns separated by value marks, and the first row
must show how many columns there are. The arrow keys, Home, End, Page Up and
Page Down move the highlight and Return ends the call with *value* set to the
column *return.col* (counted from 1) of the highlighted row. Ctrl-X ends it
with *value* null. If *index.col* is not zero the characters typed (up to ten)
search that column, which must be sorted, for a shortcut to a row; zero turns
that off. Up to thirteen rows show at a time. Two cases end the call with
*value* null and no key pressed: a list so wide that its box would not fit in
the terminal's width less four columns (tried with a 500-character row). An
empty list ends it the same way after it has shown *"No items to display"* and
waited for a key (read from the program, not run). **The box is not
erased afterwards:** a program that needs the screen under it back must redraw
it.

### Editing a data collection

| | |
|---|---|
| `!CED` | `call !ced(collection)` — the data collection editor, on a collection held in a variable; see [ced](27b-sd-tcl-ced.html) |

It needs a real terminal too, and returns at once without one.

### Printing and program information

| | |
|---|---|
| `!PCL` | `pcl(key, arg1 … arg6)` — printer control sequences, variable arguments |
| `!PROG_INFO` | `call !prog_info(obj_dir_path, obj_name, prog_name, prog_obj_sz, prog_comp_time, prog_comp_date, error_msg)` |

### The command parser

| | |
|---|---|
| `!PARSER` | `call !parser(key, type, string, keyword, voc.rec, quote.char)` — variable arguments |

`!PARSER` is the tokeniser SD's own verbs use to read their arguments, and the
keys it takes are defined in `SYSCOM/PARSER.H`. It is the most useful of the
internal routines and the one most likely to change.

## The internal ones

These exist for SD's own use. They are catalogued because SD's programs are
ordinary compiled programs and reach them the same way anything else does.

**Calling them from an application is not supported.** Several refuse a session
that has not unlocked `admin`, and some change the state of the computer.

| | |
|---|---|
| `!ATVAR` `!SETVAR` | read and set the `@` variables |
| `!GETPU` `!SETPU` | read and set one characteristic of a print unit: `call !getpu(key, unit, value, status)` and `call !setpu(key, unit, new.value, status)`, which are the `getpu()` function and the `setpu` statement wrapped as subroutines (SED uses `!SETPU` to set a print unit's banner and location). *key* is the print unit key (width 2, length 3, mode 1 and so on: see the keys in *SD Basic - Printing*); *unit* runs from -1 to 256, and any other number gives *status* 1007, "Bad print unit". They hold nothing per user. An application can use the statement and the function directly |
| `!CRED_SET` `!CRED_VERIFY` | write and check a credential in the credential store |
| `!SD_GET_SALT` `!SD_KEY_FROM_PW` | the key derivation behind that credential |
| `!SOLO_STORE_PW` | keep the copy of the account password that lets `sd-solo <command>` sign in without typing |
| `!PW_COMPLEX` | SD's password rule — 8 characters with a lower-case letter, an upper-case letter, a digit and a symbol |
| `!VOC_GUARD` | the `admin` test for a direct change to the VOC, and the refusal of a write to `gcat`, `global.bp.out` and the deny list without the global password |
| `!EUID_SET` `!EUID_RESTORE` | **do nothing on Solo** — the multiuser product's `setuid`/`setgid` calls |
| `!IS_USER` `!IS_GROUP` `!IS_GRP_MEMBER` `!IS_SD_USER` | Linux account and group questions |
| `!PEER_LOCAL` | whether a connection came from this computer |
| `!SDCLIENT` | the server side of the client API |

**There is no `!ELEVATE`, `!PS_SCRIPT`, `!DELETE_USER` or `!OS_GROUP` here**,
and no `sd-elevate` to reach: SD never asks for `root`.

## What is not here

Documentation for other MultiValue systems describes `!`-prefixed routines SD
Core for Windows does not have. Three a reader may come looking for:

| | |
|---|---|
| `!OCONV` `!ICONV` | not catalogued. `oconv()` and `iconv()` are ordinary BASIC functions and do the same work |
| `!SCREEN` | not catalogued, and there is no equivalent |

**To settle the question for any name, use `map` in the account.** It lists what
the catalogue holds. A name that is not there is not available, and calling it
fails at run time rather than at compile time.

## See also

[SD Dictionaries - Conversions and Formatting](34-sd-dicts-conversions.html) ·
[SD TCL - Programs and the Catalogue](24-sd-tcl-programs-and-the-catalogue.html) ·
[SD Basic - Program Structure](01-sd-basic-program-structure.html).
