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
| `modify.password` | **`set.password`** — your own account password with no `ADMIN`, `set.password admin` after `ADMIN`, `set.password global` from the server |
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
in each program by name — see
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

`set.server`, `delete.server` and `list.servers` have gone with it.

**The API is not affected.** `!sdclient` and the remote API are a separate
mechanism and are unchanged. See [API access](09-api-access.html).

**`NETFILES` is still accepted in `sd.conf`** and does nothing, so an existing
configuration file will not stop SD starting.

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
at the prompt. **There is no permission gate on starting it** — the same "no
second wall" reasoning that applies to `sh` and `OS.EXECUTE`; see
[Security and the operating system](12a-security-and-the-operating-system.html).

## Field-level encryption

**`encrypt.field` is gone, and with it field-level encryption from TCL.**

**Encryption in SD BASIC is unaffected and is the supported route.**
`sdencrypt()` and `sddecrypt()` ship — see *SD Basic - System and Environment* in
the User set. What has gone is the TCL verb that encrypted a field in place, and
**nothing replaces that**.

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
second**: a standalone install asks for its passwords (or takes them from files
you give it), and a managed install can be answered by a control file. See
[Installing](01-installation.html).
