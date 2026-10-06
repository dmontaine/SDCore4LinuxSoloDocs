Title: The account and its passwords
Subtitle: sduser, the account password, the kept copy, and the administrator and global passwords.

## One account: `sduser`

**SD Core for Linux Solo has one SD account, and it is always called
`sduser`** — on every computer, whatever your Linux user is called. The
installer makes it, in `~/SDCoreSolo/user_accounts/sduser`. `WHO`, `@LOGNAME`
and the audit trail all say `sduser`, and so does the user name an API client
signs in with.

**There is no way to make another account**, and none is needed: every
session — at the keyboard, over ssh, through the API, or a command from a
script — lands in `sduser`. SD's own system account, SDSYS, exists but is never
entered: `sd-solo -asdsys` is refused, and the administrator commands are in your
own account behind `ADMIN`.

## Three passwords

| | Set | Asked | Unlocks |
|---|---|---|---|
| **Account password** | at installation, or at the first `sd-solo` on a computer installed from a control file; changed with `SET.PASSWORD` | by every session | the account |
| **Administrator password** | at installation; changed with `SET.PASSWORD ADMIN` | by `ADMIN` | the administrator commands, for the rest of the session |
| **Global password** | at installation, managed mode only; changed with `SET.PASSWORD GLOBAL` by the server | by `ADMIN`, and by any session in place of the account password | the account **and** the administrator commands. It is the SD Core for Linux server's |

**Every one needs at least 8 characters, with a lower-case letter, an
upper-case letter, a digit and a symbol** — letters, digits and punctuation
only. **The global password must differ from both of the others.** One account
name carries both the account password and the global password, and the
account password is tried first — so if they were the same, the server would
land in an ordinary session.

## The account password

**Every session asks for it**, and is refused without it:

| | |
|---|---|
| `sd-solo` at a terminal | `Password:`, three tries, then the session ends |
| `sd-solo` with its input piped | the first line of the input, one try |
| ssh | the same as a terminal, after Solo's own ssh listener (port 4251) has authenticated you by a key in Solo's key file |
| the API | the client library's password, checked by SCRAM — see [API access](09-api-access.html) |
| `sd-solo <command>` | the kept copy, below — no typing |

**A wrong one is answered `Wrong password`.** On a managed computer the global
password is accepted in its place, and that session also has the administrator
commands unlocked.

**Being signed in to Linux is not enough.** The password is the gate; anyone
who has it can use the account, and a copied `~/SDCoreSolo` directory works for
whoever knows it.

### The kept copy

**SD keeps a copy of the account password for you**, in
`~/SDCoreSolo/$cred/$stored`. A command on the `sd-solo` command line signs in with
it, which is what lets scripts and scheduled jobs use SD — see
[Scheduled jobs](04-scheduled-jobs.html). The installer writes it, and
`SET.PASSWORD` updates it.

**It is the password itself, in clear.** Linux has nothing that lets a job with
no session unlock a secret for you, so SD cannot encrypt it in a way the job
could then open. The file is mode 0600 in a 0700 directory, so only your Linux
user can read it — anyone who can read your files can read it, as they could
read a `~/.pgpass`. Removing the file makes `sd-solo <command>` ask for the password
on its input instead; the account still works.

**It proves the account password only.** It never unlocks the administrator
commands.

### Changing it: `SET.PASSWORD`

```
:set.password
Current password:
New password:
Confirm the new password:
Password changed
```

**You can always change your own account password**; it needs no `ADMIN`. It
asks for the current one first, so a session left open cannot be taken over by
changing it — unless you have typed `ADMIN`, or signed in with the global
password, in which case it goes straight to the new one. A wrong current
password is answered *Wrong password - the password is unchanged*.

The new password must meet the rules above and differ from the global
password; otherwise it says why and leaves the password as it was.

**`SET.PASSWORD` also updates the kept copy.** If it cannot, it says *The new
password could not be kept for commands given on the sd command line* — the
password is changed, but commands on the `sd-solo` command line will fail until it
is set again.

### The first password on a managed computer

**A computer installed from a control file has no account password yet.** The
first `sd-solo` typed at that computer's keyboard asks you to choose one:

```
This account has no password yet. Choose one now - SD Core for Linux Solo asks for it every time it is used.
```

You get three tries; each of a weak password, one equal to the global
password, and a confirmation that does not match is refused and counted. An
empty answer gives up, and nothing is set.

**Only the console offers it.** A session over ssh, through the API, or a
command on the `sd-solo` command line is never asked to choose — otherwise whoever
reached the computer first would own the account. **Until the password is set,
only the global password is accepted** there, and the answer is:

```
This account has no password yet. Set it at this computer's keyboard first; until then only the global password is accepted.
```

So the SD Core for Linux server can reach a computer it has just set up, and
nobody else can. Choosing the password also keeps the copy for `sd-solo <command>`
and gives the account the global password's salt, so the server still signs in
afterwards.

## The administrator password

**It unlocks the administrator commands for one session**: type `ADMIN`, then
the password. See [Administrator commands](06-administrator-commands.html).

**Change it with `SET.PASSWORD ADMIN`, after `ADMIN`:**

```
:admin
Administrator password:
Administrator commands unlocked for this session
:set.password admin
New password:
Confirm the new password:
Password changed
```

It must differ from the global password.

## The global password

**Optional: only a managed computer has one** (the installer lets you leave it
blank). The SD Core for Linux server signs in as `sduser` with
it, and every such session has the administrator commands unlocked. A few
commands need it and refuse the administrator password — the ones that are the
server's rather than the user's. See [Managed mode](15-managed-mode.html).

**Only a session signed in with the global password can change it**, with
`SET.PASSWORD GLOBAL`, and **such a session can change all three**. Anyone else
— `ADMIN` included — is told *The global password can only be changed by the SD
Core server*. It must differ from the account and administrator passwords.

**Whether there is one is fixed at installation**: nothing creates or removes
one afterwards. On a computer with none, `SET.PASSWORD GLOBAL` says *This
computer has no global password - no SD Core server manages it*.

## Who can change which

| | account | administrator | global |
|---|---|---|---|
| a session with the account password | yes, after the current one | after `ADMIN` | no |
| a session signed in with the global password | yes | yes | yes |

**The global password must differ from the other two.** A change that would
make it equal one of them is refused, and the password is left as it was. The
account and administrator passwords may be the same.
