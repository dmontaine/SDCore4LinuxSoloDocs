Title: Operating System Access
Subtitle: The `sh` and `!` verbs, OS.EXECUTE, and what they reach on a Solo computer.

`sh` runs a Linux command from the SD prompt. `!` is the same verb under a
shorter name. **They are the only way out of SD to the operating system from
TCL**; `OS.EXECUTE` is the way from a program.

**On a Solo computer they are yours to use, with no `ADMIN`** — at the keyboard,
over ssh and through the API. SD, its data and the Linux user it runs as are all
one person's, so there is nobody for a gate to keep out. The multiuser SD Core
for Linux ran them at each account's own Linux permissions; Solo does the same
with the one account, which has the permissions of your own Linux user.

**What they run as:** your Linux user, with your own permissions. SD never runs
as root, so nothing it starts can be root either. A command that needs `sudo`
fails as it would in any script run without a terminal to ask for the password.

SD folds case, so a command may be typed in either case. Commands are shown here
in lower case.

## The two verbs

```
sh command
! command
```

Everything after the verb is handed to the shell as typed:

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
all, with the same rights as `sh`:

```
   os.execute 'echo alpha-beta | grep alpha'
   crt 'status=':status()
```

```
alpha-beta
status=0
```

That is the way to run a command SD's filter refuses. **The form `execute 'sh …'`
from a program goes through the `sh` verb itself**, and is filtered like a
typed one.

## The editors

The full-screen editors `nano` and `micro` run as ordinary Linux programs, in
your terminal, with your permissions and no gate; `ed` and `edit` are SD's own
line editor. See [Development and file commands](07-programmer-commands.html#editors).

## Over ssh and the API

**The same is true of a remote session.** An ssh or API session is you, and may
reach the operating system as you. Anyone who has the account password — and,
over ssh, your key or Linux password — can therefore run Linux commands as your
Linux user. That is one reason every session asks for the account password; see
[Security](12-security.html).

## On a managed computer

**The SD Core for Linux server can put `sh` on the list of denied commands** (see
[Managed mode](15-managed-mode.html)). It takes `!` with it, because SD denies a
verb by what it runs and both are the same command, and both then need `ADMIN`
first. **A program's `OS.EXECUTE` is not a command the list can hold**, so the
list alone does not close the operating system off from a program.

## See also

[Administrator commands](06-administrator-commands.html) ·
[Sessions and locks](06a-sessions-and-locks.html) ·
[Security and the operating system](12a-security-and-the-operating-system.html).
