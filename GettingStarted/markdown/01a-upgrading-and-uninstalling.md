Title: Upgrading and uninstalling
Subtitle: Replacing an existing installation, and taking SD off the machine.

This page continues [Installing](01-installation.html).

## Upgrading

**Run the new release's installer while the old release is installed:**

```sh
bash installsdsolo.sh
```

(`--home DIR` if you installed somewhere other than `~/SDCoreSolo`.) It finds the
installed tree, says *already installed: upgrading it in place*, and upgrades it;
no uninstall is needed. `--upgrade` still works and means the same. **It asks no mode, no passwords and no API or ssh choices,** and
it accepts no option that would change them: an upgrade keeps what is
installed (one exception: an installation that used the old ways in for ssh gets SD Core Solo for Linux's own ssh port, below). To change the mode or the API, reinstall; ssh can also be
changed with `solo-service.sh ssh` — see [ssh access](08-ssh-access.html). The
only options it takes are `--os-access` and `--viewer` (below), and the one
question it may ask is about a PDF reader (see [Installing](01-installation.html)).

**The shell and OS access are kept as they are.** The shell (`sh`) and OS access
(`os.execute`, a file or directory outside the account, the embedded Python and
`micro`) were on for every installation of an earlier release; **they are off
unless an administrator turned them on**. An upgrade does not ask about them: it
keeps the two switches the installation has, so it neither shuts what was opened
nor opens what was shut, and says so. **`--os-access none|sh|os|both` is still
applied on an upgrade, but only to turn a switch on**: `sh`, `os` or `both` turns
that switch on, and `none` (or no option) changes nothing. It never turns one off.
An installation from before the switches existed has neither set, so **both are
off** after the upgrade until the administrator turns them on or off with
`set.os.access` ([Operating system access](06b-operating-system-access.html)). A
program of yours that runs the shell or `os.execute`, or opens a file outside the
account, stops working after such an upgrade until then.

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
| **replaced** | the programs (`bin/`), the system programs and their catalog, SD's messages, the VOC templates and SD's own VOC and dictionaries, and the other files the release ships |
| **kept, byte for byte** | your account and its data (`user_accounts/sduser`), the credential store `$cred` (all three passwords and the kept copy), `sd.conf`, the audit trail and the error log (the audit trail keeps its old lines and appends), the service units and Solo's own ssh directory with its host key and key file (the units are renamed in place if they name the old server file, below), and on a managed computer the server's programs in `global.bp.out` and the list of denied commands in `solo.policy` |

**Upgrading an installation made before the server was renamed `sd-solo`.**
The server used to be installed as `bin/sd`, and `sd` was the command that
started it. An upgrade of such an installation also:

- **stops SD by its old name**, so the old daemon does not keep running;
- **removes the old `~/.local/bin/sd`** if it is this installation's (a link to
  `bin/sd`, or the launcher an earlier test build made). **A file of your own
  called `sd` is left alone, without a message**; plain `sd` is SD Core for Linux's name
  now, and this release never makes it;
- **moves the systemd user units** to the new file name;
- **if an `sshd_config.d` block written by an earlier release still names the
  old file, leaves a link at `bin/sd`** until you remove the block (below).

A program or script of your own that starts SD Core Solo for Linux by the file name
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

Port 4251 takes your **Linux account name and password**, as the old block did, and
a key in Solo's key file as an alternative; SD then asks the account password.

**Then it brings your account up to the release.** Replacing files is not
enough on its own: your account's VOC was built by the release that installed
it. So an upgrade also, for you:

- adds the new release's commands to your account's VOC
  (`update.accounts all`), and takes out the records on SD's deletion list - the
  ones a release stopped shipping - but only a record that is still exactly what SD
  shipped, after saving a copy (`restore.voc` puts it back); a record you keep
  your own version of, marked `[locked]` in field 1, is left alone;
- deletes the files of the system directory that this release stopped shipping, and
  takes the `sd.conf` parameters SD no longer has out of your kept `sd.conf` (SD would
  refuse to start on one);
- converts your account's files to case-insensitive record ids (below);
- merges and recompiles SD's own dictionaries;
- catalogs the server's programs in `global.bp.out` again, on a managed
  computer (`sync.global.catalog`), because the global catalog is one of the
  files replaced.

**Your files are converted to case-insensitive record ids for you.** Every file
`create.file` makes treats ids as one whatever the case (see
[Lower case](11-lower-case.html)), and an upgrade keeps your existing files, which
were built case sensitive. After the account's VOC is brought up to the release, the
upgrade reads each file the account's VOC names first, **and converts it only if no
two of its ids differ only by case**, so a conversion cannot lose a record. It also
makes the account's private catalog names lower case. What it did is printed as the
upgrade goes. A file is **left exactly as it was** and named when:

- it holds two ids that differ only by case (rename or delete one of each pair, then
  run `configure.file` *name* `no.case` on it);
- it has alternate key indexes (convert it by hand with `configure.file`);
- it lies under the SD system directory, which the upgrade makes new itself
  (convert one yourself with `configure.file` *name* `no.case` if you need to);
- it could not be read or rebuilt.

A walk that did not finish prints a warning and the upgrade goes on; what was not
converted keeps working, and ids are still found typed in any case.

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

**What changed since the previous release**, and what an upgrade might break,
is in the *Changes* document that comes with the release: every addition,
change and removal, listed by section.

## Uninstalling

**`deletesdsolo.sh` is in the installed directory, `~/SDCoreSolo/tools`, and in
the source repository.** Run it as yourself, not with `sudo`:

```sh
bash ~/SDCoreSolo/tools/deletesdsolo.sh
```

It says what it will remove, and asks whether to keep your data and configuration:

| | |
|---|---|
| `--keep-data` | **leaves your data and configuration where they are**, in `~/SDCoreSolo`: the account's files (`user_accounts/sduser`), `sd.conf`, **the passwords** (`$cred`), the audit trail (`audit` and its rotated copies), the API's TLS key (`sd-tls`), the ssh server's host key and your keys (`sshd`), the AI settings and keys (`ai`), the SDNet logins (`netlogins`), where sessions start (`start.default`), the account register (`accounts`), from which a reinstall that reloads your data turns back on the switches you had turned on (SH, OS.EXECUTE, the edit gates, net access, file transfer, scheduled jobs and the others `tool.gates` shows; nothing is turned off, and nothing is turned on that was off) and, if there are any, the `newvoc.mods` directory of your own VOC records and the saved copies of VOC records the update removed (`voc_removed.saved`) — what SD Core for Linux keeps when its accounts are kept. Everything else in the directory is removed |
| `--delete-data` | removes the whole directory |
| `--yes` | do not ask "Continue?" |

**With no terminal to ask on and neither `--keep-data` nor `--delete-data`, the data is
kept** — nothing is deleted unasked (SD Core Solo for Windows' silent uninstall does the
same). Say `--delete-data` to remove it.

**What it removes:** the systemd user units, and it stops SD; the
`~/.local/bin/sd-solo` link (only if it points at this tree); the ssh key lines an
earlier release added to your `authorized_keys` (**your other keys are not touched**);
the old `sshd_config.d` block, if you had written one (that needs `sudo`); and the
installation directory, which holds Solo's own ssh directory — all of it, except
what `--keep-data` leaves. **The firewall rules for Solo's own ports, 4249 and 4251,
stay when you keep the data** (you will probably reinstall) **and are removed when you
delete it**; the script says which. Port 22 and port 4243 are never touched.

**`--keep-data` also notes the API and ssh doors as they are** (off, this computer
only, or open), in `doors.saved`, and keeps the PDF reader allow-list (`viewers`), the
documents in `pdf/` and a managed computer's denied commands (`solo.policy`). A
reinstall that reloads the data puts them back and asks nothing about the doors; an
`--api`, `--ssh` or `--deny-verbs` you give it still wins.

**When something cannot be put back, the install says so in a red box.** If a
reinstall that reloads your data could not put something back as it was (the kept
scheduled jobs that could not be given their timers again, the administrator's
account switches that could not be carried from the kept register, or the firewall
rule for the API or ssh), the closing summary starts, right under *SD Core Solo for
Linux is installed.*, with *WARNING: SOME SETTINGS NEED TO BE FIXED.* in bold red.
Each numbered item gives the lines to type at the colon prompt after you start SD
Core Solo. Type them one at a time and in order: `admin` asks for the administrator
password, and `set.job change` asks for the account password.

A job needs the jobs switch on while it is made again, so the item may turn it on
first and, when it was off before, off again at the end. When a kept definition is
damaged the line shows `<schedule> <command>`: put in what you had. The box does not
give reasons; the reason is in the install's output above it. If a step fails when
you type it, correct what it reports and type it again.

A firewall rule that could not be added has a red warning of its own, with the
command for a terminal: Solo never changes the firewall from inside SD. The
uninstaller shows the same kind of box at its very end when it could not note the
API and ssh doors; do those steps after the next install. Nothing is printed when
everything was put back.

**Your screen theme comes back as you had it.** An install, an upgrade or a reload
leaves the `SCREENTHEME` line of a kept `sd.conf` exactly as it was; a kept file with
no such line stays on `mono`. Only a new install starts in `paper`, from the shipped
`sd.conf`.

**The units come first, and the script stops if it cannot remove them.** It looks for
your systemd user manager under `/run/user/<your number>` when the shell it was
started from has no `XDG_RUNTIME_DIR` (an `ssh` command or a cron job), removes the unit
files itself if `solo-service.sh remove` is refused, and, if any file whose name begins
`sd-solo` is still in `~/.config/systemd/user`, ends with an error that names the
files **before it removes the installation directory or your data**. Run it again from a
login session, or with `XDG_RUNTIME_DIR` set, or remove those files yourself.

**SD's own files, the deny list and the programs an SD Core server cataloged
(`global.bp.out`) are always removed**; a server puts the last two back. What
`--keep-data` leaves is your data, your configuration and what goes with them, not an
installation, plus a small stamp, `.sdcore-kept`, that says which release made the
data and when it was left. Nothing is moved, so nothing can be lost in a move. To
remove what was kept for good, delete the directory (`rm -rf ~/SDCoreSolo`).

### Installing again over kept data

**A new install into a directory that holds kept data offers it back.** The
installer recognizes it by the stamp, the account and what `--keep-data` keeps, and
nothing else (a directory with anything more in it, or with a link where one of those
should be, is refused, as any non-empty directory is):

```
Saved data was found: /home/you/SDCoreSolo
Reload your saved data and configuration into this new install? [Y/n]
```

| | |
|---|---|
| **Yes** (the default) | the account's files, `sd.conf` and the rest of what was kept are used in the new install: the passwords, the TLS and ssh host keys (so clients that trust them go on trusting them), the AI settings, the SDNet logins and where sessions start (not asked again); the audit trail goes on from where it was. The account's VOC is refreshed the way an upgrade does it. The directory as you left it is kept beside the new one as `~/SDCoreSolo.kept-<date and time>`, a safety copy — delete it when you are satisfied |
| **No** (start clean) | the kept data is moved aside to `~/SDCoreSolo.kept-<date and time>`, **never deleted**, and the install is a clean one |

**A reload keeps the passwords and asks for none**: the installer says
*Passwords: kept from the previous install - not changed*. Starting clean asks for new
ones, as a fresh install does. **To replace the kept passwords, give password files with
the reload** (`--account-password-file`, `--admin-password-file`, `--global-password-file`,
or a control file with a global password): they are set instead, and the kept ones are not
used. **This is how a computer becomes managed after the fact: uninstall keeping the data,
install again with `--reload-data --global-password-file` *file* and the other two.** A
kept install from before this release kept no passwords, so its reload asks for them.

**If SD will not start on the kept `sd.conf`** (an item this release no longer knows),
the install still completes with the default `sd.conf` and says so; the kept copy is
untouched, so you can correct it and put it in place by hand.

For an install nobody sits at, `--reload-data` and `--start-clean` answer the
question, as does the control file's `reload-data=yes` or `no`. With neither and no
terminal, the kept data is reloaded. When nothing was kept the options do nothing, so
one control file can serve many computers.

**What it leaves:** linger (a persistent setting of your account that other
things may rely on), the build packages, and, when the data is kept, the firewall
rules for ports 4249 and 4251 — it says so if it finds one. The last line of a good run is:

```
SOLO DELETE COMPLETE /home/you/SDCoreSolo
```

It never deletes the directory it is running from: from inside the tree it
copies itself to a temporary file and runs from there.

## Moving the account to another computer

Moving the account is a backup on one computer, a copy of one file, and a restore on the other.
There is no verb that does it in one go; the two verbs are described under
[Administrator commands](06-administrator-commands.html) and in
[Backing up and restoring the account](06c-backup-and-restore.html).

1. On the old computer, after `admin`: `backup.account`. It writes one zip file to the saved
   backup directory and says its name.
2. Copy that file to the new computer (`scp`, a USB stick, anything that copies a file).
3. On the new computer, install SD Core Solo for Linux, sign in, type `admin`, and run
   `restore.account` with the file's full path. It says what will be replaced and asks first.
4. **The restore happens the next time SD starts** (`systemctl --user restart sd-solo.service`,
   or stop SD and start it), because it replaces the account you are using.

**What does not come across:** a password (the new computer keeps the passwords it was installed
with), SD's own system files and settings (`sd.conf`, the global catalog, the ssh and API choices),
and symbolic links in the account. **The backup is an ordinary, unencrypted zip** of every file in
the account: keep it where only you can read it, and delete it when the move is done. A Solo backup
restores only to Solo (on Linux or on Windows), never onto the multi-user SD Core.

## Continued in

[Differences from SD Core for Linux LC2.0-0](01b-differences-from-sd-core-lc2-0-0.html)
— what Solo leaves out, adds, and does differently.
