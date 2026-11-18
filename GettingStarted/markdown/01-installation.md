Title: Installing
Subtitle: What the installer asks, what it puts where, and the control file for installing many computers.

**The installer is one script, `installsdsolo.sh`, and it installs for the Linux
user who runs it.** Everything goes into that user's home directory, in
`~/SDCoreSolo`. Nothing is installed for other users of the computer, nothing
is written outside your home directory, and SD never runs as root — the script
refuses to run as root, and so does `sd-solo`.

**There is no prebuilt package.** SD is built from source during the install.
Run from a clone of the repository, the script downloads the source from
`github.com/dmontaine/SDCore4LinuxSolo` (the `main` branch) into a temporary
directory, `~/.sdsolotmp`, builds it there, installs it, and deletes the
download when it ends.

**From a release zip.** The zip carries the installer, the documentation and
Solo's complete source (its `source` folder, with a `SHA256SUMS` list of every
file in it), and SD is built from that folder: nothing is downloaded from GitHub.
Before it asks anything, the installer checks every file of `source` against the
list and stops with a message if one is changed, missing or not listed;
download the zip again and unpack it into an empty folder. A zip made from a
release candidate is named a pre-release when the install starts and ends. The
zip can be unzipped on a USB stick and the script run from there, with
`bash /media/<you>/<stick>/installsdsolo.sh` — use `bash`, because a stick
formatted FAT, exFAT or NTFS has no execute permission. The computer must still
be online for the build packages; the script checks first, before it asks
anything or changes anything, and stops with a message if it is not. It writes
nothing to the stick.

**What else it puts in place.** From the source, the installer copies the guide
and the knowledge pack for AI agents into the installation's `agent-pack` folder
(only when the pack is for the release being installed). It also puts the
documentation pages into `docs/user`, where the `ai` verb looks for the passages
it sends with your question: from a release zip they come from its `source/docs`
folder, otherwise they are downloaded from
`github.com/dmontaine/SDCore4LinuxSoloDocs` (the same branch). If the changes document
(`sd-core-solo-for-linux-<version>-changes.pdf`, `.html`, `.md`) or manuals in PDF
sit beside the installer, they are copied into the installation's `pdf` folder,
and the manuals are made available to the `pdf` verb. **None of these stops the
install**: a pack for another release, a page download that fails or a file that
is not there is a note on the screen, and the installation is complete without it.

## Before you start

| | |
|---|---|
| Distribution | Debian or Ubuntu based (Linux Mint, Pop!_OS and the like) — read from `/etc/os-release`. Fedora, RHEL and its clones (Rocky, Alma), Arch and openSUSE are not supported. Any other, and those, are refused in words before the installer checks the network, asks a question or uses `sudo` |
| Rights | your own ordinary user. **`sudo` is used for these things and only those:** installing the build packages, opening a firewall port you asked for (the API's or ssh's), `loginctl enable-linger`, and — only where SELinux is on and ssh is on — writing Solo's own sign-in rules, `/etc/pam.d/sd-solo-ssh-<you>` (the computer's own rules need root at one step that Solo's ssh server, running as you, cannot pass; OpenSSH 10.0 or later) |
| The build tools | `git`, `make`, `gcc`, `python3` with its development headers, and `openssl`. Without `--skip-packages` the installer installs them (and `micro`, `lynx`, `libsodium` and `libssl` headers, the console mouse daemon `gpm`, and the ssh server `openssh-server` if ssh is on and the computer has none) with `sudo`; with it, it only checks they are there |
| A systemd user manager | `systemctl --user` must work. SD runs as your own systemd user service |

**SD Core for Linux (the multi-user product) may be installed on the same
computer.** The two have separate API ports (4247 for SD Core, 4249 for Solo)
and separate shared-memory keys. Type `sd-solo` to start Solo, which always
starts Solo.

**It refuses to start, and changes nothing, if the directory it would install
into is not empty** and is not a Solo tree, or if a Solo tree is already
there (see [Upgrading and uninstalling](01a-upgrading-and-uninstalling.html)).

## Running it

```sh
git clone https://github.com/dmontaine/SDCore4LinuxSolo
bash SDCore4LinuxSolo/installsdsolo.sh
```

Or fetch just `installsdsolo.sh`: it clones the source it actually builds from,
so having the repository first is a convenience. **Run it as yourself.** It asks
its questions at the terminal; every one can be answered by an option instead
(`bash installsdsolo.sh --help` lists them), which is how it is scripted.

## What you are asked

There is **no mode to choose**: one install serves a computer on its own and a
computer an SD Core for Linux server manages (a computer is managed if, and
only if, it has a global password).

### 1. The passwords

Asked in this order, each typed twice, shown as stars:

| | |
|---|---|
| **Account password** | the password every SD session asks for — at the keyboard, over ssh and through the API |
| **Administrator password** | unlocks the administrator commands, with `admin` |
| **Global password** | **optional — leave it blank if no SD Core server manages this computer.** The SD Core for Linux server signs in with it, and it also unlocks the administrator commands |

**A computer is managed if and only if it has a global password.** Leave it
blank and no SD Core server manages the computer: it is a database for this
computer alone, and there is nothing else to choose. Enter one and the computer
is managed by an SD Core for Linux server — see [Managed mode](15-managed-mode.html).

**Whether the computer has a global password cannot be changed later except by
a new installation**: no command creates or removes one, and only a session the
server signs in with can change it (`set.password global`). To add one to a
computer that has none, install again as described under *Changing any of it
afterwards* on [Installing many computers](01c-installing-many-computers.html).
An upgrade never adds, changes or removes it. The installer's "Ready to install"
summary says *global pw: none - no SD Core server manages this computer*, or *set*.

**Every password needs at least 8 characters, with a lower-case letter, an
upper-case letter, a digit and a symbol** — letters, digits and punctuation
only, no spaces. A password that breaks the rule is asked for again, up to
three times (the global one can still be left blank). The global password, when
there is one, must differ from both of the others; otherwise the server, signing
in with the same name, would land in an ordinary session. See [The account and its
passwords](05-account-types.html).

### 2. The API and ssh

**These are your choice with or without a global password.** The installer asks
SD Core's two questions, each answered no unless you type `y`:

```
Allow API access from other computers? opens TCP port 4249 (y/N)
Allow ssh access from other computers? opens TCP port 4251 (y/N)
```

| | |
|---|---|
| **API** | **no** (the default): the API is **off**, with no listener at all. **y**: it listens for other computers on port 4249 (fixed). `--api off`, `--api local` (this computer only) or `--api open` on the command line answers it without asking; afterwards `remote.api` changes it |
| **ssh straight into sd-solo** | **no** (the default): on for **this computer only**, and the ssh server package is installed with `sudo` if the computer does not have it. **y**: reachable from other computers, on Solo's own port 4251 (fixed). `--ssh off` turns it off and declines the package, and with no ssh server program and `--skip-packages` it is off and not asked; afterwards `remote.ssh` changes it. You sign in with your Linux account name and password, and SD then asks the account password; a key is an optional extra: if ssh is on, the installer asks for a public key file and adds it to Solo's own key file, and `--ssh-key FILE` turns it on. See [ssh access](08-ssh-access.html) |
| **Linger** | `loginctl enable-linger`, so SD keeps running, and reachable over the API and ssh, when nobody is signed in at the computer. **On by default**, for an unattended install too; on a terminal the installer asks, with yes as the answer Enter gives. `--no-linger`, or `enable-linger=no` in the control file, leaves it off, and then SD stops when your last session ends. Linger keeps SD and the ports you opened running while you are signed out; it is a persistent setting of your account, which `loginctl disable-linger` turns off later |

**A computer that an SD Core server manages needs both `open`**: the server has to
reach it from elsewhere (the API, and ssh to install its key). If you give the
installer a global password and choose less, it warns you. When a port is `open`
and the computer has `ufw` running, or firewalld, the installer opens it
with `sudo`; otherwise it tells you to allow it yourself.

### 3. The shell and OS access

**Both are off unless you turn them on**: `sh` runs Linux commands, and
OS access is `os.execute` in a program, the embedded Python, `micro`, and files
and directories outside the account by pathname. Either lets whoever signs in do
what your Linux user can do, so the installer asks two questions, each answered
no unless you say yes:

| | |
|---|---|
| **Turn on the shell (SH)** | the `sh` switch |
| **Turn on OS access (`os.execute`, outside files, Python, `micro`)** | the `os` switch |

`--os-access none|sh|os|both` answers both without a terminal. With no terminal,
or `--yes` and no `--os-access`, nothing is turned on. The administrator can turn
either on or off later with `set.os.access`; a change is seen at the next sign-on.
See [Operating system access](06b-operating-system-access.html).

### 4. Where sessions start

On a new installation only, one question: **Start new sessions at the home
screen or at the command prompt? (H/p)**. The answer Enter gives is the **home
screen**, the full-screen menu of tiles, as delivered; `p` starts a session at
the command prompt. `--start menu` or `--start terminal` answers it without a
terminal, and with no terminal, or `--yes` and no `--start`, the home screen is
kept. **An upgrade does not ask**: it keeps the start the installation has, and an
installation from before the home screen keeps starting at the prompt. The
administrator can change it later with `set.start.default menu` or
`set.start.default terminal`, or on the Starts in row of `tool.menus`.

### 5. The PDF reader

The `pdf` command shows a document of the documentation in a PDF reader, even for
an account that may not run programs, so it starts only a reader the
administrator has allowed, and **none is allowed as delivered**. When the computer
has a reader SD knows (`evince`, `okular`, `zathura`, `atril`, `xreader`,
`mupdf`, `qpdfview` or `xpdf`, the first found in `/usr/bin`, `/usr/local/bin` or
`/bin`) and none is allowed yet, the installer asks **Allow *reader* to show the
documentation? (y/N)**; the answer Enter gives is no. `--viewer NAME` allows that
reader, and `--viewer none` allows none, without a terminal; with no terminal, or
`--yes` and no `--viewer`, nothing is allowed. Allowing a reader runs
`set.viewer add pdf NAME`, which tests the program and refuses one that fails. An
upgrade keeps the reader the administrator chose. Later: `admin`, then
`set.viewer`, or the Bypass programs page.

**Allowing a reader is a loosening:** that one program may then be started for an
account whose `os` switch is off, and a graphical reader can open other files you
may read and follow links. Allow only a reader you trust. See
[Operating system access](06b-operating-system-access.html).

The last question is **Continue?** Answering no changes nothing.

## What lands where

Everything is under `~/SDCoreSolo`. **SDSYS is that directory itself** — there
is no `sdsys` subdirectory as there is on Windows.

| | |
|---|---|
| `bin/` | `sd-solo`, its daemon and the other programs |
| `sd.conf` | the configuration. See [Configuration](16-configuration.html) |
| `user_accounts/sduser` | the one SD account, and your data |
| `$cred/` | the credential store: the verifiers for the three passwords, and the kept copy of the account password. Mode 700 |
| `gcat/`, `gpl.bp.out/`, `voc`, `messages/`, `syscom/`… | SD's own files: the global catalog, the system programs (compiled only — no source is installed), the messages, SD's own VOC and dictionaries |
| `global.bp.out/`, `solo.policy/` | on a managed computer, the SD Core for Linux server's programs and the list of commands denied to you. See [Managed mode](15-managed-mode.html) |
| `audit`, `errlog` | the audit trail and the error log |
| `tools/` | `solo-service.sh`, `solo-ssh.sh` and `deletesdsolo.sh` — see [The installed scripts](17-the-installed-scripts.html) |
| `.sdcore-install` | which commit was installed, when, and whether the computer is `managed` or `unmanaged` (it has a global password, or not). An older tree may say `standalone`, which is read as `unmanaged` |
| `~/.local/bin/sd-solo` | a link to `~/SDCoreSolo/bin/sd-solo`, so `sd-solo` works from any new terminal (if `~/.local/bin` is on your PATH — the installer says so if it is not) |
| `~/SDCoreSolo/sshd/` | only with ssh on: the generated `sshd_config`, Solo's own ssh host key and its key file. See [ssh access](08-ssh-access.html) |
| `~/.config/systemd/user/` | the service: `sd-solo.service`, and with an API `sd-solo-api.socket` and `sd-solo-api@.service`, and with ssh `sd-solo-ssh.socket` and `sd-solo-ssh@.service` |

**Everything the installer creates is private to you**: files 0600, directories
0700. The directory can be moved: SD finds its own files from where its
programs are, not from a path written into it. The service units, the ssh
configuration and the link name the directory by its full path, and do not move with it.

**`--home DIR`** installs somewhere else. The path must be absolute and must not
contain a space, a quote, a backslash, `$`, a backtick or `%`.

## Continued in

[Installing Many Computers](01c-installing-many-computers.html) — the control
file for installing many computers, what the installer checks when it
finishes, the full-screen editors, the mouse on the console, and changing any
of it afterwards
