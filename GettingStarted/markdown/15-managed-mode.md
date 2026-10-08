Title: Managed mode
Subtitle: What an SD Core for Linux server can do to a Solo computer it manages, and how.

**A managed computer is a local database that an SD Core for Linux server also
manages.** The server is the only thing that manages Solo computers. **This page
will grow** as management features are added to SD Core for Linux — what is here
is what a managed computer offers the server today.

## What makes a computer managed

**A global password, and nothing else.** A computer is managed if, and only if,
it has one. It is given when installing — at the installer's last password
question (it can be left blank), or in the control file — and fixed until a new
installation. The server signs in with it. (Until 6 October 2026 the installer
asked "standalone or managed?" first; there is no such question now.)

**For the server to reach the computer, the API and ssh must be open to the
network** — your choice, made at the same install, because they are your choice
on every computer:

| | |
|---|---|
| **the API, `open`** | reachable from other computers — the server's way in. Port 4249 |
| **ssh, `open`** | Solo's own listener on port 4251, reachable from other computers: your Linux password, or the server's key. The installer installs the ssh server package for it and opens the port in `ufw` or firewalld if one is running; otherwise it tells you to allow TCP 4251 |

**Nothing forces either open.** If you give the installer a global password and
choose `off` or `local` for either, it warns you; the computer is then managed in
name only, because the server cannot reach it. The control file's `api=open` and
`ssh=open` lines are how an administrator sets both up for many computers.

See [Installing](01-installation.html), including the **control file** that sets
up many computers from one USB stick.

## How the server signs in

**As `sduser`, with the global password.** Over the API that is the whole of it.
Over ssh, on port 4251, the server first gets past ssh's own sign-in — its key, which
it installed through the API — and then gives SD the global password when `sd-solo`
asks. One account name carries two passwords: SD tries the
account password first and the global password second, which is why the two must
differ.

**A session signed in with the global password is a server session.** It has the
administrator commands unlocked from the start, and it is the only kind of
session that may use the commands below — the administrator password does not
open them.

**A server session can change every password on the computer** — the account
password with `set.password`, the administrator password with `set.password admin`,
and the global password with `set.password global`, which nothing else may use.
See [The account and its passwords](05-account-types.html).

**On a computer installed from a control file, the server can sign in before the
user has chosen an account password.** Until the user does, at that computer's
keyboard, the global password is the only one accepted.

## The server's ssh key (LS1.1-2)

**The server installs its own ssh key.** The server knows only this computer's
address and the name `sduser` — not the Linux user name that ssh needs. So after
it signs in over the API, it asks this computer to install the server's public
key, and the computer answers with its Linux user name, its host name, the key's
fingerprint, the fingerprint of Solo's own ssh server's host key and the port to
connect to (4251). From then on the
server reaches this computer over ssh with that key and the global password. The
same request lists and removes the server's keys.

- **Only a session signed in with the global password may ask.** The account
  password and `admin` are refused with *Only the SD Core server may manage ssh
  keys*.
- **The key can start `sd-solo` and nothing else** — no shell, no forwarding. It is the
  same kind of key line as the one you add yourself, in Solo's own key file,
  `~/SDCoreSolo/sshd/authorized_keys`: see [ssh access](08-ssh-access.html).
- **At most four server keys are kept.** A fifth is refused (*The ssh key request
  was refused: four Solo ssh keys are already installed*). **Your own ssh keys are
  never listed, counted or touched.**
- **Every use is in the audit trail**, with the key's fingerprint and the address
  it came from, never the key itself.
- **It does nothing on a computer with no global password.**
- **SD Core Solo for Windows answers the same request in the same words.**

**The server's certificate is pinned.** The first time the server's client library
connects to this computer's address and port, it remembers the computer's TLS
certificate. If a later connection to the same address presents a different one,
the library refuses it before sending anything, and says which line to remove.
Reinstalling this computer gives it a new certificate, so the server's line for it
has to be removed (the message names the file) before the server can connect
again. The very first connection is trusted. See [API access](09-api-access.html).

## The server's programs: `global.bp.out`

**The global catalogue of a managed computer holds the server's programs.** The
user can run them — `CALL *name` — and cannot add, replace or remove any, with or
without `admin`.

| | |
|---|---|
| `global.bp.out` | a file of **compiled programs only** — no source is installed. Empty after installation; the server fills it. It is `~/SDCoreSolo/global.bp.out` |
| `sync.global.catalog` | makes the global catalogue match `global.bp.out` |

**To add a program**, a server session copies its compiled object into
`global.bp.out` — for example from a `BP.OUT` it has written it to — and runs
`sync.global.catalog` (measured, with a small subroutine):

```
:copy from bp.out to global.bp.out myprog
1 record(s) copied.
:sync.global.catalog
catalogued *MYPROG
SYNC GLOBAL CATALOG DONE 1 catalogued 0 removed 0 refused
```

A session with no administrator rights can then `CALL *myprog(x)` — the name in
the `*` form is what a program uses to call a global one.

**To remove one**, delete it from `global.bp.out` and run `sync.global.catalog`
again; the `*MYPROG` entry goes:

```
:delete global.bp.out myprog
:sync.global.catalog
removed *MYPROG
SYNC GLOBAL CATALOG DONE 0 catalogued 1 removed 0 refused
```

**What `sync.global.catalog` does:** every object in `global.bp.out` is catalogued
as `*<NAME>`, **in upper case** — the global catalogue's names are upper case on
Linux — replacing any older copy; every `*` entry with no object left in
`global.bp.out` is removed. SD's own system programs in the catalogue are never
touched, because none of them starts with `*` and nothing else can make a `*`
entry. An object it cannot load is refused by name and the rest still go in. The
last line always reads `SYNC GLOBAL CATALOG DONE <n> catalogued <n> removed <n>
refused`.

**An upgrade catalogues them again for you.** It replaces the global catalogue
with the new release's, then runs `sync.global.catalog`; `global.bp.out` itself
is kept.

**Everyone else is refused**:

| | |
|---|---|
| *The global catalogue can only be changed by the SD Core server* | `sync.global.catalog`, or writing `global.bp.out`, `gcat` or the deny list, from a session that did not sign in with the global password — `admin` included |
| *The global catalogue holds the SD Core server's programs from global.bp.out and is changed only by sync.global.catalog* | `CATALOG ... GLOBAL`, a `CATALOG` name beginning `*`, `!`, `_` or `$`, or `DELETE.CATALOG` of a global entry — from any session |

**A user's own program cannot write to them either.** A BASIC program that
opens `global.bp.out` and `WRITE`s to it is refused by SD itself, with `STATUS()`
saying so, whatever the caller's rights.

**On a computer with no global password** there is no server: `sync.global.catalog`
says *this computer has no global password, so no SD Core server manages it and
there is nothing to manage*, changes nothing, and the global catalogue holds only
SD's own programs.

## Commands the user may not run: `deny.verbs`

**The server keeps a list of commands the user of the computer may not run
without the administrator or global password.** A command on the list behaves
like the administrator commands: refused with *Command requires administrator
privileges* until `admin`.

```
deny.verbs                       list them
deny.verbs add listf,create.file add to the list
deny.verbs remove listf          take one off
deny.verbs set listf,copy        replace the list
```

**Every form answers with the list as it now stands:**

```
DENY.VERBS 2: LISTF,COPY
```

**A command is denied under every name that runs it.** Denying `SH` denies `!`,
because both run the same command. Some names that look different share one:
`EDIT`, `NANO` and `MICRO` all run SD's editor program, so denying one denies all
three. The answer says what else was taken:

```
:deny.verbs add sh
deny.verbs also denies, as the same command: !
DENY.VERBS 1: SH
```

| | |
|---|---|
| **Who may use it** | a server session only. Anyone else, `admin` included, is told *The denied verbs can only be listed or changed by the SD Core server* |
| **Never denied** | `admin`, `OFF`, `QUIT` and `LO` — a list naming one says it is dropped |
| **Set at installation** | the control file's `deny-verbs=` line (or `--deny-verbs` to the stage script), on a new installation only. **On a computer with no global password no session can change the list afterwards** — only the server can — so it is fixed until a new installation |
| **Kept by an upgrade** | yes — it lives in `~/SDCoreSolo/solo.policy` |

**It only adds.** It cannot lift the check an administrator command carries in
its own code; a command already needing `admin` needs it whatever the list says.

**It is checked where every command is dispatched** — typed, run from a paragraph,
`EXECUTE`d by a program, or sent over the API — so no route skips it.

## The limit of all this

**SD enforces these rules; Linux does not.** The whole `~/SDCoreSolo` directory
belongs to the computer's Linux user, who can change any file in it from outside
SD — `global.bp.out`, the catalogue, the list of denied commands, even the kept
password. What managed mode protects is what happens inside SD. A computer whose
user must not be able to change these needs that user to lack the Linux rights to
the directory, and Solo does not set that up.

## Coming later

**Further management features are to be added to the SD Core for Linux server**,
and each will have its counterpart here. This page will say what each one does to
a managed computer as it arrives.
