Title: Differences from multiuser SD Core for Linux L1.1-1
Subtitle: What SD Core for Linux Solo leaves out, adds and does differently.

**SD Core for Linux Solo was made from the multiuser SD Core for Linux
L1.1-1**, and the language, the query processor, the file system and nearly
every command are the same. This page is for a reader who knows the multiuser
product. The User set applies to both.

## One user, one account, no root

| multiuser L1.1-1 | Solo |
|---|---|
| many accounts, one per person, made by SDSYS | **one account, `sduser`**, made by the installer. `WHO` and `@LOGNAME` say `sduser` on every computer, whatever the Linux user is called |
| SDSYS, entered by logging in to Linux as the `sdsys` user | **SDSYS is never entered.** Nobody logs in or `LOGTO`s to it; the administrator commands run from your own account |
| `create.account`, `delete.account`, `modify.account`, `grant`, `revoke`, `list.grants`, `modify.password` | **gone** |
| the Linux groups `sdusers` and one `sdu_<name>` per account; `usermod -aG` as the grant | **gone.** There is one Linux user, and it is yours |
| SD runs as root, then drops to the account | **SD never runs as root** and refuses to start as root. It runs as you |
| `sudo` helpers, the `sdsys` OS user, `/usr/local/sdsys`, `/etc/sd.conf`, `/home/sd` | **none of them.** Everything is in `~/SDCoreSolo` |
| ssh and the API are open to every account except SDSYS | ssh and the API reach the one account, `sduser`, after the account password |

## Passwords

| multiuser L1.1-1 | Solo |
|---|---|
| a local sign-in asks for no SD password — Linux has authenticated you | **every session asks for the account password**: at the keyboard, over ssh, and through the API |
| a command on the command line (`sd list customers`) needs no password | **`sd-solo list customers`** uses **a copy of the account password kept for you**, in a file only you can read, so scripts and scheduled jobs need no typing |
| administration is being SDSYS | **administration is `ADMIN`** and a password set at installation |
| `modify.password`, run by SDSYS | **`SET.PASSWORD`**: your own account password with no `ADMIN` (it asks the current one), `SET.PASSWORD ADMIN` after `ADMIN`, `SET.PASSWORD GLOBAL` by the SD Core for Linux server only. It also updates the kept copy |

**The kept copy is the password itself, in clear, in a file only you can
read** (mode 0600, in a 0700 directory), because Linux has nothing that lets a
job with no session unlock a secret for you. Anyone who can read your files can
read it, as they could read a `~/.pgpass`. See
[The account and its passwords](05-account-types.html).

## Administration

| multiuser L1.1-1 | Solo |
|---|---|
| the administrator verbs are SDSYS's, and only SDSYS has them | the same verbs are in your account and **need `ADMIN` first** — including eight that had no check of their own because only SDSYS had them: `CONFIG`, `LISTU`, `LIST.LOCKS`, `LIST.READU`, `LOCK`, `CLEAR.LOCKS`, `SET.DATE`, `CLEAN.ACCOUNT` (`CONFIG GPL` and `CONFIG CONTRIB`, which the sign-on banner tells everyone to type, need no `ADMIN`) |
| editing the VOC directly is any account's own business | `ED VOC`, a program's `WRITE` or `DELETE` to the VOC, `COPY` into it, and saving or deleting a sentence with `.S` and `.D` **need `ADMIN`**. What SD writes to the VOC as a side effect — `CREATE.FILE`'s entry, the command stack — does not |
| SDSYS can `CATALOG ... GLOBAL` | **nobody changes the global catalogue**, `ADMIN` or not. On a managed computer it holds the SD Core for Linux server's programs. See [Other hardening](13-hardening.html) |
| `remote.ssh`, `remote.api` | **gone.** The API and ssh are chosen when installing, and changed with the scripts in `~/SDCoreSolo/tools` |
| `update.accounts` updates every account | updates the one account, and an upgrade runs it for you |

See [Administrator commands](06-administrator-commands.html).

## Managed mode

**New in Solo.** A computer installed in managed mode is also managed by an SD
Core for Linux server, which signs in with a **global password** set when the
computer was installed. The server can put compiled programs into the global
catalogue (`GLOBAL.BP.OUT`, `SYNC.GLOBAL.CATALOG`) and keep a list of commands
the user may not run (`DENY.VERBS`). An installer control file,
`sd-solo-setup.conf`, sets up many computers the same way, leaving the account
password to be chosen at first login. From LS1.1-2 the server can also install its
own ssh key over the API, and the client library pins a server's TLS certificate
the first time it connects. See [Managed mode](15-managed-mode.html).

## Installing and running

| multiuser L1.1-1 | Solo |
|---|---|
| `installsdcore.sh`, run by a user who can `sudo`, installs for the computer | `installsdsolo.sh` installs for **one user, all in `~/SDCoreSolo`**, run as that user. `sudo` only for the build packages, a firewall rule, the optional `sshd_config.d` block and linger |
| `deletesdcore.sh`, and an upgrade is uninstall-keeping-accounts then install | `deletesdsolo.sh`, and **`installsdsolo.sh --upgrade`** upgrades in place with a safety copy that is put back if anything fails |
| a system `sd.service` and `sdclient.socket` | user units: **`sd-solo.service`**, and with an API `sd-solo-api.socket`. They run as you, and stop when your last session ends unless you enabled linger |
| the system programs' BASIC source is installed | **compiled programs only**; no system source is installed |
| `sd -internal` needs `sudo` | **`sd-solo -internal` is closed** once the installer has finished: it needs a one-shot marker file the installer writes before each of its own steps |
| each account lands in `sd` over ssh (`ForceCommand` for the group) | **a key line the installer adds to your `authorized_keys` forces `sd-solo`**; a `sshd_config.d` block for password logins is optional and needs `sudo`. There is no shell over ssh for that key, and `scp`/`sftp` to it do not work |

See [Installing](01-installation.html) and [Running SD](03-running-sd.html).

## Smaller differences

- **The API user name is always `sduser`.**
- **A session reading from a pipe ends at end of input**, with or without
  `OFF`, instead of waiting at the prompt.
- **The audit trail is not append-only.** The multiuser product could make it
  so with `chattr +a`, which needs root; Solo cannot, so you can edit your own
  audit trail. It is a record for you, not evidence against you.
- **The API's TLS relay cannot become `nobody`**, because nothing in Solo is
  root. Instead a system-call filter locks it down before it touches the
  network: it can copy bytes and nothing else, and it is killed if it tries to
  open a file, make a connection or run a program.

## What might stop working

- **Anything that creates, grants or deletes accounts**, or signs in to more
  than one account.
- **Scripts that `LOGTO SDSYS`**, or that expect administrator verbs to work
  without `ADMIN`.
- **A client that signs in with a Linux user name**, or with the old cleartext
  login.
- **Anything that writes the global catalogue.** Catalogue programs locally
  (`CATALOG ... LOCAL`) instead.
- **Anything that expects `/usr/local/sdsys`, `/etc/sd.conf` or `/home/sd`.**
  Use `~/SDCoreSolo`.
- **`sd-solo` started by root, `sudo`, or another user.** It refuses.
