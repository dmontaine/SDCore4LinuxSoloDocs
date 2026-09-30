Title: The installed scripts
Subtitle: The scripts in the installed directory, what each does, and what their exit codes mean.

**Solo ships three scripts into the installed directory, in `~/SDCoreSolo/tools`,
and one that you run from the source repository.** They are there for two
reasons: so that a step which failed during the installation can be run again
without reinstalling, and so that a choice made at install time can be changed
afterwards.

**They are shell scripts, not SD verbs.** Nothing here is typed at an `sd`
prompt. **None of them needs `root`, and each refuses to run as root.** Where
something does need `sudo` — linger, the `sshd_config.d` block — the script says
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

## `installsolo.sh` — in the source repository

```sh
bash installsolo.sh [options]
bash installsolo.sh --upgrade [--home DIR]
```

Installs, or upgrades. **It is not copied into the installed directory**: it
downloads the source, and the source contains it. Its options are listed by
`bash installsolo.sh --help` and described on
[Installing](01-installation.html) and
[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html).

## `solo-service.sh`

```sh
bash ~/SDCoreSolo/tools/solo-service.sh install ~/SDCoreSolo [--api off|local|open] [--api-port N] [--enable-linger]
bash ~/SDCoreSolo/tools/solo-service.sh remove
bash ~/SDCoreSolo/tools/solo-service.sh status
```

| | |
|---|---|
| `install` | writes the three **user** units into `~/.config/systemd/user` — `sd-solo.service`, and with an API `sd-solo-api.socket` and `sd-solo-api@.service` — naming the tree by its full path; enables and starts them. **Running it again with a different `--api` or `--api-port` changes the API**: the old listener is stopped first. Ends `SOLO SERVICE READY daemon=<state> api=<off\|local\|open> linger=<yes\|no>` |
| `remove` | stops SD and removes the units. Ends `SOLO SERVICE REMOVED` |
| `status` | which unit files are present, whether the daemon and the API socket are active, and whether linger is on |
| `--enable-linger` | also runs `loginctl enable-linger`. **Linger is a persistent setting of your account, so it is opt-in.** Without the flag, or if it is refused, the script prints the one `sudo` command to run and says `linger=no`; **it never runs `sudo` itself and never turns linger off**, since something else may rely on it |

`sd-solo.service` is a one-shot that remains after exit
(`Type=oneshot`, `RemainAfterExit=yes`): `sd -start` forks a daemon that forks
again, and `Type=forking` would make systemd guess the wrong main process.
`sd-solo-api@.service` runs one `sd -n -q` per API connection.

## `solo-ssh.sh`

```sh
bash ~/SDCoreSolo/tools/solo-ssh.sh key-add    ~/SDCoreSolo PUBKEY_FILE
bash ~/SDCoreSolo/tools/solo-ssh.sh key-remove ~/SDCoreSolo PUBKEY_FILE
bash ~/SDCoreSolo/tools/solo-ssh.sh key-list   ~/SDCoreSolo
bash ~/SDCoreSolo/tools/solo-ssh.sh match      ~/SDCoreSolo [--apply | --remove]
```

`--authorized-keys FILE` on the three key commands names a different file from
`~/.ssh/authorized_keys`. What each does, and what the `Match` block is, is on
[ssh access](08-ssh-access.html). **`match --apply` and `--remove` need `sudo`,
and are written but not measured.**

## `deletesolo.sh`

```sh
bash ~/SDCoreSolo/tools/deletesolo.sh [--home DIR] [--keep-data | --delete-data] [--yes]
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
