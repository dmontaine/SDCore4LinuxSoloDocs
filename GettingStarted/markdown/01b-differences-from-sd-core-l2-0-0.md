Title: Differences from SD Core for Linux L2.0-0
Subtitle: What SD Core Solo for Linux leaves out, adds and does differently.

**SD Core Solo for Linux LS2.0-0 is the single-user edition of SD Core for Linux
L2.0-0**, and the language, the query processor, the file system, the editors,
the full-screen pages and nearly every command are the same. This page is for a
reader who knows the multiuser product. The User set applies to both.

## One user, one account, no root

| SD Core for Linux L2.0-0 | Solo |
|---|---|
| many accounts, one per person, made by SDSYS | **one account, `sduser`**, made by the installer. `WHO` and `@LOGNAME` say `sduser` on every computer, whatever the Linux user is called |
| SDSYS, entered by logging in to Linux as the `sdsys` user | **SDSYS is never entered.** Nobody logs in to it; the administrator commands run from your own account after `admin` |
| `logto` *account*, to switch accounts without logging out | **not in Solo**. There is one account, so there is nowhere to switch to. Typed anyway it answers *LOGTO is not in your VOC* |
| `create.account`, `delete.account`, `modify.account`, `modify.password`, `tool.accounts` | **not in Solo** |
| the Linux groups `sdusers` and one `sdu_<name>` per account | **none.** There is one Linux user, and it is yours |
| SD runs as root, then drops to the account; the root helper `sd-elevate` | **SD never runs as root** and refuses to start as root. It runs as you, and there is no helper |
| the `sdsys` OS user, `/usr/local/sdsys`, `/etc/sd.conf`, `/home/sd` | **none of them.** Everything is in `~/SDCoreSolo` |
| ssh and the API are each a per-account gate, off until SDSYS turns it on (`modify.account`) | ssh and the API reach the one account, `sduser`, after the account password; each is chosen when installing and changed with `remote.ssh` and `remote.api` |

## Passwords

| SD Core for Linux L2.0-0 | Solo |
|---|---|
| a local sign-in asks for no SD password — Linux has authenticated you | **every session asks for the account password**: at the keyboard, over ssh, and through the API |
| a command on the command line (`sd list customers`) needs no password | **`sd-solo list customers`** uses **a copy of the account password kept for you**, in a file only you can read, so scripts and scheduled jobs need no typing |
| administration is being SDSYS | **administration is `admin`** and a password set at installation |
| `modify.password`, run by SDSYS | **`set.password`**: your own account password with no `admin` (it asks the current one), `set.password admin` after `admin`, `set.password global` by the SD Core for Linux server only. It also updates the kept copy |
| `remote.sdsys`, the administrator's ssh door | **not in Solo**: there is no `sdsys` to let in |

**The kept copy is the password itself, in clear, in a file only you can
read** (mode 0600, in a 0700 directory), because Linux has nothing that lets a
job with no session unlock a secret for you. Anyone who can read your files can
read it, as they could read a `~/.pgpass`. See
[The account and its passwords](05-account-types.html).

## Administration

| SD Core for Linux L2.0-0 | Solo |
|---|---|
| the administrator verbs are SDSYS's, and only SDSYS has them | the same verbs are in your account and **need `admin` first** — including eight that had no check of their own because only SDSYS had them: `CONFIG`, `LISTU`, `LIST.LOCKS`, `LIST.READU`, `LOCK`, `CLEAR.LOCKS`, `SET.DATE`, `CLEAN.ACCOUNT` (`CONFIG GPL` and `CONFIG CONTRIB`, which the sign-on banner tells everyone to type, need no `admin`) |
| the edit gates `voc-edit`, `bp-edit` and `dict-edit`, per account | **no edit gates.** `ED VOC`, a program's `WRITE` or `DELETE` to the VOC, `COPY` into it, and saving or deleting a sentence with `.S` and `.D` **need `admin`**. What SD writes to the VOC as a side effect — `CREATE.FILE`'s entry, the command stack — does not |
| the `sh`, `os`, `net` and `ai` gates of each account, set with `modify.account` | the same switches for the one account, set with **`set.os.access`** (`sh`, `os`), **`set.net.access`** and **`set.ai gate`** after `admin`; off as delivered. See [Operating system access](06b-operating-system-access.html) |
| SDSYS can `CATALOG ... GLOBAL` | **nobody changes the global catalog**, `admin` or not. On a managed computer it holds the SD Core for Linux server's programs. See [Other hardening](13-hardening.html) |
| `remote.ssh {on \| off}` moves the firewall rule for port 22; `remote.api {on \| local \| off}` the system socket and the firewall | **`remote.ssh` and `remote.api`**, each `off`, `local` or `open`, move Solo's own socket units (ports 4251 and 4249) and never a firewall. See [ssh access](08-ssh-access.html) and [API access](09-api-access.html) |
| `update.accounts` updates every account | updates the one account, and an upgrade runs it for you |
| scheduled jobs defined by SDSYS (`set.job`, `tool.jobs`) and run as a system service | **scheduled jobs are yours**: any scheduler that can run `sd-solo` *command*. See [Scheduled jobs](04-scheduled-jobs.html) |
| `run.in` and `basic ... no.catalog`, to work in another account from SDSYS | **not in Solo**: there is no other account |
| the master files page `tool.entries`, and the account holder's gate page `tool.mygates` | **not in Solo** |
| the shared account (`shared`), its group and its documentation directory | **not in Solo.** The documentation PDFs are in `~/SDCoreSolo/pdf`, and `pdf` shows one |
| the audit trail is written by a root daemon | **you write your own audit trail**: nothing in Solo is root |

See [Administrator commands](06-administrator-commands.html).

## Only in Solo

| | |
|---|---|
| `admin` | becomes the administrator for the session, with the administrator password. See [Administrator commands](06-administrator-commands.html) |
| `set.os.access` | the account's `sh` and `os` switches. See [Operating system access](06b-operating-system-access.html) |
| `set.password` | the account, administrator and global passwords |
| `set.mcp` | the MCP door, through which an AI agent on this computer works in SD as you. See [The MCP door](25-the-mcp-door.html) |
| `pdf`, `set.viewer`, `tool.viewers` | read a documentation PDF in a reader the administrator has allowed (SD Core for Linux has no viewer: its PDFs are files to copy) |
| managed mode | a global password lets an SD Core for Linux server manage the computer (below) |

## Managed mode

**New in Solo.** A computer installed with a **global password** is *managed*: an
SD Core for Linux server also manages it and signs in with that password. The
installer lets the global password be left blank, in which case no server manages
the computer; there is no "standalone or managed" question. The server can put compiled programs into the global
catalog (`global.bp.out`, `sync.global.catalog`), keep a list of commands
the user may not run (`deny.verbs`), and administer the account's files over the
API (indices, `configure.file`, `create.file`, `delete.file`). An installer control file,
`sd-solo-setup.conf`, sets up many computers the same way, leaving the account
password to be chosen at first login. The server can also install its
own ssh key over the API, and the client library pins a server's TLS certificate
the first time it connects. See [Managed mode](15-managed-mode.html).

## Installing and running

| SD Core for Linux L2.0-0 | Solo |
|---|---|
| `installsdcore.sh`, run by a user who can `sudo`, installs for the computer | `installsdsolo.sh` installs for **one user, all in `~/SDCoreSolo`**, run as that user. `sudo` only for the build packages, a firewall rule and linger |
| the installer asks whether a session starts at the home screen or the command prompt | **no question**: a session starts at the home screen; `set.start.default terminal` (after `admin`) changes it |
| `deletesdcore.sh`, and an upgrade is uninstall-keeping-accounts then install | `deletesdsolo.sh`, and **`installsdsolo.sh --upgrade`** upgrades in place with a safety copy that is put back if anything fails |
| a system `sd.service` and `sdclient.socket` | user units: **`sd-solo.service`**, and with an API `sd-solo-api.socket`, and with ssh `sd-solo-ssh.socket`. They run as you, and stop when your last session ends unless you enabled linger |
| the system programs' BASIC source is installed | **compiled programs only**; no system source is installed |
| `sd -internal` needs `sudo` | **`sd-solo -internal` is closed** once the installer has finished: it needs a one-shot marker file the installer writes before each of its own steps |
| each account lands in `sd` over ssh on port 22 (`ForceCommand` for the group) | **Solo runs its own ssh listener on port 4251**, as you, with no `sudo` and no change to the computer's ssh server, so both products can be installed and reached together. You sign in with your Linux account name and password (a key is optional), straight into `sd-solo`; no shell, no `scp`/`sftp`, no forwarding. See [ssh access](08-ssh-access.html) |
| the API on port 4247 | the API on port **4249**, off unless chosen when installing |

See [Installing](01-installation.html) and [Running SD](03-running-sd.html).

## Smaller differences

- **The API user name is always `sduser`.**
- **`SDConnectLocal` is disabled**: a client connects with `SDConnect` and the account password.
- **The files route of the API** (the Linux groups `sdfiles`, `sdfileswrite` and `sdfilesadmin`) **is not in Solo**; API request 29 is behind the NET gate and `NETFILES`.
- **`option inherit.ownership` is not in Solo**: every file is yours already.
- **A session reading from a pipe ends at end of input**, with or without
  `OFF`, instead of waiting at the prompt.
- **The audit trail is not append-only.** The multiuser product can make it
  so with `chattr +a`, which needs root; Solo cannot, so you can edit your own
  audit trail. It is a record for you, not evidence against you.
- **The API's TLS relay cannot become `nobody`**, because nothing in Solo is
  root. The same system-call filter that SD Core puts on it after it has dropped
  its privileges is all that locks it down here: the relay can copy bytes and
  nothing else, and it is killed if it tries to open a file, make a connection
  or run a program.

## What might stop working

- **Anything that creates, grants or deletes accounts**, or signs in to more
  than one account.
- **Scripts that use `LOGTO`**, or that expect administrator verbs
  to work without `admin`.
- **A client that signs in with a Linux user name**, or with the old cleartext
  login, or with `SDConnectLocal`.
- **Anything that writes the global catalog.** Catalog programs locally
  (`CATALOG ... LOCAL`) instead.
- **Anything that expects `/usr/local/sdsys`, `/etc/sd.conf` or `/home/sd`.**
  Use `~/SDCoreSolo`.
- **`sd-solo` started by root, `sudo`, or another user.** It refuses.
