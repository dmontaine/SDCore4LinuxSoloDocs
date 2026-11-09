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
[sd-solo]
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

**The parameters SD no longer has are removed from a kept `sd.conf` for you.**
In 1.2-0 thirteen names went, because they did nothing or imitated another
product: `APILOGIN`, `CODEPAGE`, `CREATUSR`, `EXCLREM`, `FILERULE`, `FIXUSERS`,
`GDI`, `INVCASE`, `JNLDIR`, `JNLMODE`, `PORTMAP`, `RINGWAIT` and `TXCHAR`. A line
naming one would now stop SD, so an upgrade, and a new install that reloads a kept
`sd.conf`, take each such line out first and say which. A configuration file you
copy in by hand from an older installation needs the same done to it.

Range failures behave the same way. `GRPSIZE`, `INTPREC`, `LPTRHIGH`, `LPTRWIDE`,
`MAXCALL`, `RECCACHE`, `SORTMRG` and `MAXIDLEN` are bounds-checked once the file
has been read, and a value outside its range stops start-up with a message naming
the parameter.

## Reading the settings

The `config` verb reports what is in force — after `admin`, apart from `config
gpl` and `config contrib`:

```
:config
Virtual Machine Version Number LS1.1-3
CLEANUP   300 seconds
CMDSTACK  99
DEADLOCK  0
DHCACHE   10
DUMPDIR   
ERRLOG    50 kb
FLTDIFF   0.00000000002910
FSYNC     0
GRPDIR    /home/you/SDCoreSolo/group_accounts
GRPSIZE   2
INTPREC   11
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
contributors — those two need no `admin`.

## Changing a parameter

Editing `sd.conf` and restarting SD is the durable route, and for most parameters
it is the only one.

**Some parameters can also be changed for the current session** (after `admin`):

```
config sortmem 8192
```

That writes to the session's own copy of the settings, taken from shared memory
when the session started. It does not reach `sd.conf`, it does not affect any
other session, and it is gone when the session ends.

The parameters that can be changed this way are `DUMPDIR`,
`FLTDIFF`, `FSYNC`, `GRPSIZE`, `INTPREC`, `LPTRHIGH`,
`LPTRWIDE`, `MAXCALL`, `MUSTLOCK`, `OBJECTS`, `OBJMEM`, `RECCACHE`,
`SAFEDIR`, `SDCLIENT`, `SH`, `SH1`, `SORTMEM`, `SORTMRG`, `SORTWORK`, `SPOOLER`,
`TEMPDIR`, `TERMINFO` and `YEARBASE`.

Everything else takes effect only when SD is next started (`sd-solo -stop`, then
`sd-solo -start`). That includes every limit which sizes the shared memory segment.

## Sessions and limits

| Parameter | Default | Effect |
|---|---|---|
| `NUMUSERS` | 20 | Maximum concurrent sessions. Sizes the user table in shared memory |
| `NUMFILES` | 80 | Maximum open files across all sessions |
| `NUMLOCKS` | 100 | Maximum record locks across all sessions |
| `MAXCALL` | 10000 | Maximum subroutine call depth. Range 10 to 1000000 |
| `CMDSTACK` | 99 | Depth of the command stack |
| `NETWAIT` | 5 | Seconds to wait for a connection to a server defined for SDNet (`set.server`, `set.netlogin`, a file opened through an `N` pointer). 1 to 3600; zero is refused |
| `NETREPLY` | 30 | Seconds to wait for each reply of such a server. 1 to 3600; zero is refused. `config('NETWAIT')` and `config('NETREPLY')` read them. See the SDNet page of the User set |
| `FDS` | unset | Limit on file descriptors. No limit when unset |

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

## Directories

| Parameter | Default on a new install | Effect |
|---|---|---|
| `SDSYS` | the installation directory itself, `~/SDCoreSolo` | The system directory. **You do not set it**: it is worked out from where `bin/sd` is, and SD does not start if the global catalogue is not found beneath it |
| `USRDIR` | `<home>/user_accounts` | Where the account lives |
| `GRPDIR` | `<home>/group_accounts` | Where group accounts would live. Solo has none, and the directory is empty |
| `DUMPDIR` | unset | Where process dumps are written |
| `TEMPDIR` | unset | Temporary files. Falls back to the system default when unset |
| `SORTWORK` | unset | Work files for a sort that does not fit in memory. Falls back to the system default when unset |
| `TERMINFO` | empty | An additional terminfo directory. The shipped definitions are found without it |

**A process dump carries the whole variable state of the session that wrote it**,
so it may hold a password typed into a program. Solo's tree is private to you, and
if `DUMPDIR` is left unset the dumps go where SD's system default puts them; if
you set it, choose a directory only you can read.

## The API

| Parameter | Default | Effect |
|---|---|---|
| `SDCLIENT` | 0 | Restricts what an API session may do: `1` refuses `CALL` and `EXECUTE`, `2` refuses any subroutine not compiled as callable from a client |

**There is no `APIPORT` and no `NETDIRS` in `sd.conf`.** Whether SD listens for
the API at all, and on which port, is your systemd user unit
`sd-solo-api.socket`, activated independently of any running `sd-solo` process; it is
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
| `OBJECTS` | 0 (no limit) | Compiled programs held in memory at once |
| `OBJMEM` | 0 (no limit) | Memory those programs may occupy, in KB |

## The shell

| Parameter | Default | Effect |
|---|---|---|
| `SH` | empty | The shell a bare `sh` starts |
| `SH1` | empty | The shell `sh command` uses |

**On a Solo tree both read as empty in `config`**, and `sh command` runs the
command through `bash -c` from the account's directory — see
[Operating system access](06b-operating-system-access.html). **A bare `sh` starts
an interactive shell** (see the `SH` line above). **Both run with your own
Linux permissions, once the `sh` switch is on; as delivered it is off and `sh`
is refused** (`set.os.access sh on`).

## Parameters that were removed

Until 1.2-0 seven parameters were parsed and did nothing (`JNLMODE`, `JNLDIR`,
`CREATUSR`, `CODEPAGE`, `EXCLREM`, `RINGWAIT`, `TXCHAR`), and six more imitated
another product or chose between ways of working that SD no longer has
(`APILOGIN`, `FILERULE`, `FIXUSERS`, `GDI`, `INVCASE`, `PORTMAP`). All thirteen
are gone; see [A name SD does not recognise stops it starting](#a-name-sd-does-not-recognise-stops-it-starting)
above for what happens to a kept `sd.conf` that still names one. The `PATH:` form
of a file name went with `FILERULE`: write an F-pointer instead.

`NETFILES` is a live parameter, though an earlier edition of this page
said it did nothing. A `server;file` VOC reference is refused whatever it holds, but **bit 2
(`NETFILES=2`) opens API request 29**, which opens an `account;file` file for a
client that has signed in. It is 0 as delivered. With it set, the request is
still refused unless the account's NET gate is on (`set.net.access on`), and
under `SDCLIENT`; see [API access](09-api-access.html). Since 21 October 2026 a
pointer in the account's VOC that leads out of the account is refused too, unless
the account has the `os` switch on. Before 20 October 2026 it
was refused for nothing but the bit, so a site that had set `NETFILES=2` had
let an API login open a file named in the VOC of any directory that holds a
`voc`: take the setting out, or turn the NET gate on knowing what it allows.

There is no licence parameter and no licence verb. SD Core is GPL v3 and is not
licensed per site or per user; `config gpl` displays the licence text.
