Title: Upgrading and uninstalling
Subtitle: Installing a new release over an existing one, and taking SD off the computer.

This page continues [Installing](01-installation.html).

## Upgrading

**Run the installer with `--upgrade` while the old release is installed:**

```sh
bash installsdsolo.sh --upgrade
```

(`--home DIR` if you installed somewhere other than `~/SDCoreSolo`.) Without
`--upgrade`, an installer meeting an installed tree stops and prints exactly
this command. **It asks nothing** — no mode, no passwords, no API or ssh
choices — and it accepts no option that would change them: an upgrade keeps
what is installed (one exception: an installation that used the old ways in for ssh gets Solo's own ssh port, below). To change the mode or the API, reinstall; ssh can also be
changed with `solo-service.sh ssh` — see [ssh access](08-ssh-access.html).

**It downloads and builds the new release first**, before it touches your
installation, so a failed download or build changes nothing. Then it stops SD
and its systemd service, and:

1. **makes a safety copy of the whole tree**, data included, next to it:
   `~/SDCoreSolo.before-upgrade-<date and time>`;
2. replaces the code and the shipped system files, runs the bootstrap over
   them, brings your account up to the release, and starts the service again.

**If any step fails, the tree is put back from the safety copy** and the
service is started again, so a failed upgrade leaves you where you were. The
copy is **left in place after a good upgrade too**; delete it when you are
satisfied. The upgrade prints its path.

**What an upgrade replaces, and what it keeps:**

| | |
|---|---|
| **replaced** | the programs (`bin/`), the system programs and their catalogue, SD's messages, the VOC templates and SD's own VOC and dictionaries, and the other files the release ships |
| **kept, byte for byte** | your account and its data (`user_accounts/sduser`), the credential store `$cred` (all three passwords and the kept copy), `sd.conf`, the audit trail and the error log (the audit trail keeps its old lines and appends), the service units and Solo's own ssh directory with its host key and key file (the units are renamed in place if they name the old server file, below), and on a managed computer the server's programs in `global.bp.out` and the list of denied commands in `solo.policy` |

**Upgrading an installation made before the server was renamed `sd-solo`.**
The server used to be installed as `bin/sd`, and `sd` was the command that
started it. An upgrade of such an installation also:

- **stops SD by its old name**, so the old daemon does not keep running;
- **removes the old `~/.local/bin/sd`** if it is this installation's (a link to
  `bin/sd`, or the launcher an earlier test build made). **A file of your own
  called `sd` is left alone, without a message**; plain `sd` is SD Core's name
  now, and this release never makes it;
- **moves the systemd user units** to the new file name;
- **if an `sshd_config.d` block written by an earlier release still names the
  old file, leaves a link at `bin/sd`** until you remove the block (below).

A program or script of your own that starts SD Core Solo by the file name
`bin/sd` must now say `bin/sd-solo`.

**Upgrading an installation made before Solo had its own ssh port (LS1.1-3).**
Earlier releases let ssh into Solo through the computer's own ssh server: a
forced-command line in your `~/.ssh/authorized_keys`, and optionally a block in
`sshd_config.d`. Both are gone. Solo's ssh now has its own port, 4251 — see
[ssh access](08-ssh-access.html). An upgrade of such an installation:

- **turns Solo's own ssh on, `local`** (this computer only), if you had used
  either way in; and does nothing about ssh if you had not;
- **moves the Solo key lines out of your `~/.ssh/authorized_keys`** into Solo's key file
  (`~/SDCoreSolo/sshd/authorized_keys`), after keeping a copy of the old file as
  `authorized_keys.sdsolo-backup-<time>`. Every other key line is left as it was;
- **prints the one command that removes the old `sshd_config.d` block**, which
  needs `sudo`: `bash ~/SDCoreSolo/tools/solo-ssh.sh match ~/SDCoreSolo --remove`.
  **Run it.** Solo no longer uses the block, and until it is gone it still sends
  your ssh logins on port 22 into Solo (it also removes the link at `bin/sd`);
- **says so if the ssh server program is not installed**, and prints what to run
  once it is.

Port 4251 takes **keys only**: a password login that worked through the old
block does not work on the new port.

**Then it brings your account up to the release.** Replacing files is not
enough on its own: your account's VOC was built by the release that installed
it. So an upgrade also, for you:

- adds the new release's commands to your account's VOC
  (`UPDATE.ACCOUNTS ALL`) — it never takes anything away, and a record you keep
  your own version of, marked `[locked]` in field 1, is left alone;
- recompiles SD's own dictionaries;
- catalogues the server's programs in `global.bp.out` again, on a managed
  computer (`SYNC.GLOBAL.CATALOG`), because the global catalogue is one of the
  files replaced.

The installer records where it came from: `~/SDCoreSolo/.sdcore-install` gains
an `upgraded-from` line naming the old commit. The last line of a good upgrade
is:

```
SOLO UPGRADE COMPLETE /home/you/SDCoreSolo
```

**Limits.** An upgrade replaces every file it lists, so a system file you edited
by hand is replaced; edit `sd.conf` (kept) or your account (kept), not SD's own
files. A tree that was not installed by `installsdsolo.sh` (it has no
`.sdcore-install` record) is refused rather than guessed at.

## Uninstalling

**`deletesdsolo.sh` is in the installed directory, `~/SDCoreSolo/tools`, and in
the source repository.** Run it as yourself, not with `sudo`:

```sh
bash ~/SDCoreSolo/tools/deletesdsolo.sh
```

It says what it will remove, and asks whether to keep your data:

| | |
|---|---|
| `--keep-data` | moves `user_accounts/sduser` to `~/SDCoreSolo-data-<date and time>` first |
| `--delete-data` | removes it with the rest |
| `--yes` | do not ask "Continue?" — the data choice is still required when there is no terminal |

**What it removes:** the systemd user units, and it stops SD; the
`~/.local/bin/sd-solo` link (only if it points at this tree); the ssh key lines an
earlier release added to your `authorized_keys` (**your other keys are not touched**);
the old `sshd_config.d` block, if you had written one (that needs `sudo`); and the
installation directory, which holds Solo's own ssh directory. A firewall rule for
port 4251 or 4249 is not removed; the script says so.

**The passwords, the audit trail and SD's own files are always removed.** What
`--keep-data` keeps is your data — the account's files — not an installation: a
kept `user_accounts` cannot be attached to a new install by this script yet, so
copy what you need out of it.

**What it leaves:** linger (a persistent setting of your account that other
things may rely on), the build packages, and any firewall rule the installer
added — it says so if it finds one. The last line of a good run is:

```
SOLO DELETE COMPLETE /home/you/SDCoreSolo
```

It never deletes the directory it is running from: from inside the tree it
copies itself to a temporary file and runs from there.

## Continued in

[Differences from multiuser SD Core for Linux L1.1-1](01b-differences-from-multiuser-l1-1-1.html)
— what Solo leaves out, adds, and does differently.
