Title: Your first thirty minutes
Subtitle: From a finished install to a file with data in it, an administrator command, and a command run from a script.

This page assumes SD Core for Linux Solo is installed. It is a walkthrough,
not a reference — every step links to the page that explains it properly.

## 1. Start SD

**Open a new terminal** — one opened before the install may not have
`~/.local/bin` on its PATH yet — and type:

```
sd-solo
```

**SD is already running.** It is your own systemd user service, so you do not
type `sd-solo -start`. See [Running SD](03-running-sd.html).

**It asks for the account password**, the one you chose when installing. On a
computer installed from a control file there is none yet, and it asks you to
choose one now:

```
This account has no password yet. Choose one now - SD Core for Linux Solo asks for it every time it is used.
```

The sign-on banner names the product and its version, `LS1.1-2`.

## 2. Look around

```
who
listf
term
```

| | |
|---|---|
| **`who`** | the account — always `sduser` |
| `listf` | the files in it |
| **`term`** | your terminal type and page size |

**`term` also reports the page size, and SD's default is 120 × 36 — not
80 × 24.** The shipped dictionaries and the default `list` layouts are
formatted for 120 columns, so **a terminal narrower than that makes ordinary
reports look wrapped or truncated** and the report is not at fault. Widen the
window, or set it for the session with `term default`. See
[Other hardening](13-hardening.html#the-terminal).

## 3. Make a file and put something in it

```
create.file customers
ed customers 1001
```

**`ed`** is the **line** editor, and it needs nothing installed — **`edit`** is
an alias for it. For a full screen, **`nano`** opens the record in `nano` and
**`micro`** opens it in `micro` — see
[Development and file commands](07-programmer-commands.html#editors).

In **`ed`**: `i` to insert, type your lines, a full stop on its own line to stop
inserting, then `fi` to file and exit.

> **You do not have to write programs in `ed`.** The account's `bp` file is a
> **directory file** — an ordinary Linux directory with one file per program —
> so any text editor works on it just as well:
>
> ```
> ~/SDCoreSolo/user_accounts/sduser/bp
> ```
>
> Save the file, then **`basic`** and **`catalog`** it from inside SD as usual.

```
list customers
count customers
```

**Commands are lower case now.** Typing `LIST` still works — SD converts it.
See [Lower case](11-lower-case.html).

**THIS IS THE POINT AT WHICH MOST THINGS SHOULD FEEL LIKE OpenQM.** If
anything in ordinary data work behaves differently and is not described in this
set, that is worth reporting.

## 4. An administrator command

```
listu
```

```
Command requires administrator privileges
```

**That is the administrator gate.** Unlock it for this session:

```
admin
```

Type the administrator password — or, on a managed computer, the global
password — and `listu` works, and so does every other administrator command,
until you type `admin off` or leave. See
[Administrator commands](06-administrator-commands.html).

## 5. Leave

```
off
```

## 6. A command from a script

**From a terminal**, not from inside SD:

```
sd-solo list customers
```

**It runs the one command and returns, with no password prompt.** A command on
the command line uses a copy of the account password kept for you — which is
what lets a script or a scheduled job use SD. See
[Scheduled jobs](04-scheduled-jobs.html).

## What to try next

1. **Your own application data.** **There is no restore utility**, so the
   way in is a short BASIC program that reads your exported data and writes the
   records. Then query it — the query processor is where most of the surface
   area is.
2. **A client program against the API**, if you chose it when installing. It
   signs in as `sduser` with the account password, on port 4249, and needs a client library from this release, because the old
   cleartext login is gone. See [API access](09-api-access.html) and
   [Client distribution](10-client-distribution.html).
3. **ssh straight into `sd-solo`**, if you gave the installer a public key: `ssh
   <your Linux user>@localhost` lands at SD's password prompt. See
   [ssh access](08-ssh-access.html).
4. **An upgrade.** `bash installsdsolo.sh --upgrade` — see
   [Upgrading and uninstalling](01a-upgrading-and-uninstalling.html).

## When something goes wrong

| | |
|---|---|
| Something SD did, and who did it | `audit`, in `~/SDCoreSolo` |
| Diagnostics, and API connections | `errlog`, same place |
| The service | `systemctl --user status sd-solo.service`, and `journalctl --user -u sd-solo.service` |

[Other hardening](13-hardening.html#the-logs) explains which log answers which
question.

**When you report something, say which build.** The release stamp is on the
sign-on banner, in `sd-solo --version`, and in `~/SDCoreSolo/changelog`.
