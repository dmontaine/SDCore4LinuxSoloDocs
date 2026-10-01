Title: Configuration
Subtitle: The sd.conf file, the two ways to read a setting, and every parameter SD accepts.

SD reads one configuration file at start-up. It sizes the shared memory segment,
sets the limits every session inherits, and names the directories SD writes to.

SD folds case, so a command may be typed in either case. Commands are shown here
in lower case.

> The parameter list and categories on this page are read from `config.c` itself
> (`tools/confmap.py`), and the shipped file and the `config` output were checked
> on a Solo tree; the sample output below is a real capture with the tree's own
> paths shortened.

## The file

```
~/SDCoreSolo/sd.conf
```

**SD finds it from where its programs are**: the directory that holds `bin/` (and
the marker file `.sdcoresolo`). The server and the client both read the
`SD_CONFIG` environment variable first, if you set it, and use that file instead.

It is plain text in one section, and **the shipped file names no path**:

```
[sd]
GRPSIZE=2
NUMUSERS=20
SORTMEM=4096
ERRLOG=50
```

Lines beginning `#` are comments. The shipped file is heavily commented and those
comments record why each value was chosen. Read them before changing anything.
**An upgrade keeps your `sd.conf`** — see
[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html).

### A name SD does not recognise stops it starting

An unrecognised parameter is not ignored. The parser abandons the file with
`Unrecognised configuration parameter`, and SD does not start.

That is why obsolete names are still parsed rather than deleted. `CREATUSR` has
done nothing since 14 August 2026, but a configuration file copied from another
installation still carries it, and removing the branch would turn a tidy-up into a
failure to start.

Range failures behave the same way. `GRPSIZE`, `INTPREC`, `LPTRHIGH`, `LPTRWIDE`,
`MAXCALL`, `RECCACHE`, `SORTMRG` and `MAXIDLEN` are bounds-checked once the file
has been read, and a value outside its range stops start-up with a message naming
the parameter.

## Reading the settings

The `config` verb reports what is in force — after `ADMIN`, apart from `config
gpl` and `config contrib`:

```
:config
Virtual Machine Version Number LS1.1-2
CMDSTACK  99
CREATUSR  1
DEADLOCK  0
DUMPDIR   
ERRLOG    50 kb
EXCLREM   0
FILERULE  0
FLTDIFF   0.00000000002910
FSYNC     0
GDI       0
GRPDIR    /home/you/SDCoreSolo/group_accounts
GRPSIZE   2
INTPREC   13
...
USRDIR    /home/you/SDCoreSolo/user_accounts
YEARBASE  1930
```

**`config` pages its output** — press Return for more, or `Q`.

The `config()` function reads one parameter from a program:

```
group.size = config('GRPSIZE')
```

The name is truncated to eight characters before it is matched; no parameter name
is longer than eight, so that only shows up if you pass something which is not a
parameter.

Two further forms exist. `config lptr` reports the settings of the default
printer, and `config gpl` and `config contrib` display the licence and the list of
contributors — those two need no `ADMIN`.

## Changing a parameter

Editing `sd.conf` and restarting SD is the durable route, and for most parameters
it is the only one.

**Some parameters can also be changed for the current session** (after `ADMIN`):

```
config sortmem 8192
```

That writes to the session's own copy of the settings, taken from shared memory
when the session started. It does not reach `sd.conf`, it does not affect any
other session, and it is gone when the session ends.

The parameters that can be changed this way are `CODEPAGE`, `DUMPDIR`, `EXCLREM`,
`FILERULE`, `FLTDIFF`, `FSYNC`, `GDI`, `GRPSIZE`, `INTPREC`, `LPTRHIGH`,
`LPTRWIDE`, `MAXCALL`, `MUSTLOCK`, `OBJECTS`, `OBJMEM`, `RECCACHE`, `RINGWAIT`,
`SAFEDIR`, `SDCLIENT`, `SH`, `SH1`, `SORTMEM`, `SORTMRG`, `SORTWORK`, `SPOOLER`,
`TEMPDIR`, `TERMINFO` and `YEARBASE`.

Everything else takes effect only when SD is next started (`sd -stop`, then
`sd -start`). That includes every limit which sizes the shared memory segment.

## Sessions and limits

| Parameter | Default | Effect |
|---|---|---|
| `NUMUSERS` | 20 | Maximum concurrent sessions. Sizes the user table in shared memory |
| `NUMFILES` | 80 | Maximum open files across all sessions |
| `NUMLOCKS` | 100 | Maximum record locks across all sessions |
| `MAXCALL` | 10000 | Maximum subroutine call depth. Range 10 to 1000000 |
| `CMDSTACK` | 99 | Depth of the command stack |
| `FDS` | unset | Limit on file descriptors. No limit when unset |
| `FIXUSERS` | unset | `base,range` — user numbers reserved for sessions that ask for a specific number |
| `PORTMAP` | unset | `base_port,base_user,range` — gives a session a fixed user number derived from the port its connection arrived on. Refused if the range overlaps `FIXUSERS` |

## Files and locking

| Parameter | Default | Effect |
|---|---|---|
| `GRPSIZE` | 2 | Default group size for a new dynamic file, in 1 KB units. Read by `create.file` and `configure.file` when no group size is given |
| `MAXIDLEN` | 63 | Maximum record id length. 63 is also the lower bound |
| `MUSTLOCK` | 0 | When 1, a `write` or `delete` requires the record to be locked first |
| `DEADLOCK` | 0 | When 1, SD traps deadlocks |
| `SAFEDIR` | 0 | When 1, directory files are updated by write-and-rename rather than in place |
| `RECCACHE` | 0 | Records cached per file. Range 0 to 32 |
| `FSYNC` | 0 | Bit flags controlling when SD forces data to disk |
| `FILERULE` | 0 | Bit flags deciding which special VOC file references are honoured. Bit 4 allows a `PATH:` reference to name a pathname directly |

## Directories

| Parameter | Default on a new install | Effect |
|---|---|---|
| `SDSYS` | the installation directory itself, `~/SDCoreSolo` | The system directory. **You do not set it**: it is worked out from where `bin/sd` is, and SD does not start if the global catalogue is not found beneath it |
| `USRDIR` | `<home>/user_accounts` | Where the account lives |
| `GRPDIR` | `<home>/group_accounts` | Where group accounts would live. Solo has none, and the directory is empty |
| `DUMPDIR` | unset | Where process dumps are written |
| `TEMPDIR` | unset | Temporary files. Falls back to the system default when unset |
| `SORTWORK` | unset | Work files for a sort that does not fit in memory. Falls back to the system default when unset |
| `JNLDIR` | empty | Journal directory |
| `TERMINFO` | empty | An additional terminfo directory. The shipped definitions are found without it |

**A process dump carries the whole variable state of the session that wrote it**,
so it may hold a password typed into a program. Solo's tree is private to you, and
if `DUMPDIR` is left unset the dumps go where SD's system default puts them; if
you set it, choose a directory only you can read.

## The API

| Parameter | Default | Effect |
|---|---|---|
| `APILOGIN` | 1 | Whether the API requires authentication. `0` is the weaker setting, not the safer one |
| `SDCLIENT` | 0 | Restricts what an API session may do: `1` refuses `CALL` and `EXECUTE`, `2` refuses any subroutine not compiled as callable from a client |

**There is no `APIPORT` and no `NETDIRS` in `sd.conf`.** Whether SD listens for
the API at all, and on which port, is your systemd user unit
`sd-solo-api.socket`, activated independently of any running `sd` process; it is
changed by running `tools/solo-service.sh install` again — see
[API access](09-api-access.html).

`SDCLIENT` is not reported by the `config` verb and has no entry in the shipped
`sd.conf`. It defaults to 0, which permits everything.

## Printing

| Parameter | Default | Effect |
|---|---|---|
| `LPTRWIDE` | 80 | Default page width for the printer. Range 10 to 1000 |
| `LPTRHIGH` | 66 | Default page depth for the printer. Range 10 to 32767 |
| `SPOOLER` | empty | The spooler to hand print jobs to |
| `GDI` | 0 | Selects the Windows printing interface used by default. Inert on Linux |

## Sorting

| Parameter | Default | Effect |
|---|---|---|
| `SORTMEM` | 4096 kb | Above this much data a sort works on disk instead of in memory |
| `SORTMRG` | 4 | Files merged at once in a disk sort. Range 2 to 10 |

## Numbers and dates

| Parameter | Default | Effect |
|---|---|---|
| `INTPREC` | 13 | Digits of precision in integer arithmetic. Range 0 to 14 |
| `FLTDIFF` | 0.00000000002910 | Two floating point numbers closer together than this compare equal |
| `YEARBASE` | 1930 | The century a two-digit year is read into |

## Diagnostics

| Parameter | Default | Effect |
|---|---|---|
| `ERRLOG` | 50 kb | Size of the error log. A non-zero value below 10 KB is raised to 10 KB, and the oldest entries are discarded when it fills |
| `PDUMP` | 0 | Bit flags controlling when a process dump is written |
| `DEBUG` | unset | Bit flags enabling debugging features |
| `STARTUP` | empty | A command run when SD starts |
| `JNLMODE` | 0 | Journalling mode |
| `OBJECTS` | 0 (no limit) | Compiled programs held in memory at once |
| `OBJMEM` | 0 (no limit) | Memory those programs may occupy, in KB |

## The shell

| Parameter | Default | Effect |
|---|---|---|
| `SH` | empty | The shell a bare `sh` starts |
| `SH1` | empty | The shell `sh command` uses |

**On a Solo tree both read as empty in `config`**, and `sh command` runs the
command through `bash -c` from the account's directory — see
[Operating system access](06b-operating-system-access.html). **A bare `sh` is
refused** rather than starting an interactive shell. **Both run with your own
Linux permissions, unconditionally — SD keeps no second wall here.**

## Parameters that are accepted and do nothing

These are parsed so that an existing `sd.conf` still loads. None of them changes
SD's behaviour, and none should be offered to a site as a control.

| Parameter | Why it is inert |
|---|---|
| `NETFILES` | SDNet was removed. A `server;file` VOC reference is refused, and setting it opens nothing |
| `CREATUSR` | There are no accounts to create, and the value is discarded as it is read. Solo's `config` still prints a `CREATUSR` line |
| `CODEPAGE` | Stored, readable and settable. Nothing acts on it |
| `EXCLREM` | Stored, readable and settable. It described exclusive access to a remote file, and there are no remote files |
| `RINGWAIT` | Stored, readable and settable. Nothing acts on it |
| `TXCHAR` | Stored, and not readable — there is no `config('TXCHAR')` |

`FILERULE` is **not** in this list, and is easy to mistake for it. Its remote bits
died with SDNet, but bit 4 is live and decides whether a `PATH:` VOC reference may
name a pathname directly.

There is no licence parameter and no licence verb. SD Core is GPL v3 and is not
licensed per site or per user; `config gpl` displays the licence text.
