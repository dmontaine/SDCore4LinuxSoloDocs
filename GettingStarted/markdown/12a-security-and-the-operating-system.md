Title: Security and the operating system
Subtitle: Reaching the computer from inside SD, and the audit trail.

This page continues [Security](12-security.html).

## Reaching the operating system from inside SD

**There are three ways out of SD onto the computer, and all three are open to
your session with no `ADMIN`.**

| | What it is |
|---|---|
| **`sh`** and `!` | one operating-system command from the `:` prompt |
| `OS.EXECUTE` | the operating system from inside a BASIC program |
| **`nano`** and **`micro`** | a text editor, running outside SD |

**They run as your Linux user.** Nothing in SD stands between a session and the
computer beyond what Linux itself allows that user — see
[Operating system access](06b-operating-system-access.html). SD keeps no second
permission list behind them: Linux already provides the wall SD would otherwise
have to build.

**That applies to every kind of session**, ssh and the API included. Anyone who
has the account password — and, over ssh, your key or your Linux password — can
run Linux commands as you. That is why the account password matters, and why it
should be one only you know.

**On a managed computer**, the SD Core for Linux server can put `sh` on the list
of denied commands, and `!` goes with it. **A program's `OS.EXECUTE` is not a
command the list can hold**, so the list alone does not close the computer off
from a program. See [Managed mode](15-managed-mode.html).

## There is no privileged helper

The multiuser SD Core for Linux runs its account and password work through a
`sudo`-scoped helper, `sd-elevate`. **Solo has none**: nothing in SD ever asks
for `root`, and the one place it once ran one — creating accounts — is gone. The
few things that do need `sudo` — packages, a firewall rule, the `sshd_config.d`
block, linger — are done by scripts you run, and the scripts say when.

## The audit trail

`~/SDCoreSolo/audit` records what happened in SD, one line each, with the date,
time and user:

```
2026-09-30 02:56:31 user=sduser uid=1 pid=109059 LOGIN PASSWORD account=sduser via=stored
2026-09-30 02:56:31 user=sduser uid=1 pid=109059 LOGIN account=sduser
```

| Recorded | |
|---|---|
| **Sign-ins** | every one, with how the password was proved — `via=account`, `via=global`, `via=first` (the first password, chosen at the console), or `via=stored` for a command-line `sd <command>` — and every refusal |
| **`ADMIN`** | every unlock and every refusal |
| **Passwords** | a change of the account, administrator or global password, and a refused change |
| **The API** | every login, every refused request, and every failed login with its reason — `API REFUSED user=sduser reason=wrong password`. **The address is not recorded** |
| **Managed mode** | `DENY.VERBS` changes and `SYNC.GLOBAL.CATALOG` runs, and the installer's own internal sessions |

**The refusals are the interesting half.** An `ADMIN REFUSED`, a `LOGIN REFUSED`
or an `API REFUSED` is somebody trying something that did not work.

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

## What is still not true

**SD has no file-level access control of its own** on the keyboard and ssh paths:
a session opens files as you, so Linux's permissions on your directory are the
only boundary. On the API the multiuser product confines a session to the account
it stands in; that has **not been measured on Solo** — see
[API access](09-api-access.html).
