Title: Operating System Access
Subtitle: The two switches, sh and os, that decide whether SD can reach the operating system: off as delivered, turned on by the administrator.

`sh` runs a Linux command from the SD prompt. `!` is the same verb under a
shorter name. `OS.EXECUTE` is the way from a program, and a program can also
reach the operating system by naming a file or directory outside the account.
**All of it is off as delivered**, and the administrator turns it on, in two
independent switches, with `set.os.access`.

## The two switches

| Switch | What it lets the account do |
|---|---|
| **`sh`** | the `sh` verb and `!` |
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

## What a refusal looks like

With both switches off:

```
:sh echo hi
sduser is not permitted to use the operating system shell
:! echo hi
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
`nano` and `micro` say that they need OS.EXECUTE permission. `ed` and `edit`, SD's own line editor,
need none of it.

## The two verbs

```
sh command
! command
```

With `sh` on, everything after the verb is handed to the shell as typed:

```
:sh echo hello-from-the-shell
hello-from-the-shell
:! echo via-the-bang-form
via-the-bang-form
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

**Both verbs refuse a command that contains a shell metacharacter or a line
break** — `;` `|` `&` `$` `` ` `` `<` `>`:

```
:sh echo alpha-beta | grep alpha
Error 2 executing operating system command
:! echo hi; ls /somewhere
Error 2 executing operating system command
```

**That is a sanitizer, not a permission.** It stops a one-line command being
turned into two by accident — a record id or a variable pasted into it — and it
is applied the same way to everyone. A plain command with arguments is fine:
`sh ls -la`, `sh grep -c error /var/log/messages`. **A bare `sh`, with nothing
after it, is refused the same way**, so there is no interactive shell to be left
in from TCL: to get a shell, use your own terminal.

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

The full-screen editors `nano` and `micro` run as ordinary Linux programs, in
your terminal, with your permissions, and are **behind the `os` switch** (they
are a way out of SD); `ed` and `edit` are SD's own line editor and are not. See
[Development and file commands](07-programmer-commands.html#editors).

## Over ssh and the API

**The switches are loaded the same way for a remote session.** An ssh or API
session signs in to the account, and the sign-on reads the register: with a
switch off, it is off there too. With one on, anyone who has the account
password (and, over ssh, your key or Linux password) can run Linux commands as
your Linux user. That is one reason every session asks for the account password;
see [Security](12-security.html).

## At install and at upgrade

**The installer asks**, once, on a terminal, with the default no for each:
`Turn on the shell (sh and !) for your SD account? [y/N]` and `Turn on OS access
(os.execute, outside files, Python, nano, micro) for your SD account? [y/N]`. An
upgrade asks the same, because **before this release the shell and OS access
were always on**, and an existing Solo would otherwise lose them without a word;
say yes to each you want to keep. The answer can be given on the command line,
`--os-access none|sh|os|both`, or in a control file (`os-access=`); with no
terminal, or with `--yes` and no answer, nothing is turned on here, which on a
new account is off and on an upgrade leaves the switches as they were.

## On a managed computer

**The SD Core for Linux server can put `sh` on the list of denied commands** (see
[Managed mode](15-managed-mode.html)). It takes `!` with it, because SD denies a
verb by what it runs and both are the same command, and both then need `admin`
first. That list is separate from the switches: a denied `sh` needs `admin` even
when the switch is on, and `set.os.access` is itself an administrator command.

## See also

[Administrator commands](06-administrator-commands.html) ·
[Sessions and locks](06a-sessions-and-locks.html) ·
[Security and the operating system](12a-security-and-the-operating-system.html).
