Title: Configuration - The Parameters
Subtitle: Every parameter SD accepts, by subject, and the parameters that were removed.

This page continues [Changing a Parameter from SD](16b-changing-a-parameter-from-sd.html).

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
| `CLEANUP` | 300 | Seconds between the checks that look for sessions that ended without logging out (a killed terminal, a lost connection) and clear them from the user table. 30 to 3600. The daemon wakes every ten seconds, so an interval is kept to within ten seconds; a very short one costs a pass over the user table that often. `config('CLEANUP')` reads it |
| `PWDELAY` | 3 | Seconds SD waits after a failed network sign-on before it answers; 0 removes the pause |
| `NETWAIT` | 5 | Seconds to wait for a connection to a server defined for SDNet (`set.server`, `set.netlogin`, a file opened through an `N` pointer): the TCP connection, then the TLS handshake, each. 1 to 3600; zero is refused |
| `NETREPLY` | 30 | Seconds to wait for each reply of such a server. 1 to 3600; zero is refused. `config('NETWAIT')` and `config('NETREPLY')` read them. See the SDNet page of the User set |
| `FDS` | unset | Limit on file descriptors. No limit when unset |

## Files and locking

| Parameter | Default | Effect |
|---|---|---|
| `GRPSIZE` | 4 | Default group size for a new dynamic file, in 1 KB units. Read by `create.file` and `configure.file` when no group size is given. **4 as shipped**. An SD Core Solo for Linux installed with an older release keeps the `sd.conf` it has, and so may keep 2 until you change it; a file keeps the size it was made with |
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
| `SDCLIENT` | 0 | Restricts what an API session may do. Non-zero disables file access outright (no file can be opened, so no dictionary can be evaluated either) and refuses `SDIConv`, `SDOConv`, `SDIConvs` and `SDOConvs` a conversion code that reads a file (`T` with a file) or calls a program (`U`); any non-zero value also refuses `SDOpenSeq`, `SDExecute`, `SDTrans` and `SDRTrans`; `2` additionally refuses any subroutine not compiled as callable from a client (a subroutine whose name begins with `$` must be so compiled at every setting, `0` included) |

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
| `INTPREC` | 11 | Digits of precision in integer arithmetic. Range 0 to 14 |
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

Thirteen parameters of older releases do not exist. Seven did nothing (`JNLMODE`, `JNLDIR`,
`CREATUSR`, `CODEPAGE`, `EXCLREM`, `RINGWAIT`, `TXCHAR`), and six more imitated
another product or chose between ways of working that SD does not have
(`APILOGIN`, `FILERULE`, `FIXUSERS`, `GDI`, `INVCASE`, `PORTMAP`). See [A name SD does not recognize stops it starting](16-configuration.html#a-name-sd-does-not-recognize-stops-it-starting)
for what happens to a kept `sd.conf` that still names one. The `PATH:` form
of a file name went with `FILERULE`: write an F-pointer instead.

`NETFILES` is a live parameter. A `server;file` VOC reference is refused whatever it holds, but **bit 2
(`NETFILES=2`) opens API request 29**, which opens an `account;file` file for a
client that has signed in. It is 0 as delivered. With it set, the request is
still refused unless the account's NET gate is on (`set.net.access on`, an administrator command, off as delivered), and
under `SDCLIENT`; see [API access](09-api-access.html). A
pointer in the account's VOC that leads out of the account is refused too, unless
the account has the `os` switch on. Setting `NETFILES=2` is therefore not enough to
open such a file: turn the NET gate on only knowing what it allows. When it opens
the file, the connection's containment applies: only the account the session stands
in, the shipped system files its VOC names and the directories named in `NETDIRS`.
The SDNet **client** (`set.server`, `set.netlogin`, a VOC pointer of type `N`) does
not read `NETFILES`; see the SDNet page of the User set.

There is no license parameter and no license verb. SD Core Solo for Linux is GPL v3 and is not
licensed per site or per user; `config gpl` displays the license text.
