Title: Installing
Subtitle: What the installer asks, what it puts where, and the control file for installing many computers.

**The installer is one script, `installsolo.sh`, and it installs for the Linux
user who runs it.** Everything goes into that user's home directory, in
`~/SDCoreSolo`. Nothing is installed for other users of the computer, nothing
is written outside your home directory, and SD never runs as root — the script
refuses to run as root, and so does `sd`.

**There is no prebuilt package.** The script downloads the source from
`github.com/dmontaine/SDCore4LinuxSolo` (the `main` branch) into a temporary
directory, `~/.sdsolotmp`, builds it there, installs it, and deletes the
download when it ends. It can be carried on a USB stick: it needs the network
only for the build packages and that download.

## Before you start

| | |
|---|---|
| Distribution | Debian or Ubuntu based, Fedora or RHEL based, openSUSE based, or Arch based — read from `/etc/os-release`. Any other is refused in words before anything changes |
| Rights | your own ordinary user. **`sudo` is used for four things and only those:** installing the build packages, opening a firewall port you asked for, the optional `sshd_config.d` block, and `loginctl enable-linger` |
| The build tools | `git`, `make`, `gcc`, `python3` with its development headers, and `openssl`. Without `--skip-packages` the installer installs them (and `micro`, `lynx`, `libsodium` and `libssl` headers) with `sudo`; with it, it only checks they are there |
| A systemd user manager | `systemctl --user` must work. SD runs as your own systemd user service |

**The multiuser SD Core for Linux must not be installed.** If
`/usr/local/sdsys` or `/etc/sd.conf` exists the installer stops with a
message saying so: the two cannot share a computer, because both use API port
4243 and the same shared-memory name.

**It refuses to start, and changes nothing, if the directory it would install
into is not empty** and is not a Solo tree, or if a Solo tree is already
there (see [Upgrading and uninstalling](01a-upgrading-and-uninstalling.html)).

## Running it

```sh
git clone https://github.com/dmontaine/SDCore4LinuxSolo
bash SDCore4LinuxSolo/installsolo.sh
```

Or fetch just `installsolo.sh`: it clones the source it actually builds from,
so having the repository first is a convenience. **Run it as yourself.** It asks
its questions at the terminal; every one can be answered by an option instead
(`bash installsolo.sh --help` lists them), which is how it is scripted.

## What you are asked

### 1. The mode

| | |
|---|---|
| **Standalone** | a database for this computer only. The default |
| **Managed client of an SD Core server** | a computer an SD Core for Linux server also manages. See [Managed mode](15-managed-mode.html) |

**The mode cannot be changed later except by a new installation.** It decides
whether a global password exists, and nothing sets or clears that afterwards.

### 2. The passwords

Asked in this order, each typed twice, shown as stars:

| | |
|---|---|
| **Account password** | the password every SD session asks for — at the keyboard, over ssh and through the API |
| **Administrator password** | unlocks the administrator commands, with `ADMIN` |
| **Global password** | managed mode only. The SD Core for Linux server signs in with it, and it also unlocks the administrator commands |

**Every password needs at least 8 characters, with a lower-case letter, an
upper-case letter, a digit and a symbol** — letters, digits and punctuation
only, no spaces. A password that breaks the rule is asked for again, up to
three times. The global password must differ from both of the others;
otherwise the server, signing in with the same name, would land in an ordinary
session. See [The account and its passwords](05-account-types.html).

### 3. The API and ssh — standalone only

| | |
|---|---|
| **API listener** | `off` (the default), `local` (this computer only) or `open` (reachable from the network). Port 4243 unless you give `--api-port` (1024–65535: a user cannot bind lower) |
| **ssh straight into sd** | if you say yes, the installer asks for a public key file and adds it to your `~/.ssh/authorized_keys` with a forced command, so that key lands in `sd`. See [ssh access](08-ssh-access.html) |
| **The `sshd_config.d` block** | optional, needs `sudo`: makes every ssh login of your user — password too — land in `sd`. See [ssh access](08-ssh-access.html) |
| **Linger** | `loginctl enable-linger`, so SD keeps running after you sign out. Without it SD stops when your last session ends. It is a persistent setting of your account, so it is a question, not a default |

**In managed mode the API and ssh are not asked**: the API is open to the
network on the port given, and ssh is required — the server has to reach the
computer from elsewhere. If the computer has `ufw` running, the installer opens
the API port with `sudo`; otherwise it tells you to allow it yourself.

The last question is **Continue?** Answering no changes nothing.

## What lands where

Everything is under `~/SDCoreSolo`. **SDSYS is that directory itself** — there
is no `sdsys` subdirectory as there is on Windows.

| | |
|---|---|
| `bin/` | `sd`, its daemon and the other programs |
| `sd.conf` | the configuration. See [Configuration](16-configuration.html) |
| `user_accounts/sduser` | the one SD account, and your data |
| `$cred/` | the credential store: the verifiers for the three passwords, and the kept copy of the account password. Mode 700 |
| `gcat/`, `gpl.bp.out/`, `voc`, `messages/`, `syscom/`… | SD's own files: the global catalogue, the system programs (compiled only — no source is installed), the messages, SD's own VOC and dictionaries |
| `global.bp.out/`, `solo.policy/` | on a managed computer, the SD Core for Linux server's programs and the list of commands denied to you. See [Managed mode](15-managed-mode.html) |
| `audit`, `errlog` | the audit trail and the error log |
| `tools/` | `solo-service.sh`, `solo-ssh.sh` and `deletesolo.sh` — see [The installed scripts](17-the-installed-scripts.html) |
| `.sdcore-install` | which commit was installed, when, and in which mode |
| `~/.local/bin/sd` | a link to `~/SDCoreSolo/bin/sd`, so `sd` works from any new terminal (if `~/.local/bin` is on your PATH — the installer says so if it is not) |
| `~/.config/systemd/user/` | the service: `sd-solo.service`, and with an API `sd-solo-api.socket` and `sd-solo-api@.service` |

**Everything the installer creates is private to you**: files 0600, directories
0700. The directory can be moved: SD finds its own files from where its
programs are, not from a path written into it. The service units, the ssh key
line and the link name the directory by its full path, and do not move with it.

**`--home DIR`** installs somewhere else. The path must be absolute and must not
contain a space, a quote, a backslash, `$`, a backtick or `%`.

## Installing many computers: the control file

**A file passed as `--control-file` answers the installer's questions.** It is
for **managed mode only** — its presence makes the install managed — and it is
how one USB stick sets up several computers.

```sh
bash installsolo.sh --control-file /media/stick/sd-solo-setup.conf
```

| | |
|---|---|
| `admin-password=` | the administrator password |
| `global-password=` | the global password |
| `deny-verbs=` | a comma-separated list of commands the user of the computer may not run without the administrator or global password. See [Managed mode](15-managed-mode.html) |
| `ssh-public-key-file=` | a public key whose owner may ssh straight into `sd` |
| `ssh-match=yes` | also write the `sshd_config.d` block (needs `sudo`) |
| `enable-linger=yes` | run `loginctl enable-linger` |

**The account password is deliberately not in it.** On a computer installed
from a control file, the user sets the account password **the first time they
run `sd` at that computer's keyboard**. Until then ssh and the API accept only
the global password — the server can reach the computer, and nobody else can.
(Give `--account-password-file` as well if you would rather set it at install
time.)

**A blank answer is asked for**, and one that breaks the password rules is
refused. The repository carries `sd-solo-setup.conf.sample`, which explains each
item and shows a sample answer, commented out, above the line for your answer;
copy it and fill it in. **A commented sample is never taken as an answer.**

**The file holds passwords in clear text.** Keep the stick safe, and do not
leave the file on a computer after installing.

## What the installer checks when it finishes

It signs in as `sduser` and runs `WHO`, and then confirms that **`sd -internal`
is closed**: the door the installer itself uses is a one-shot, opened only by a
marker file the installer writes immediately before each of its own steps, and
it is shut when the install ends. The last line of a good install is:

```
SOLO INSTALL COMPLETE /home/you/SDCoreSolo
```

**If a step fails the installer stops and says which**, with the end of its
log. Nothing is put back — a failed first install leaves whatever it had made;
remove it with `deletesolo.sh` and start again.

## Changing any of it afterwards

| | |
|---|---|
| The mode | a new installation: uninstall (keeping your data if you want it), then install |
| The API or ssh choices | uninstall and install again, or use the scripts in `~/SDCoreSolo/tools` — see [The installed scripts](17-the-installed-scripts.html) |
| The passwords | `SET.PASSWORD`, `SET.PASSWORD ADMIN`, and on a managed computer `SET.PASSWORD GLOBAL` from the server. See [The account and its passwords](05-account-types.html) |

## Continued in

[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html) —
installing a new release over an existing one, and taking SD off the computer.
