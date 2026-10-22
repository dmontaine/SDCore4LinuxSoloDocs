Title: Not in SD Core
Subtitle: What has been removed, why, and what to use instead.

This page exists so you do not spend time hunting for something that is not
there. **It names what is gone and what to use in its place; it does not document
the removed features themselves.**

Everything here was in OpenQM, in ScarletDME, or in upstream `sdb64`, or in the
multiuser SD Core for Linux this release was made from, and is not in SD Core for
Linux Solo. **Most of the removals before Solo were made independently on SD Core
for Windows too, for the same reasons.**

> **If you had a use for any of these, say so.** Several were removed on the
> reasoning that nothing needed them. That reasoning is worth testing against real
> use.

## Accounts, groups and grants — removed for Solo

| Gone | Use instead |
|---|---|
| `create.account`, `delete.account`, `modify.account` | nothing: there is one account, `sduser`, made by the installer |
| `grant`, `revoke`, `list.grants` | nothing: there is one Linux user and no groups to join |
| `modify.password` | **`set.password`** — your own account password with no `admin`, `set.password admin` after `admin`, `set.password global` from the server |
| `remote.ssh`, `remote.api` | the installer's choices, changed with `tools/solo-service.sh` and `tools/solo-ssh.sh` — see [ssh access](08-ssh-access.html) and [API access](09-api-access.html) |
| `sdsys` as a Linux user, `sdusers` and `sdu_<name>` groups, `sd-elevate` | none of them. SD runs as you, and never as root |
| `batch.jobs` | none: a scheduled job may run any command — see [Scheduled jobs](04-scheduled-jobs.html) |
| `ssh`/`api`/`both`/`none` route keywords, `sh-on`/`sh-off`/`os-on`/`os-off` | gone earlier, with the `os.users` permit list they set |
| the account tiers (Standard, Programmer, Administrator) | gone earlier: one VOC for everybody |
| Group accounts, suspending an account | there is nobody to suspend |

## Editors

| Gone | Use instead |
|---|---|
| `sed` — the full-screen editor | **`nano`** or **`micro`**, or **`ed`**/**`edit`** for the line editor |
| `update.record` — the full-screen record editor | **`nano`** or **`micro`** |
| `modify` — the full-screen record editor from OpenQM | **`nano`** or **`micro`** |

**SD Core's own full-screen editing is `nano` and `micro`**, which open the record
in each program by name and, because they run outside SD, need the `os` switch
(off as delivered; the administrator turns it on with `set.os.access os on`) — see
[Development and file commands](07-programmer-commands.html#editors), which also
says what they are good for and what they cannot do. **Unlike SD Core for Windows,
`edit` is not one of them here — it aliases `ed`, the line editor, because there
is no Linux program to alias it to the way Microsoft Edit serves that role
there.** The three removed programs above are gone as *programs*; the capability
is not.

`modify` is not in SD Core at all.

> **`ed`** was never affected by any full-screen keyboard fault — it reads whole
> lines and goes through the command-line editor.

## The PROC language

`PROC` is gone. **A VOC item of type `PQ` now reports that PROC is not supported
instead of running.** Your `PQ` records are left alone — it is the interpreter
that has gone, not the records.

**Do not confuse PROC with the query processor.** `list`, `count`, `select` and
`sort` are unaffected. They are a different thing despite the similar name.

## SDNet — remote file access

SD could open a file held on another SD server by putting `server;file` in a VOC
entry. **That is gone. A VOC entry containing a semicolon is now simply a file
name that does not resolve.**

`set.server`, `delete.server` and `list.servers` went with it, **and came back on 20
October 2026 in a new form**: an SDNet **client** that reaches another SD server over its
API (TLS 1.3 and a SCRAM login), through a VOC pointer of type `N`, with the administrator
defining each server (`set.server`, pinned to the hash of the server's public key), each
person keeping their own login on it (`set.netlogin`, kept as values derived from the
password, never the password) and the NET gate (`set.net.access on`) switched on. **All of
it is off as delivered.** See the SDNet page of the User set. Nothing from the old
version is back: no `;` in a path, no port 4245, no password in any file.

**The API is not affected.** `!sdclient` and the remote API are a separate
mechanism and are unchanged. See [API access](09-api-access.html).

**One door of SDNet is still in the server: API request 29**, which opens an
`account;file` file for an API client. **`NETFILES` in `sd.conf` is what it is
behind**: bit 2 of it opens the request, and it is 0 as delivered, so the request
is refused. With the bit set, it still refuses unless the account's NET gate is
on (`set.net.access on`, an administrator command, off as delivered), and it
refuses under `SDCLIENT`, an account named by a path, another account, SDSYS and a
suspended account, and (since 21 October 2026) a file the VOC reaches through a
pointer that leads out of the account, unless the account has the `os` switch on.
See [API access](09-api-access.html).

## Virtual file systems

**SD has never been able to open a virtual file system.** Nothing in the
file-opening code ever recognised a `VFS:` pathname. What the language carried was
the *outline* of one, and none of it could be reached.

All of it has been removed, so the language no longer offers a feature it cannot
perform. **These names are no longer defined, and a program mentioning one will no
longer compile:**

```
FL$TYPE.VFS      SYSCOM KEYS.H
ER$VFS.NAME      SYSCOM ERR.H
ER$VFS.CLASS     SYSCOM ERR.H
ER$VFS.NGLBL     SYSCOM ERR.H
```

`ftype` no longer returns `VFS` for a `VFS:` pathname. **If one of your programs
refers to any of these, it was testing for a state SD could not reach, and the
test can be deleted.**

## Tape and restore

The `TAPE`/`RESTORE` subsystem is gone, along with the assumption of a tape-backed
sequential medium it was built around. Back up and restore SD data the ordinary
Linux way — at the file level, with the daemon stopped (`sd-solo -stop`), or through
your own export/import BASIC.

## Language and locale

**SD Core is English only.** `set.language` and `load.language` do not exist.
`nls` is still there: it shows and sets the currency symbol and the thousands and
decimal separators, which is all it ever did.

## Embedded Python — kept

**Unlike SD Core for Windows, which dropped it and only later brought back a
narrower, process-isolated form, this port never removed embedded Python.** It
links directly against the interpreter, and there is no separate helper process to
talk to over a pipe. **The installer needs the Python development headers to build
it.**

**BASIC-callable programs** (`call !py_createdict`, and so on) are the interface —
there is no TCL verb, so this is a programming capability, not a command you type
at the prompt. **Starting it, and running code in it, needs the `os` switch** (off
as delivered): Python runs arbitrary code in the SD process, so it is held to the
same grant as `OS.EXECUTE`; see
[Operating system access](06b-operating-system-access.html) and
[Security and the operating system](12a-security-and-the-operating-system.html).

## Field-level encryption

**`encrypt.field` is gone, and with it the TCL verb that encrypted a field in
place.** What replaces it is the application's own: `sdencrypt()` and `sddecrypt()`
with a key that `sdkey()` makes from a passphrase, and the `XE` conversion, which
shows an encrypted field decrypted to a session that holds the key. SD does not
keep the key. See *Encryption and the SDEXT interface* in this set and *SD Basic -
Encrypting Data* in the User set.

## Configuration items

| Gone | Notes |
|---|---|
| `CREATUSR` | `config` no longer lists it; a `CREATUSR` line in `sd.conf` is still accepted and ignored |
| `APPEND.SD.PATH` | never existed on Linux: the installer links `sd-solo` into `~/.local/bin` |

**`umask` is kept, deliberately, and is not on this list** — a real difference
from SD Core for Windows, where it was removed as inert. It is a live mechanism
here.

## SDSYS test and legacy programs

`bigstr_test`, `msgtest`, `pcl`, `pcl.grid`, `pcode_list`, `sdtest_v8`,
`sd_encrypt`/`_b64`/`_ext`, `sd_ext`, `test.then.else`, `testsz`, `u0032`,
`u50bb`, `vfs.cls`, `pref_t`, `sdTests` and `tilde_test` are no longer installed
— developer-era test and demonstration programs with no product function, not
something an application ever called. **`py_json`, `py_term`, `py_test` and
`py_test2` were kept deliberately**, as the documented examples for embedded
Python.

**`pcl` is unaffected as a printer feature** — the `pcl` keyword and the
catalogued `pcl` routine are both still there.

## Things that were never features, and are not coming

These are not removals. They are stated here because a reader coming from another
MultiValue system will otherwise assume they exist.

**`scp` and `sftp` do not work inbound over an ssh key that lands in `sd-solo`**, which
is the accepted cost of that landing. See [ssh access](08-ssh-access.html).

**The cleartext API login is gone**, and a client that still sends a password in
clear is refused outright. So is `SDConnectLocal`, which sent none. See
[API access](09-api-access.html).

**There is no unattended install of the first kind and no silent one of the
second**: an install asks for its passwords (or takes them from files you give
it), and a control file can answer every other question, including the API and
ssh choices, whether or not the computer is managed. See
[Installing](01-installation.html).
