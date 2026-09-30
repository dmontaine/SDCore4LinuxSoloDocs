Title: Administrator commands
Subtitle: ADMIN, the commands that need it, and the maintenance verbs.

**The administrator commands are in your own account, and they need `ADMIN`
first.** There is no separate administrator account to sign in to, and no
`sdsys` Linux user to become.

## `ADMIN`

```
:admin
Administrator password:
Administrator commands unlocked for this session
```

**Type the administrator password** — or, on a managed computer, the global
password. **It lasts until you leave SD or type `ADMIN OFF`**:

```
:admin off
Administrator commands locked
```

| | |
|---|---|
| *Wrong password - administrator commands stay locked* | one try; type `ADMIN` again |
| *Administrator commands are already unlocked* | nothing to do |
| *No administrator password is set on this system* | the installation did not set one; install again |

**A session signed in with the global password is already unlocked.**

**Every attempt is recorded** in the audit trail, unlocked or refused. The
password is never displayed, stored or logged.

## What needs it

**Refused without `ADMIN`, with *Command requires administrator privileges*:**

| | |
|---|---|
| `SET.PASSWORD ADMIN` | change the administrator password — [The account and its passwords](05-account-types.html) |
| `CONFIG` | report or set configuration — except `CONFIG GPL` and `CONFIG CONTRIB`, which need nothing |
| `SET.DATE` | set the session's date |
| `CLEAN.ACCOUNT` | empty the account's scratch files |
| `UPDATE.ACCOUNTS` | refresh the account's VOC |
| `LISTU`, `LOGOUT ALL` | [Sessions and locks](06a-sessions-and-locks.html) |
| `LIST.READU`, `LIST.LOCKS`, `LOCK`, `CLEAR.LOCKS`, `UNLOCK` | [Sessions and locks](06a-sessions-and-locks.html) |
| anything on the deny list | managed mode only — [Managed mode](15-managed-mode.html) |

**Refused without `ADMIN`, with *The VOC can only be changed after ADMIN*:**
editing the VOC directly — `ED VOC`, a program's `WRITE` or `DELETE` to the
VOC, `COPY` into it — and saving or deleting a sentence with `.S` and `.D`.
What SD writes to the VOC as a side effect of an ordinary command —
`CREATE.FILE`'s entry, the command stack — is not gated.

**Refused even with `ADMIN`**: changing the global catalogue (`CATALOG ...
GLOBAL`, `DELETE.CATALOG` of a global entry, a write to `global.bp.out`), and
the commands that are the SD Core for Linux server's — see
[Managed mode](15-managed-mode.html).

**`LOGOUT` on its own needs nothing** — it ends your own session, like `QUIT`
— **and nor does `sd -k`**, because it is a switch on the program, not a
command in a session. **`sh` needs nothing either**; see
[Operating system access](06b-operating-system-access.html). **Nor does
`SET.PASSWORD`** for your own account password — it asks for the current one
instead; see [The account and its passwords](05-account-types.html).

## The maintenance verbs

### `CONFIG`

```
config                     report every setting
config lptr                the same, to the default printer
config param value         set one, for this session only
config gpl                 display the licence
config contrib             display the contributors
```

**`config param value` sets a private, session-local value, not the
installation's.** The installation's settings live in `sd.conf` and are read
when SD starts — see [Configuration](16-configuration.html). This form
overrides one for the session you are in: the right tool for trying a value,
the wrong one for changing the installation.

| | |
|---|---|
| *New parameter value required* | `config numlocks` with nothing after it. **The report form is `config` alone** |
| *Not a recognised private configuration parameter name* | the name cannot be set per session |
| *Invalid value for this parameter* | it can, and the value is wrong |

### `SET.DATE`

```
set.date date
```

**Sets the date SD reports in this session, not the computer's clock.** SD
keeps an offset from the real date for the session; `DATE()` and `TIMEDATE()`
return the new date until the session ends. Other sessions and Linux are not
affected. The argument goes through SD's `D` conversion, so anything
`iconv(…, 'D')` accepts will do:

| | |
|---|---|
| *Date required* | nothing after `set.date` |
| *Invalid date format* | the argument is not a date SD can read |

It is for testing date-dependent code without touching the clock.

### `CLEAN.ACCOUNT`

```
:clean.account
Cleaned $COMO
Cleaned $hold
Cleaned $savedlists
```

Empties the account's captured transcripts (`$COMO`), its hold file of reports
(`$hold`) and its saved select lists (`$savedlists`). **Nothing else is
touched** — no data file, no program, no dictionary. A como capture that is
running is left alone and says so.

### `UPDATE.ACCOUNTS`

```
update.accounts {all}
```

**Copies SD's shipped command definitions into your VOC**, adding what is
missing and leaving your own VOC records alone. An upgrade runs it for you, so
you will not normally type it. With one account, `all` and no keyword do the
same thing.

**It never takes anything away.** A record you removed stays removed. To keep
your own version of one of SD's records, put `[locked]` in field 1 **after the
type code** — `V[locked]`, `PA[locked]` — and it is left alone. **A verb is
updated anyway**, because a locked verb would go on naming a program this
release replaced; you are told which ones.

## Not here

There is no `APPEND.SD.PATH`: the installer links `sd` into `~/.local/bin`, and
whether that directory is on your PATH is between you and your shell's startup
file. There are no `CREATE.ACCOUNT`, `DELETE.ACCOUNT`, `MODIFY.ACCOUNT`,
`GRANT`, `REVOKE` or `LIST.GRANTS` — see [Not in SD Core](14-not-in-sd-core.html).
