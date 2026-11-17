Title: Configuration
Subtitle: The sd.conf file, the two ways to read a setting, and every parameter SD accepts.

SD reads one configuration file at start-up. It sizes the shared memory segment,
sets the limits every session inherits, and names the directories SD writes to.

SD folds case, so a command may be typed in either case. Commands are shown here
in lower case.

> The sample output below is illustrative; the values on your system will
> differ where your configuration does.

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
SCREENTHEME=paper
GRPSIZE=4
NUMUSERS=20
SORTMEM=4096
ERRLOG=50
```

Lines beginning `#` are comments. The shipped file is heavily commented and those
comments record why each value was chosen. Read them before changing anything.
**An upgrade keeps your `sd.conf`** — see
[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html).

### A name SD does not recognize stops it starting

An unrecognized parameter is not ignored. The parser abandons the file with
`Unrecognized configuration parameter`, and SD does not start.

**The parameters SD no longer has are removed from a kept `sd.conf` for you.**
In 2.0-0 thirteen names went, because they did nothing or imitated another
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
Virtual Machine Version Number LS2.0-0
CLEANUP   300 seconds
CMDSTACK  99
DEADLOCK  0
DHCACHE   10
DUMPDIR   
ERRLOG    50 kb
FLTDIFF   0.00000000002910
FSYNC     0
GRPDIR    /home/you/SDCoreSolo/group_accounts
GRPSIZE   4
INTPREC   11
...
USRDIR    /home/you/SDCoreSolo/user_accounts
YEARBASE  1930
```

SD accepts **58** parameters (`APIPORT` does not exist here — see *The API*, below). The verb prints all of them except `BACKUPDIR` and `SDSYS`, and `DEBUG` and `FDS` only when they are set.

**`config` pages its output** — press Return for more, or `Q`.

The `config()` function reads one parameter from a program:

```
group.size = config('GRPSIZE')
```

**55 parameters are readable this way.** The two that are not are `BACKUPDIR` and
`SDSYS`. A name of more than sixteen characters is not found (the function answers
an empty string and `status()` is not 0); no parameter name is that long, so that
only shows up if you pass something which is not a parameter.

Two further forms exist. `config lptr` reports the settings of the default
printer, and `config gpl` and `config contrib` display the license and the list of
contributors — those two need no `admin`.

## Changing a parameter

Editing `sd.conf` and restarting SD is the durable route, and for most parameters
it is the only one. Fourteen tuning parameters can also be changed from SD itself, with
`set.config` or the `E` key of `tool.config` (see *Changing a parameter from SD*, below).

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

**In Solo every change to `sd.conf` waits for the next start.** SD reads the file
once, when it starts, and keeps the settings in shared memory; a new session copies
them from there and does not read the file itself. (The multiuser SD Core has each
session read `sd.conf` when it starts, so there an edit reaches the sessions
started after it.) So `tool.config` shows every parameter as `restart`, and
`set.config` says the change takes effect when SD is next stopped and started: stop
SD and start it again before you count on the new value.

## The parameters on one screen: `tool.config`

```
tool.config
```

A full-screen page for the administrator (`admin` first; a window of 140 columns by 43 lines or
more). **It looks, and it can change 14 of the parameters**; it writes no file itself. A change is
made by the verb `set.config` (below), and only after you have typed a value, the page has checked
it against the range SD enforces and a box has shown the exact command and been answered Run. The
other parameters are not changed from SD; editing the file and restarting SD stays the way to change
them.

The page has three views. **P, the parameters**: one row for every parameter SD knows, from the table
`config.table` in the installation directory, which names every parameter SD reads. The columns are
the value this SD is running with (what `config()` reports), the value `sd.conf` says now (the file is
read again, a parameter written twice counted the way SD counts it, the flag parameters added up),
the default (what SD uses when the file does not say), when a change takes effect and whether anything
acts on it:

| Takes effect | Meaning |
|---|---|
| `restart` | the parameter takes effect when SD is next started: SD reads the file once, when it starts, and a session takes its settings from shared memory (every parameter but `BACKUPDIR`) |
| `now` | a program reads `sd.conf` each time it uses the parameter (`BACKUPDIR`), so a change needs no restart |

| Status | Meaning |
|---|---|
| `live` | something acts on it |
| `file` | read from `sd.conf` by a program each time it is used, not by the kernel (`BACKUPDIR`) |
| `unknown` | a name in the file that SD does not know |

A row marked `*` is one where the file and the running SD disagree. The box at the foot says for that
parameter that SD keeps the old value until it is restarted. **A name in the file
that SD does not know is listed at the end as `unknown`, and the top of the page names it**: SD does
not start with it (the section *A name SD does not recognize stops it starting* above). `D` shows only
the rows where the file and SD disagree, and `S` only the parameters the file sets. The last column,
`Change`, says `e` for a parameter the page can change (press `E`) and `by hand` for every other.

`SDSYS`, `BACKUPDIR` and `NETDIRS` cannot be read with the `config()` function (a name longer than sixteen characters, or one the function does not have); the page says `(not readable)` for what SD is running with and shows the file's value. `DEBUG` is not compared: the function reports it in another form than the file writes it.

**B, the backup directory**: the directory saved with `set.backup.directory` and whether it is there;
**R, the report**: the operating system part of `settings.report`, made when the view is opened and
on F5. It holds no key and no password hash. `F5` reads the file, the backup directory and the report
again; `E` changes the parameter under the cursor, or says why it cannot be changed here; `F1`
explains the page; `Ctrl-Q` leaves. Opening the page writes the audit line
`tool start app=tool.config`; a change writes the line of `set.config` (below), and nothing else the
page does is written.

### Changing a parameter from SD: `set.config` and the `E` key

```
set.config cleanup 600
```

The administrator's verb (`admin` first). It changes **one** line of the `sd.conf` this SD reads (the
file `SD_CONFIG` names if it is set, else `~/SDCoreSolo/sd.conf`): the first word is a parameter name
(letters and digits; SD makes it capitals, as the file spells it) and the second a whole number. It
writes the file itself, since the file is yours. It accepts only the 14 parameters below, and only a
value inside the range SD itself enforces for that parameter when SD starts (so it cannot accept what
SD would refuse). It adds or replaces the line **inside the `[sd]` section**, the only place SD reads;
a second line for the same parameter in that section is dropped, because SD keeps the last one; every
other line is kept exactly as it was. The new file is written beside the old one and renamed over it
with the old file's mode, and **the file as it was is kept as `sd.conf.bak`** (one generation).
Success is the file read back: exactly one line for the parameter in the `[sd]` section, saying the
value. A refusal says why and writes nothing.

**One audit line** for each change asked for and checked: `set.config key=cleanup value=600 status=ok`,
or `status=refused`.

**On the page.** `E` on a row whose `Change` column says `e` asks for the new value (the question
shows what SD runs with and what the file says), checks that it is a whole number inside the range
before anything else happens, and then shows the exact command in the toolkit's box with a plain
sentence about when it takes effect. Cancel, which is what Enter chooses, runs nothing. `E` on any
other row writes nothing: it shows why the parameter is not changed from SD, and the exact line to
add by hand in the `[sd]` section.

**The 14 parameters** are the tuning ones: each is range-checked by SD and each only sizes or times
something SD does; none is a door, a path, a command, a log or a security setting:

| Parameter | Range |
|---|---|
| `CLEANUP` | 30 to 3600 |
| `DHCACHE` | 0 to 1024 |
| `GRPSIZE` | 1 to 8 |
| `LGNWAIT` | 0 to 32767 |
| `LPTRHIGH` | 10 to 32767 |
| `LPTRWIDE` | 10 to 1000 |
| `MAXCALL` | 10 to 1000000 |
| `MAXRLOCK` | 0 to 100000 |
| `NETREPLY` | 1 to 3600 |
| `NETWAIT` | 1 to 3600 |
| `NUMLOCKS` | 1 to 32767 |
| `RECCACHE` | 0 to 32 |
| `SORTMRG` | 2 to 10 |
| `USPLIT` | 0 to 1 |

**The other parameters are not changed from SD**, and `set.config` refuses them by name (a name it does not
know at all it refuses with a different sentence, because an unknown name in `sd.conf` stops SD from
starting). They are the doors and what they allow (`NETDIRS`, `NETFILES`, `SDCLIENT`), every
path and command, the log, dump and diagnostic settings, the security throttle `PWDELAY`, the sizes
and combinations that SD does not check or checks against each other, and the ones that change
what existing programs compute. Change them by hand, as before. The page
shows the same sentence in its box:

| Parameter | Why it is not changed from SD |
|---|---|
| `BACKUPDIR` | A directory: set.backup.directory saves it, with its own checks of the path. |
| `CMDSTACK` | Sizes a table in shared memory and SD checks no range: a mistake shows only at the next start. |
| `DEADLOCK` | A diagnostic: 2 makes SD write a file of the locks involved; set it by hand while you hunt one. |
| `DEBUG` | Debugging flags of the kernel: they change what SD writes and exposes. |
| `DUMPDIR` | A directory: process dumps are written there and can hold what a session had in memory. |
| `ERRLOG` | Sizes the error log; SD checks no range, and a small size discards its oldest entries sooner. |
| `FDS` | Limits the file descriptors SD uses; SD checks no range: a mistake shows at the next start. |
| `FLTDIFF` | Changes how every program compares floating point numbers. |
| `FSYNC` | Trades the safety of data against speed; flag bits that SD does not check. |
| `GRPDIR` | A directory: where group accounts would be made. Solo has none. |
| `INTPREC` | Changes the results of arithmetic in programs that already exist. |
| `MAXIDLEN` | Structural: the longest record id of the whole system, which reaches every file and program. |
| `MUSTLOCK` | Makes a write fail unless the record is locked first, which can stop programs that exist. |
| `NETDIRS` | A door: it widens what a session without the `os` switch may reach beyond the account's own directory. |
| `NETFILES` | A door: bit 2 gates API request 29, a remote open of an account. |
| `NUMFILES` | Sizes the open file table in shared memory; SD checks no range. |
| `NUMUSERS` | Sizes the user table, which bounds who can sign on; SD checks no range. |
| `OBJECTS` | Limits the programs a session keeps in memory; SD checks no range. |
| `OBJMEM` | Limits the memory those programs may take; SD checks no range. |
| `PDUMP` | Flag bits that say when a process dump is written; a dump can hold what a session had in memory. |
| `PHANTOMS` | Must agree with USERPOOL: a wrong combination stops SD from starting. |
| `PWDELAY` | A security throttle: the pause after a failed network sign-on; 0 removes it. |
| `SAFEDIR` | Changes how SD replaces the files of a directory file, by writing a new one and renaming it. |
| `SCREENTHEME` | Text that SD acts on; SD checks no value. |
| `SDCLIENT` | A door: restricts what an API session may do, and a lower value loosens it. |
| `SDSYS` | The system directory: SD cannot start, or finds another system, if it is wrong. |
| `SH` | A command SD runs as an interactive shell. |
| `SH1` | A command SD runs for a shell command line. |
| `SORTMEM` | A size in kilobytes that SD does not check, so a mistake shows only when a sort runs. |
| `SORTWORK` | A directory: the work files of a large sort are written there. |
| `SPOOLER` | Text that names the spooler print jobs are handed to. |
| `STARTUP` | A command SD runs when it starts. |
| `TEMPDIR` | A directory: temporary files are written there. |
| `TERMINFO` | A directory: terminal definitions are read from there. |
| `USERPOOL` | The highest user number: PHANTOMS is checked against it; a mistake stops SD. |
| `USRDIR` | A directory: the account is made there. |
| `XFER` | A door: 1 opens file transfer over ssh once the account's own switch is on (`set.xfer.access on`). |
| `XFERFILE` | A transfer limit: raising it lets one file fill more of the disk. |
| `XFERIDLE` | A transfer limit: raising it keeps an unused slot open longer. |
| `XFERKIND` | A transfer filter: blank takes any plain-text file. |
| `XFERMAX` | A transfer limit: 0 lets a slot stay open for as long as its session does. |
| `XFERSLOT` | A transfer limit: raising it lets one slot fill more of the disk. |
| `XFERWAIT` | A transfer limit: raising it keeps a slot open longer before anyone connects. |
| `YEARBASE` | Changes how every program reads a two-digit year; SD checks no range. |

**What this does not do.** It does not restart SD: every change to `sd.conf` takes effect when SD is next stopped and started (`sd-solo -stop`, then `sd-solo -start`). It does not reach a file other than the `sd.conf` this SD reads. It cannot change a parameter that SD does not range-check, however useful that would be; such a parameter joins the list when SD itself checks it.

## Sessions and limits

| Parameter | Default | Effect |
|---|---|---|
| `NUMUSERS` | 20 | Maximum concurrent sessions. Sizes the user table in shared memory |
| `NUMFILES` | 80 | Maximum open files across all sessions |
| `NUMLOCKS` | 100 | Maximum record locks across all sessions. Range 1 to 32767 (the lock table's cells are 16 bits; a value outside it stops SD starting) |
| `MAXCALL` | 10000 | Maximum subroutine call depth. Range 10 to 1000000 |
| `CMDSTACK` | 99 | Depth of the command stack |
| `USERPOOL` | 1023 | The highest user number handed out; user numbers rotate through 1 to this, 0 always takes the lowest free one |
| `USPLIT` | 0 | `1`: interactive sessions take the lowest free user number and every other process (a phantom, an API session) the highest free one in 1 to `NUMUSERS`, so the two never meet. `PHANTOMS` and `USERPOOL` are ignored while it is on. `config('USPLIT')` reads it |
| `PHANTOMS` | unset | `base,range` — user numbers kept for phantoms started with `phantom user n`. Nobody else is given one: not a session, not an API session, not a phantom started without `user`. It must lie between 1 and the highest user number, or SD will not start. `config('PHANTOMS')` reads it as `base,range`, null when none are kept |
| `CLEANUP` | 300 | Seconds between the checks that look for sessions that ended without logging out (a killed terminal, a lost connection) and clear them from the user table. 30 to 3600. `config('CLEANUP')` reads it |
| `PWDELAY` | 3 | Seconds SD waits after a failed network sign-on before it answers; 0 removes the pause |
| `NETWAIT` | 5 | Seconds to wait for a connection to a server defined for SDNet (`set.server`, `set.netlogin`, a file opened through an `N` pointer): the TCP connection, then the TLS handshake, each. 1 to 3600; zero is refused |
| `NETREPLY` | 30 | Seconds to wait for each reply of such a server. 1 to 3600; zero is refused. `config('NETWAIT')` and `config('NETREPLY')` read them. See the SDNet page of the User set |
| `FDS` | unset | Limit on file descriptors. No limit when unset |

## Files and locking

| Parameter | Default | Effect |
|---|---|---|
| `GRPSIZE` | 4 | Default group size for a new dynamic file, in 1 KB units. Read by `create.file` and `configure.file` when no group size is given. **4 as shipped**. A Solo installed with an older release keeps the `sd.conf` it has, and so may keep 2 until you change it; a file keeps the size it was made with |
| `MAXIDLEN` | 63 | Maximum record id length. 63 is also the lower bound |
| `MUSTLOCK` | 0 | When 1, a `write` or `delete` requires the record to be locked first |
| `DEADLOCK` | 0 | When 1, SD traps deadlocks |
| `SAFEDIR` | 0 | When 1, directory files are updated by write-and-rename rather than in place |
| `RECCACHE` | 0 | Records cached per file. Range 0 to 32 |
| `FSYNC` | 0 | Bit flags controlling when SD forces data to disk |

## Directories

| Parameter | Default on a new install | Effect |
|---|---|---|
| `SDSYS` | the installation directory itself, `~/SDCoreSolo` | The system directory. **You do not set it**: it is worked out from where `bin/sd` is, and SD does not start if the global catalog is not found beneath it |
| `USRDIR` | `<home>/user_accounts` | Where the account lives |
| `GRPDIR` | `<home>/group_accounts` | Where group accounts would live. Solo has none, and the directory is empty |
| `DUMPDIR` | unset | Where process dumps are written |
| `TEMPDIR` | unset | Temporary files. Falls back to the system default when unset |
| `SORTWORK` | unset | Work files for a sort that does not fit in memory. Falls back to the system default when unset |
| `TERMINFO` | empty | An additional terminfo directory. The shipped definitions are found without it |
| `NETDIRS` | no line | Directories outside the account that a session without the `os` switch may still reach, separated by semicolons. See below |

**`NETDIRS` is a door, and it is shut as delivered.** With the `os` switch off
(`set.os.access`), a session opens no file by pathname outside the account's own
directory. `NETDIRS=/srv/sales;/srv/shared` names directories it may reach all
the same - a shared data directory, say - without turning the switch on, which
would open the whole computer. Once you list a directory, every session with the
switch off (a terminal, ssh, a phantom, the API) can read, change, create and
delete files in it and below it by pathname, as far as your Linux permissions
allow. `sh` and `os.execute` stay shut, and the VOC still changes only after
`admin`.

An entry is ignored, and named in the error log when SD starts, when it is not
an absolute path, is not an existing directory, is `/`, or holds or lies inside
the Solo folder itself. A name a program gives is resolved first, links and `..`
included, and then compared: a link inside a listed directory that leads
somewhere unlisted is refused, upper and lower case differ, and `/data/xkey` is
not inside `/data/x`. At most 16 entries are taken, and a value longer than 191
characters stops SD from starting. A change takes effect the next time SD is
started.

**A process dump carries the whole variable state of the session that wrote it**,
so it may hold a password typed into a program. Solo's tree is private to you, and
if `DUMPDIR` is left unset the dumps go where SD's system default puts them; if
you set it, choose a directory only you can read.

## File transfer over ssh

`receive` and `send` (see *Moving Files In and Out* in the User documentation).
Each takes effect when SD is next started, as every parameter not listed above. None of them is
one of the 14 parameters `set.config` changes: `XFER` is a door, and the limits are added by hand.

| Parameter | Default | Range | Effect |
|---|---|---|---|
| `XFER` | 0 | 0 to 1 | **The system's switch, off as delivered.** 1 lets the account move files with `receive` and `send` over Solo's ssh, once its own switch is on too (`set.xfer.access on`). It opens a door from the network into the account's transfer folders: see [Security](12-security.html) |
| `XFERFILE` | 10240 | 1 to 1048576 | The largest file one transfer takes, in kilobytes; a bigger file is refused and discarded |
| `XFERSLOT` | 102400 | 1 to 4194304 | The most one transfer takes in all its files, in kilobytes |
| `XFERWAIT` | 600 | 10 to 3600 | Seconds a transfer waits for the sftp program's first connection before it closes |
| `XFERIDLE` | 120 | 10 to 3600 | Seconds a transfer stays open after its last file has finished; a file still moving is never cut off |
| `XFERMAX` | 3600 | 0 to 86400 | Seconds a transfer may stay open at most; 0 removes the ceiling |
| `XFERKIND` | blank | | The endings a received file's name must have, separated by commas (`csv,txt`); blank takes any plain-text file |

## The API

| Parameter | Default | Effect |
|---|---|---|
| `SDCLIENT` | 0 | Restricts what an API session may do. Non-zero disables file access outright (no file can be opened, so no dictionary can be evaluated either) and refuses `SDIConv`, `SDOConv`, `SDIConvs` and `SDOConvs` a conversion code that reads a file (`T` with a file) or calls a program (`U`); any non-zero value also refuses `SDOpenSeq` and `SDExecute`; `2` additionally refuses any subroutine not compiled as callable from a client (a subroutine whose name begins with `$` must be so compiled at every setting, `0` included) |

**There is no `APIPORT` in `sd.conf`.** Whether SD listens for
the API at all, and on which port, is your systemd user unit
`sd-solo-api.socket`, activated independently of any running `sd-solo` process; it is
changed by running `tools/solo-service.sh install` again — see
[API access](09-api-access.html).

`SDCLIENT` has no entry in the shipped `sd.conf`. It defaults to 0, which permits
everything.

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

## The screen

| Parameter | Default | Effect |
|---|---|---|
| `SCREENTHEME` | unset (`mono`); `paper` in the shipped `sd.conf` | The color theme that a program using the `SCREEN` statement gets when it does not choose one: `paper`, `night`, `mono`, `classic` or `blue`. `paper` is black and dark blue text on light gray, with the menu bar and menus the other way round (light gray on black); `night` is white and yellow text on black, its menus black on light gray; `mono` uses only reverse video, underline and bold and reads on any terminal. The shipped `sd.conf` says `SCREENTHEME=paper`, so the toolkit pages, the IDE, SED and the debugger show paper on a color terminal on a fresh install. An upgrade adds `SCREENTHEME=paper` to a kept `sd.conf` that has no `SCREENTHEME` line, and leaves a line that is there alone. A terminal with no color (vt100, a phantom, a pipe) is `mono` whatever the line says, and so is a name that is not a theme, without an error. A program that asks for a theme gets that one, and reads the value with `config('SCREENTHEME')`. **Your own choice wins**: Ctrl-T or View > Theme on any full-screen program keeps the theme chosen, and every full-screen program starts in it from then on; `SCREENTHEME` is what you see until you have chosen |

## The shell

| Parameter | Default | Effect |
|---|---|---|
| `SH` | `/bin/bash -i`, unset in the file | The shell a bare `sh` starts |
| `SH1` | `/bin/bash -c`, unset in the file | The shell `sh command` uses |

Unset falls back to the compiled-in default:

```
SH        /bin/bash -i
SH1       /bin/bash -c
```

**Both run with your own Linux permissions, and only once the `sh` switch is
on**; as delivered it is off and `sh` is refused (`set.os.access sh on`; see
[Operating system access](06b-operating-system-access.html)). The setting says
which shell it is, the switch says whether it may be used.

## Parameters that were removed

Until 2.0-0 seven parameters were parsed and did nothing (`JNLMODE`, `JNLDIR`,
`CREATUSR`, `CODEPAGE`, `EXCLREM`, `RINGWAIT`, `TXCHAR`), and six more imitated
another product or chose between ways of working that SD no longer has
(`APILOGIN`, `FILERULE`, `FIXUSERS`, `GDI`, `INVCASE`, `PORTMAP`). All thirteen
are gone; see [A name SD does not recognize stops it starting](#a-name-sd-does-not-recognize-stops-it-starting)
above for what happens to a kept `sd.conf` that still names one. The `PATH:` form
of a file name went with `FILERULE`: write an F-pointer instead.

`NETFILES` is a live parameter. A `server;file` VOC reference is refused whatever it holds, but **bit 2
(`NETFILES=2`) opens API request 29**, which opens an `account;file` file for a
client that has signed in. It is 0 as delivered. With it set, the request is
still refused unless the account's NET gate is on (`set.net.access on`), and
under `SDCLIENT`; see [API access](09-api-access.html). A
pointer in the account's VOC that leads out of the account is refused too, unless
the account has the `os` switch on. Setting `NETFILES=2` is therefore not enough to
open such a file: turn the NET gate on only knowing what it allows.

There is no license parameter and no license verb. SD Core Solo is GPL v3 and is not
licensed per site or per user; `config gpl` displays the license text.
