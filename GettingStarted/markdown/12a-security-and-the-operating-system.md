Title: Security and the operating system
Subtitle: Reaching the computer from inside SD, and the audit trail.

This page continues [Security](12-security.html).

## Reaching the operating system from inside SD

**There are three ways out of SD onto the computer, and each is behind a switch
that is off as delivered.** The administrator turns a switch on with
`set.os.access` (after `admin`); no other command does.

| | What it is | Switch |
|---|---|---|
| **`sh`** | one operating-system command from the `:` prompt | `sh` |
| `OS.EXECUTE` | the operating system from inside a BASIC program, and a file or directory outside the account | `os` |
| **`micro`** | a text editor, running outside SD | `os` |

**When a switch is on, what it opens runs as your Linux user.** Nothing in SD
stands between a session and the computer beyond what Linux itself allows that
user — see [Operating system access](06b-operating-system-access.html). The
switches are the one wall SD keeps in front of that, per switch, and they are
off until you say otherwise: at install, at upgrade (the installer asks), or
later with `set.os.access`.

**That applies to every kind of session**, ssh and the API included: they read the
same switches. With a switch on, anyone who has the account password — and, over
ssh, your key or your Linux password — can run Linux commands as you. That is why
the account password matters, and why it should be one only you know.

**On a managed computer**, the SD Core for Linux server can put `sh` on the list
of denied commands. **A program's `OS.EXECUTE` is not a
command the list can hold**, so the list alone does not close the computer off
from a program; the `os` switch does. See [Managed mode](15-managed-mode.html).

## There is no privileged helper

The multiuser SD Core for Linux runs its account and password work through a
`sudo`-scoped helper, `sd-elevate`. **Solo has none**: nothing in SD ever asks
for `root`, and the one place it once ran one — creating accounts — is gone. The
few things that do need `sudo` — packages, a firewall rule, linger — are done by
scripts you run, and the scripts say when.

## The audit trail

`~/SDCoreSolo/audit` records what happened in SD, one line each, with the date,
time and user:

```
2026-09-30 02:56:31 user=sduser uid=1 pid=109059 login password account=sduser via=stored
2026-09-30 02:56:31 user=sduser uid=1 pid=109059 login account=sduser
```

Every word before the first `=` is lower case, the event names included. What
follows an `=` is data and keeps its case: the account name, a `reason=` text,
and what a caller typed in `command=`. Lines written before 7 October 2026 are
upper case (`LOGIN REFUSED`) and are not rewritten, so search a long-lived file
without regard to case.

| Recorded | |
|---|---|
| **Sign-ins** | every one, with how the password was proved — `via=account`, `via=global`, `via=first` (the first password, chosen at the console), or `via=stored` for a command-line `sd-solo <command>` — and every refusal |
| **`admin`** | every unlock and every refusal |
| **Passwords** | a change of the account, administrator or global password, and a refused change |
| **The API** | every login, every refused request, and every failed login with its reason — `api refused user=sduser reason=wrong password`. **The address is not recorded** |
| **Managed mode** | `deny.verbs` changes and `sync.global.catalog` runs, and the installer's own internal sessions |

**The refusals are the interesting half.** An `admin refused`, a `login refused`
or an `api refused` is somebody trying something that did not work.

**Nothing is ever discarded.** When SD starts and the file is 1 MB or more, it is
**renamed with the date and time and a new one started**. Removing the old ones
is your decision; SD will not, and they accumulate.

**This is not the error log and does not behave like it.** `errlog` throws away
its oldest half when it fills, and holds diagnostics, not a record of who did
what.

**The audit trail does not protect itself from you.** The multiuser product makes
the file append-only at the kernel level (`chattr +a`), which needs root to set
and to lift; Solo has no root, so the file is an ordinary one that belongs to
your Linux user. **You can edit or delete it, and so can `root`.** It is a record
for reading, not evidence that survives somebody with your Linux sign-in. The
system log — `journalctl` — keeps its own separate record of the service; **read
the two together.**

### Reading it: `tool.audit`

`tool.audit` is a full-screen page for the administrator (`admin` first). It needs a window of at
least 140 columns by 43 lines, like the other toolkit pages, and it asks for the administrator
before it reads anything.

The top box names the file shown, its size, the lines loaded, whether older lines remain, the filters in
force and whether the page is following the file. The list under it is the trail, the oldest loaded line
at the top and the newest at the bottom, one row to a line: time, user, event words, then the pairs. The
box at the foot shows the whole line under the cursor, wrapped, and the same line cut into its parts.

**The page reads from the end of the file, in blocks, and never reads a large trail whole.** It opens
on the newest 200 lines that pass the filter; when the cursor goes above the first line it loads 200
more. It keeps at most 3000 lines; beyond that it drops the newest while you go back and the oldest
while you follow. A last line that has no line end yet is left until it is whole.

| Key | What it does |
|---|---|
| Up, Down, PgUp, PgDn | move; above the first loaded line, older lines are loaded when the file has them |
| Home, End | the first loaded line; the newest line (End reads the file again) |
| F | asks, one after another: from, to, event words, account, user, text. `from` and `to` are the start of a time stamp (`2026-10-25`, or `2026-10-25 14:30`; `to` with only a date includes the whole day). Event, account, user and text are matched without regard to case; `account` is the line's `account=` value, `user` its `user=`. Esc leaves every filter as it was |
| / | the same as F: on every page `/` is the key that finds by typing |
| C | clears the filters |
| D | goes to the first line at or after a time; no filter changes |
| O | shows the next audit file: the current one, then the renamed ones, newest first |
| L | follows the file: new lines are read every three seconds, and the cursor stays on the newest line if it was there |
| X | saves the lines loaded to a record of the account's `$HOLD` file, named `audit.yyyymmdd.hhmmss`, and writes the audit line `tool export app=tool.audit lines=n record=id`. `ED $HOLD id` reads it |
| F5 | reads again |
| F1, ? | explain the page |
| Ctrl-Q | leave |

**The page changes nothing.** It runs no command and writes no record but the one X saves. Opening it
writes `tool start app=tool.audit`, and X writes the one export line; both are on the trail.

## What is still not true

**SD has no file-level access control of its own** on the keyboard and ssh paths:
a session opens files as you, so Linux's permissions on your directory are the
only boundary. On the API the multiuser product confines a session to the account
it stands in; that has **not been measured on Solo** — see
[API access](09-api-access.html).
