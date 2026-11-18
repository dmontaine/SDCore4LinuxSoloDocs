Title: Changing a Parameter from SD
Subtitle: set.config and the E key of the configuration screen: changing a parameter from inside SD.

This page continues [Configuration](16-configuration.html).

## Changing a parameter from SD: `set.config` and the `E` key

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
| `GRPDIR` | A directory: accounts are found there. |
| `INTPREC` | Changes the results of arithmetic in programs that already exist. |
| `MAXIDLEN` | Structural: the longest record id of the whole system, which reaches every file and program. |
| `MUSTLOCK` | Makes a write fail unless the record is locked first, which can stop programs that exist. |
| `NETDIRS` | A door: it widens what a session without the `os` switch, and an API session, may reach beyond the account's own directory. |
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
| `USRDIR` | A directory: accounts are found there. |
| `XFER` | A door: 1 opens file transfer over ssh when the account's switch is on. |
| `XFERFILE` | A transfer limit: raising it lets one file fill more of the disk. |
| `XFERIDLE` | A transfer limit: raising it keeps an unused slot open longer. |
| `XFERKIND` | A transfer filter: blank takes any plain-text file. |
| `XFERMAX` | A transfer limit: 0 lets a slot stay open for as long as its session does. |
| `XFERSLOT` | A transfer limit: raising it lets one slot fill more of the disk. |
| `XFERWAIT` | A transfer limit: raising it keeps a slot open longer before anyone connects. |
| `YEARBASE` | Changes how every program reads a two-digit year; SD checks no range. |

**What this does not do.** It does not restart SD: every change to `sd.conf` takes effect when SD is next stopped and started (`sd-solo -stop`, then `sd-solo -start`). It does not reach a file other than the `sd.conf` this SD reads. It cannot change a parameter that SD does not range-check, however useful that would be; such a parameter joins the list when SD itself checks it.

## Continued in

[Configuration - The Parameters](16c-the-parameters.html) — every parameter SD
accepts, by subject, and the parameters that were removed
