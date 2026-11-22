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
**An upgrade keeps your `sd.conf`**: the installer puts a kept file back and takes
out only the parameters SD no longer has.

### A name SD does not recognize stops it starting

An unrecognized parameter is not ignored. The parser abandons the file with
`Unrecognized configuration parameter`, and SD does not start.

That is why a name SD does not have has to be taken out of the file. These
thirteen names are not parameters of SD: `APILOGIN`, `CODEPAGE`, `CREATUSR`,
`EXCLREM`, `FILERULE`, `FIXUSERS`, `GDI`, `INVCASE`, `JNLDIR`, `JNLMODE`,
`PORTMAP`, `RINGWAIT` and `TXCHAR`. The installer removes those lines from a
`sd.conf` it keeps (an upgrade, or a reinstall that keeps the configuration), and
says which it removed. A file copied in by hand from an older installation must have
them taken out before SD will start.

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
...
YEARBASE  1930
```

SD accepts **58** parameters (`APIPORT` does not exist here — see *The API* in [Configuration - The Parameters](16c-the-parameters.html#the-api)). The verb prints all of them except
`BACKUPDIR`, `NETDIRS` and `SDSYS`, and `DEBUG`, `FDS` and `PHANTOMS` only when they are set; it also prints `TIMEZONE`, which is not a file parameter.

`NETDIRS` is the one to know about. It decides what a session held to its account
by the `os` switch, the API's included, may reach outside its own account (see
[The parameters](16c-the-parameters.html)).
The verb will not tell you what it is set to, and `config('NETDIRS')` does not read it. Read it from the file.

**`config` pages its output** — press Return for more, or `Q`.

The `config()` function reads one parameter from a program:

```
group.size = config('GRPSIZE')
```

**55 of the 58 parameters are readable this way.** The three that are not are `BACKUPDIR`,
`NETDIRS` and `SDSYS`; `TIMEZONE`, which is not a file parameter, can be read as well. A name of more than sixteen characters is not found (the function answers
an empty string and `status()` is not 0); no parameter name is that long, so that
only shows up if you pass something which is not a parameter.

Two further forms exist. `config lptr` reports the settings of the default
printer, and `config gpl` and `config contrib` display the license and the list of
contributors — those two need no `admin`.

## Changing a parameter

Editing `sd.conf` and restarting SD is the durable route, and for most parameters
it is the only one. Fourteen tuning parameters can also be changed from SD itself, with
`set.config` or the `E` key of `tool.config` (see [Changing a Parameter from SD](16b-changing-a-parameter-from-sd.html)).

**29 settings can also be changed for the current session** (after `admin`):

```
config sortmem 8192
```

That writes to the session's own copy of the settings, taken from shared memory
when the session started. It does not reach `sd.conf`, it does not affect any
other session, and it is gone when the session ends.

The 29 are `DHCACHE`, `DUMPDIR`, `FLTDIFF`, `FSYNC`, `GRPSIZE`, `INTPREC`,
`LPTRHIGH`, `LPTRWIDE`, `MAXCALL`, `MAXRLOCK`, `MUSTLOCK`, `NETREPLY`, `NETWAIT`,
`OBJECTS`, `OBJMEM`, `PWDELAY`, `RECCACHE`, `SAFEDIR`, `SDCLIENT`, `SH`, `SH1`,
`SORTMEM`, `SORTMRG`, `SORTWORK`, `SPOOLER`, `TEMPDIR`, `TERMINFO`, `TIMEZONE` and
`YEARBASE`.

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
made by the verb [`set.config`](16b-changing-a-parameter-from-sd.html), and only after you have typed a value, the page has checked
it against the range SD enforces and a box has shown the exact command and been answered Run. The
other parameters are not changed from SD; editing the file and restarting SD stays the way to change
them.

**After a change the page offers to restart SD.** When the file then says something else than SD is
running with, a second box, titled **Restart SD?**, says why the change needs a restart and that **a
restart stops and starts SD and signs out everyone, this session too**. It offers **Restart** and
**Not now**; Not now is what Enter chooses, and it leaves the change for the next time SD is started.
Restart closes the page, clears the screen, says SD is restarting and asks your own systemd to restart
`sd-solo.service`; the session ends, and you start SD again as usual a few seconds later. If SD does
not end the session within a minute, or the service is not active (SD was started by hand with
`sd-solo -start`), the reason is shown and the page comes back. The audit trail gets
`sd restart app=tool.config status=asked`, or `status=refused`. A value set back to what SD is already
running with needs no restart, and none is offered.

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
**R, the report**: the operating system part of `settings.report` (the configuration file and the install-time policy, the mode, the API and ssh, and the service), made when the view is opened and
on F5. It holds no key and no password hash. `F5` reads the file, the backup directory and the report
again, and while the report is made the status line says "Reading the settings (a few seconds)...",
and while `set.config` runs a change it says "Making the change (a few seconds)..."; `E` changes the parameter under the cursor, or says why it cannot be changed here; `F1`
explains the page; `Ctrl-Q` leaves. Opening the page writes the audit line
`tool start app=tool.config`; a change writes the line of [`set.config`](16b-changing-a-parameter-from-sd.html), and nothing else the
page does is written.

## Continued in

[Changing a Parameter from SD](16b-changing-a-parameter-from-sd.html) —
changing a parameter from SD with set.config and the E key, then every
parameter
