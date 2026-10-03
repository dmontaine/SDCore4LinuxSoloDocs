Title: The installed scripts
Subtitle: The scripts in the installed directory, what each does, and what their exit codes mean.

**Solo ships three scripts into the installed directory, in `~/SDCoreSolo/tools`,
and one that you run from the source repository.** They are there for two
reasons: so that a step which failed during the installation can be run again
without reinstalling, and so that a choice made at install time can be changed
afterwards.

**They are shell scripts, not SD verbs.** Nothing here is typed at an `sd-solo`
prompt. **None of them needs `root`, and each refuses to run as root.** Where
something does need `sudo` — linger, removing an old `sshd_config.d` block — the script says
so and, where it can, prints the one command to run.

*Italics* mark something you supply, **bold** a word typed as it stands, and
braces an optional part.

## Exit codes

| | |
|---|---|
| **0** | done. That includes *"it was already done"* — the scripts are written to be run twice |
| **1** | a step failed. The line above says which |
| **2** | **refused to start.** Nothing was changed. The line above names why, as `REFUSED: <reason>` |

Every script ends its output with a line naming what it did, which is what a
caller should look for and not the exit code alone: `SOLO SERVICE READY …`,
`SOLO SERVICE REMOVED`, `SOLO DELETE COMPLETE …`, and the installer's
`SOLO INSTALL COMPLETE …` or `SOLO UPGRADE COMPLETE …`.

## `installsdsolo.sh` — in the source repository

```sh
bash installsdsolo.sh [options]
bash installsdsolo.sh --upgrade [--home DIR]
```

Installs, or upgrades. **It is not copied into the installed directory**: it
downloads the source, and the source contains it. Its options are listed by
`bash installsdsolo.sh --help` and described on
[Installing](01-installation.html) and
[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html).

## `solo-service.sh`

```sh
bash ~/SDCoreSolo/tools/solo-service.sh install ~/SDCoreSolo [--api off|local|open] [--ssh off|local|open] [--enable-linger]
bash ~/SDCoreSolo/tools/solo-service.sh ssh ~/SDCoreSolo off|local|open
bash ~/SDCoreSolo/tools/solo-service.sh remove
bash ~/SDCoreSolo/tools/solo-service.sh status
```

| | |
|---|---|
| `install` | writes the **user** units into `~/.config/systemd/user` — `sd-solo.service`, with an API `sd-solo-api.socket` and `sd-solo-api@.service`, and with ssh `sd-solo-ssh.socket` and `sd-solo-ssh@.service` — naming the tree by its full path; enables and starts them. **Running it again with a different `--api` or `--ssh` changes it**: the old listener is stopped first. Ends `SOLO SERVICE READY daemon=<state> api=<off\|local\|open> ssh=<off\|local\|open> linger=<yes\|no>` |
| `ssh` | writes or removes **only** the two ssh units, leaving the daemon and the API as they are. It needs the ssh directory that `solo-ssh.sh setup` makes. Ends `SOLO SSH LISTENER <mode>` |
| `remove` | stops SD and removes the units. Ends `SOLO SERVICE REMOVED` |
| `status` | which unit files are present, whether the daemon and the API and ssh sockets are active, and whether linger is on |
| `--enable-linger` | also runs `loginctl enable-linger`. **Linger is a persistent setting of your account, so it is opt-in.** Without the flag, or if it is refused, the script prints the one `sudo` command to run and says `linger=no`; **it never runs `sudo` itself and never turns linger off**, since something else may rely on it |

`sd-solo.service` is a one-shot that remains after exit
(`Type=oneshot`, `RemainAfterExit=yes`): `sd-solo -start` forks a daemon that forks
again, and `Type=forking` would make systemd guess the wrong main process.
`sd-solo-api@.service` runs one `sd-solo -n -q` per API connection, and
`sd-solo-ssh@.service` one `sshd -i` per ssh connection, on Solo's own configuration, started through
`tools/solo-sshguard.py`, which counts wrong passwords per address (see `solo-ssh.sh locked` below).

## `solo-ssh.sh`

```sh
bash ~/SDCoreSolo/tools/solo-ssh.sh setup      ~/SDCoreSolo
bash ~/SDCoreSolo/tools/solo-ssh.sh key-add    ~/SDCoreSolo PUBKEY_FILE
bash ~/SDCoreSolo/tools/solo-ssh.sh key-remove ~/SDCoreSolo PUBKEY_FILE
bash ~/SDCoreSolo/tools/solo-ssh.sh key-list   ~/SDCoreSolo
bash ~/SDCoreSolo/tools/solo-ssh.sh migrate    ~/SDCoreSolo
bash ~/SDCoreSolo/tools/solo-ssh.sh match      ~/SDCoreSolo [--remove]
bash ~/SDCoreSolo/tools/solo-ssh.sh locked     ~/SDCoreSolo
bash ~/SDCoreSolo/tools/solo-ssh.sh unlock     ~/SDCoreSolo [ADDRESS]
```

| | |
|---|---|
| `setup` | makes `~/SDCoreSolo/sshd`: Solo's own host key (once), the generated `sshd_config` (rewritten each time) and the key file. Says which path `StrictModes` would refuse. Ends `SOLO SSHD READY port=4251 hostfp=<fingerprint> user=<you>` |
| `key-add`, `key-remove`, `key-list` | the keys in Solo's key file. `--authorized-keys FILE` names another file. Ends `SOLO SSH KEY ADDED <file>`, `SOLO SSH KEY REMOVED <n>` or `SOLO SSH KEYS <n>` |
| `migrate` | moves the Solo key lines an earlier release put in your `~/.ssh/authorized_keys` into Solo's key file, after a copy of the old file. Ends `SOLO SSH MIGRATED <n>`. The upgrade runs it |
| `match` | says whether the old `sshd_config.d` block is still there; with `--remove` (**needs `sudo`**) removes it. Ends `SOLO SSH MATCH PRESENT\|ABSENT\|REMOVED <file>` |
| `locked` | lists the addresses locked out for three wrong passwords in ten minutes, with the time each lock ends. Ends `SOLO SSH GUARD LOCKED <n>` |
| `unlock` | lets one address back in, or every address when none is named. Ends `SOLO SSH GUARD UNLOCKED <n>` |

What each does is on [ssh access](08-ssh-access.html). **`match --remove` has been run once** (2 October 2026), by hand,
after an upgrade.

## `deletesdsolo.sh`

```sh
bash ~/SDCoreSolo/tools/deletesdsolo.sh [--home DIR] [--keep-data | --delete-data] [--yes]
```

Removes SD — see [Upgrading and uninstalling](01a-upgrading-and-uninstalling.html).
Ends `SOLO DELETE COMPLETE <home>`.

## What is here and what is not

**Everything else in the project's `gplbld` directory — the verifiers, the probes,
the build and test cycle — is development tooling and is deliberately not
installed.** If you have read about `assert-current.py`, a `verify-solo-*` script
or `ptyrun.py` and cannot find it on an installed computer, that is why: they
compare an install against the source tree it was built from, or run a test
against a scratch tree, and some start a daemon of their own.

## Continued in

[The scripts SD runs itself](17a-scripts-sd-runs-itself.html) — what the installer
calls, and what nobody types.
