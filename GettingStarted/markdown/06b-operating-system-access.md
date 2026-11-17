Title: Operating System Access
Subtitle: The two switches, sh and os, that decide whether SD can reach the operating system: off as delivered, turned on by the administrator.

`sh` runs a Linux command from the SD prompt (the shorter name `!` was removed in
2.0-0). `OS.EXECUTE` is the way from a program, and a program can also
reach the operating system by naming a file or directory outside the account.
**All of it is off as delivered**, and the administrator turns it on, in two
independent switches, with `set.os.access`.

## The two switches

| Switch | What it lets the account do |
|---|---|
| **`sh`** | the `sh` verb |
| **`os`** | `OS.EXECUTE`; `OPENSEQ`, `OPENPATH`, `DIR()`, `DELETESEQ`, `OPENTEMP`, `DIGEST` of a file and the file-system keys of `OSPATH()` on a path **outside the account's own directory**; a print unit's pathname (`SETPTR ... AS PATHNAME`, `SETPU`, a CSV report's `TO` pathname) and its `SPOOLER` command outside it; the embedded Python; the editors `nano` and `micro`; and a VOC pointer to a file outside the account |

**Both are off as delivered, on a new install and on an upgrade unless you said
yes when the installer asked.** The account's own directory is never asked
about: inside it everything works whatever the switches say. **The
administrator needs no switch**: after `admin` the session passes both for as
long as it lasts.

**What they run as, when they are on:** your Linux user, with your own
permissions. SD never runs as root, so nothing it starts can be root either. A
command that needs `sudo` fails as it would in any script run without a terminal
to ask for the password. Turning a switch on is therefore not a small thing:
whoever signs in to SD, at the keyboard, over ssh or through the API, can do
what your Linux user can do.

**One narrower door for files only.** With the `os` switch off, an `sd.conf` line
`NETDIRS=/srv/sales;/srv/shared` lets the account read and write files in the folders it
names, and below them, by pathname, without turning the switch on. None is listed as
delivered, and listing one is a loosening for every session of the account. See
[Configuration](16-configuration.html).

## Looking and changing: `set.os.access`

An administrator command (type `admin` first):

```
:set.os.access
Account sduser has SH no, OS.EXECUTE no
:set.os.access sh on
Account sduser has SH yes, OS.EXECUTE no
:set.os.access os on
Account sduser has SH yes, OS.EXECUTE yes
:set.os.access sh off
Account sduser has SH no, OS.EXECUTE yes
```

With no argument it shows the state and changes nothing. **One switch, one
action, per command**: a command that names both, one word, or anything but `sh`
or `os` followed by `on` or `off` prints `Syntax: set.os.access ...` and changes
nothing. Saying it a second time says `sduser already had that access; nothing
changed`. The answer always gives **both** switches, because `sh off` says
nothing about `os`. The change is written to the audit trail
(`set.os.access sh on account=sduser`), and it is kept in fields 7 and 8 of the
account's register record (`yes` is on; anything else is off, and a record with
fewer fields is off).

**A change is seen at the next sign-on, never by a session that is already
running**: sign out and in again, or open a new ssh or API session. The API
server reads the switches when it enters the account, as the sign-on does.

## All the gates on one page: `tool.gates`

```
tool.gates
```

**A full-screen page, in any session** (140 columns by 43 lines at least). It
lists the six gates this account has - **Shell** (`sh`), **OS.EXECUTE** (`os`),
**Remote files** (`net`, [API access](09-api-access.html)), **AI**
([The AI helper](24-the-ai-helper.html)), **File transfer** (`xfer`, `set.xfer.access`;
*Moving Files In and Out* in the User documentation) and **Scheduled jobs** (`jobs`,
`set.jobs.access`; [Scheduled jobs](04-scheduled-jobs.html)), the last two read each time
they are used, so "this session" is what the register says now - with, for each: whether it is ON or
off **for this session**, what the register holds (shown to the administrator
only), how it is delivered (off, every one), whether it reaches beyond SD (every
one does) and when a change reaches you. A box under the list explains the row
under the cursor in plain sentences: what ON lets the account do, what it can
still do with it off, and the command that changes it.

| Key | What it does |
|---|---|
| Up, Down, Page Up, Page Down, Home, End, a click | move to a gate |
| Enter, the space bar (**after `admin`**) | turn the gate on the cursor's row ON, or off, through its own verb |
| F5 | ask the system again |
| F1, ?, a click on the [?] | explain the page |
| Ctrl-Q | leave |

**What the columns say.** *This session* is what the system holds for the
session you are typing in: `sh`, `os` and `net` were loaded when you signed on,
so a change made later shows there **after the next sign-on**, which is why the
*Register* column is there (what the next sign-on will load). The AI's gate is
the helper's own setting, looked at at every request, so the two columns agree;
if the helper is not running both show a dash and the row says so. The page
**asks and does not try**: it never tests an operation to find out whether it
works.

**What Enter does (administrator only).** It builds one command - `set.os.access
sh on`, `set.os.access os off`, `set.net.access on`, `set.ai gate on` and so on -
shows it in a box with what it will do, and runs it only when you answer Run;
Cancel is what Enter chooses. **Turning a gate ON asks you to type its name**
(`sh`, `os`, `net` or `ai`), because every one of them reaches beyond SD, and the
box says that a gate off as delivered is being turned on and that leaving it off
is the tighter choice. The verb asks for the administrator itself and writes its
own audit line (`set.os.access sh on account=sduser`); the page writes only the
line that says it was started (`tool start app=tool.gates`), and only for the
administrator. After the command it reads again and shows the verb's own last line
in the status bar. **There is no key that turns everything on, and nothing is
on as delivered.** A session that has not typed `admin` sees the page and cannot
change anything: Enter says so and opens no box.

## What a refusal looks like

With both switches off:

```
:sh echo hi
sduser is not permitted to use the operating system shell
```

A program's `OS.EXECUTE` is a run-time error, and a path outside the account is
the `ELSE` branch with status 3035 (the number of "permission"; it is the same
number as the VOC refusal, and the statement says what was refused):

```
<address>: sduser is not permitted to use OS.EXECUTE at line 2
```

```
   openseq '/etc/hostname' to f then crt 'opened' else crt 'refused ' : status()
refused 3035
   x = dir('/etc') ; crt dcount(x, @fm) : ' ' : status()
0 3035
```

`DIGEST` of an outside file, `OSPATH()` and `OPENTEMP` give nothing and status
3035; a print file or a `SETPTR ... AS PATHNAME` outside the account says
`sduser has no OS access, so a file or directory outside its own account cannot
be reached by pathname (the administrator can give it: set.os.access os on)`;
`micro` says that it needs OS.EXECUTE permission. `ed` and `edit`, SD's own line editor,
need none of it.

## The two verbs

```
sh command
```

With `sh` on, everything after the verb is the command. It runs in the account's own directory, at the account's own Linux permissions:

```
:sh echo hello-from-the-shell
hello-from-the-shell
:sh pwd
/home/you/SDCoreSolo/user_accounts/sduser
```

**The shell is `bash -c`.** The command starts in the account's directory,
`~/SDCoreSolo/user_accounts/sduser`, and runs to completion with its output
shown; there is no terminal for it to take over. Your PATH is the one SD was
started with — under systemd that is the user manager's, which normally does not
include `~/.local/bin`, so `sd-solo` itself is not found by name inside `sh` (use
the full path).

## A filter on the command, not on you

**`sh` refuses a command that contains a shell metacharacter or a line
break** — `;` `|` `&` `$` `` ` `` `<` `>`:

```
:sh echo alpha-beta | grep alpha
Error 2 executing operating system command
:sh echo hi; ls /somewhere
Error 2 executing operating system command
```

**That is a sanitizer, not a permission.** It stops a one-line command being
turned into two by accident — a record id or a variable pasted into it — and it
is applied the same way to everyone. A plain command with arguments is fine:
`sh ls -la`, `sh grep -c error /var/log/messages`. **A bare `sh`, with nothing
after it, starts an interactive shell** (the filter is for a typed command); `exit` brings you back to the SD
prompt. It needs the `sh` switch like any other `sh`.

## `OS.EXECUTE` is not filtered

**A program's `OS.EXECUTE` runs the string it is given as written**, pipes and
all, with the same rights as `sh`, once the `os` switch is on:

```
   os.execute 'echo alpha-beta | grep alpha'
   crt 'status=':status()
```

```
alpha-beta
status=0
```

That is the way to run a command SD's filter refuses. **The form `execute 'sh …'`
from a program goes through the `sh` verb itself**, and is filtered, and asked
about the `sh` switch, like a typed one.

## The editors

The full-screen editor `micro` runs as an ordinary Linux program, in
your terminal, with your permissions, and is **behind the `os` switch** (it
is a way out of SD); `ed` and `edit` (SD's own line editor) and `sed` (SD's own
full-screen editor) are not. See
[Development and file commands](07-programmer-commands.html#editors).

## Over ssh and the API

**The switches are loaded the same way for a remote session.** An ssh or API
session signs in to the account, and the sign-on reads the register: with a
switch off, it is off there too. With one on, anyone who has the account
password (and, over ssh, your key or Linux password) can run Linux commands as
your Linux user. That is one reason every session asks for the account password;
see [Security](12-security.html).

## At install and at upgrade

**A fresh install asks**, once, on a terminal, with the default no for each:
`Turn on the shell (SH) for your SD account? [y/N]` and `Turn on OS access
(os.execute, outside files, Python, micro) for your SD account? [y/N]`. The
answer can be given on the command line, `--os-access none|sh|os|both`, or in a
control file (`os-access=`); with no terminal, or with `--yes` and no answer,
nothing is turned on, and both are off.

**An upgrade does not ask**: it keeps the two switches as they are. A Solo from
before this release (LS1.1-3) had no switches, its shell and OS access always
on; after the upgrade it has neither set, so **both are off** until you type
ADMIN and then `set.os.access sh on` or `set.os.access os on`.

## On a managed computer

**The SD Core for Linux server can put `sh` on the list of denied commands** (see
[Managed mode](15-managed-mode.html)), and it then needs `admin` first. That list is separate from the switches: a denied `sh` needs `admin` even
when the switch is on, and `set.os.access` is itself an administrator command.

## Continued in

[Showing a Document with pdf](06b1-showing-a-document-with-pdf.html) — the one
bypass of the os switch: showing a document with pdf, naming a reader, where
the documents come from, and the context help
