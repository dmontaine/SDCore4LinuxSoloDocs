Title: Other hardening
Subtitle: The global catalogue, the logs, line endings, and the rest of the smaller changes.

Everything on this page is a change you may notice while testing, grouped by
what it touches. The identity model and the file permissions are on
[Security](12-security.html); this is the remainder.

## The global catalogue

**Nobody changes the global catalogue in a session** — not with `ADMIN`, not with
the global password. `catalog ... global`, and `delete.catalog` of a name that is
global (a name starting `*`, `!`, `_` or `$`), are refused, and so is a program's
own `write`, `delete` or `clear` of `global.bp.out` and `gcat`:

```
:catalog bp myprog global
The global catalogue holds the SD Core server's programs from GLOBAL.BP.OUT and is changed only by SYNC.GLOBAL.CATALOG
```

This matters because the global catalogue holds the programs SD runs for
everybody, `$login` among them. **Replacing one runs your code in every session,
and deleting one stops everybody signing in.** In the multiuser product `ADMIN`
was enough to overwrite SD's own `$` and `!` programs; here it is not.

**On a managed computer the catalogue is the server's**, made from the compiled
programs the server puts in `global.bp.out` and kept in step by
`SYNC.GLOBAL.CATALOG`, which only a session signed in with the global password
can run. See [Managed mode](15-managed-mode.html). **On a standalone computer
the global catalogue is only SD's own** and `SYNC.GLOBAL.CATALOG` says there is
nothing to manage.

**Nothing changes for local and private cataloguing**, which is what programmers
use day to day:

```
catalog bp myprog          private catalogue, this account
catalog bp myprog local    this account's VOC
```

Both still work, and neither offers to remove a same-named global entry any
more. The only thing you cannot do is catalogue a program whose name starts with
`*`, `!`, `_` or `$` — those characters mean *global*. Name it without one.

## The pcode library

`~/SDCoreSolo/bin/pcode` is the interpreter itself, which SD loads into shared
memory at start-up and every session then runs. It is your file, like all the
rest, and is replaced by an upgrade.

## Scheduled jobs

A cron job or systemd timer can run any SD command, as you, with the kept
password. There is no permit list. It has its own page:
**[Scheduled jobs](04-scheduled-jobs.html)**.

## The logs

There are two SD keeps itself, and they are not interchangeable.

| File | Where | For |
|---|---|---|
| `audit` | `~/SDCoreSolo` | **who did what** — sign-ins, refusals, `ADMIN`, passwords. See [Security and the operating system](12a-security-and-the-operating-system.html#the-audit-trail) |
| `errlog` | `~/SDCoreSolo` | diagnostics |

**A third place is worth checking, and it is not a file SD writes at all:
`journalctl`.** SD logs to syslog under the tag `sd_Log`, and your user manager
logs the service:

```sh
journalctl -t sd_Log
journalctl --user -u sd-solo.service
```

### API connections in the journal

Every accepted API connection is logged with the address and port it came from
(measured on a Solo tree, with the API on `local`):

```
sd_Log[115214]: API connection over TCP from 127.0.0.1 port 41506 (SD login required)
```

**Nothing is refused on the strength of it.** This records who connected; it does
not decide who may. The API's own checks — the SCRAM login — are what decide.

**A connection forwarded over ssh shows the tunnel's own endpoint, not the person
at the far end**, because the tunnel genuinely does terminate on this computer.

## Line endings

**Directory files exist so you can edit their records with an ordinary text
editor**, and a file that started life on a Windows machine — a CSV saved from
Excel, a record pasted from Notepad — may still carry CR+LF line endings. SD reads
either ending correctly: only the CR+LF pair that ends a line is treated as a line
ending, so a bare CR that happens to be data is left exactly as it is.

**What SD itself writes is LF only**, for `writeseq`/`writecsv` output,
`como`-captured output and the error log. **SD's CSV statements are documented as
following RFC 4180**, which technically asks for CR+LF — if you need output
another program expects to be CR+LF-terminated, check that program's own
tolerance for LF-only lines rather than assume SD supplies the pair.

**Dynamic files are unaffected** — they are stored in SD's own format and are not
readable by other programs.

## The terminal

**The default terminal type is `linux`.** `term` on its own should report your
session's actual `TERM` — over ssh, whatever your client sent; locally, whatever
the terminal emulator or console set.

**63 definitions ship, compiling to 100 terminal names** — the extra names are
variants such as `vt100-w` and `vt220-at` — so `term wyse60` still works.

**A name that is not installed is refused and your current type is kept** —
*"Unrecognised terminal name"* — so a typo costs you nothing. **`term` with no
argument reports the type actually in force**, which is how to check.

Watch for near-misses all the same. There is no plain `vt320` — the shipped name
is `vt320-at`. `terminfo.src` ships with SD, so `sdtic` can add a definition that
is not there.

**Backspace works**, at the prompt and when you are asked for a password.

### The page is 120 × 36, not 80 × 24

**SD's default terminal size is 120 columns by 36 lines.** It is not a cosmetic
default: the shipped `@` dictionary records and the default `list` report layouts
are formatted for 120 columns. **A terminal narrower than that makes ordinary
reports look wrapped or truncated**, which reads as a formatting bug and is not
one.

`term` reports the size in force, above the `Device` line (measured on Solo from
a piped session, which is why the type reads `linux` and not your terminal's):

```
:term
Page width: 120
Page depth: 36
Device    : linux
```

**The size is worked out at login**, in this order: the `LINES` and `COLUMNS`
environment variables if they are numeric, otherwise the terminfo entry's `lines`
and `cols`, **otherwise 36 and 120** — then raised to a minimum of 10 × 20 if
smaller. So a console or ssh session normally gets its real window size and
120 × 36 is the fallback when nothing answers, which is the case for a phantom or
a piped script.

> **`term default` restores it, and it prints nothing when it does.** It sets the
> same 120 × 36 the login path falls back to and returns silently, so run a bare
> `term` after it to see the result. `term 120,36` does the same by hand.

## Running SD

| | |
|---|---|
| The unit | `sd-solo.service`, and with an API `sd-solo-api.socket` |
| After an unclean shutdown | SD starts anyway, once the daemon's own liveness — not just the segment's presence — is checked. See [Running SD](03-running-sd.html) |
| `sd-solo <command>` | uses the kept password, and any command may be given — see [Scheduled jobs](04-scheduled-jobs.html) |
| End of piped input | ends the session — at the command prompt, at `PAUSE` and at the Ctrl-R search. It used to spin at full CPU |

## Setting no password

**On Solo every session asks for the account password**, and an account with none
set cannot be signed in to at all — the installer always sets one, or, on a
computer installed from a control file, the first console session does. See
[The account and its passwords](05-account-types.html).
