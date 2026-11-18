Title: Backing up and restoring the account
Subtitle: Writing your account to a zip file, putting it back, remembering the backup folder, and recording the system's settings.

This page continues [Administrator commands](06-administrator-commands.html).

## `backup.account` and `restore.account`

```
backup.account {to directory}
restore.account archive
```

There is one account, so **neither command needs a name**: with none, they fill
in the account's name, `sduser`, exactly as if you had typed it (`all`, and the
name, are still accepted). After `admin`, `backup.account` writes
**one zip file**, named for the computer, the account and the time (for example
`SD-office-sduser-20261001-144419.zip`), holding the account's
files and a plain-text description of it. `restore.account archive` puts the
account back, on this computer or another one. **A backup from another person's SD Core Solo replaces
this account's data**, so it says what will be replaced and asks first (the default
answer is no; `no.query` skips the question). A backup
made here restores on SD Core Solo for Windows, and the other way round.

```
:backup.account
Backed up 1 account(s) to /home/you/backups/SD-office-sduser-20261001-144419.zip
  sduser: 4 files, 78848 bytes, 6 directories
```

**Every backup is checked as it is made**, and one that does not match the
account is deleted rather than kept: the account is counted before it is packed,
the counts are written in the zip, and if what was written differs the zip is
deleted and the verb says by how much. A backup or restore starts only when no
other session is logged in, and no one can log in — at the terminal, over ssh or
through the API — until it has finished. **A backup never overwrites a file:** if
the name is taken, SD says so and writes nothing. **An archive that lies is
refused with nothing changed** — counts that disagree with the manifest, an
entry path with `..` in it or an absolute path, no manifest or two of them, or a
backup of another product (a Solo backup restores only into a Solo).

**A restored account comes back as it was:** each directory has the permissions
it was backed up with (a directory that was `0700` stays `0700`).

**A restore of the account you are using cannot happen while SD is running.**
`restore.account` checks the archive, prepares it, and says so; the restore
happens the next time SD starts (`systemctl --user restart sd-solo.service`, or
stop SD and start it); **nothing has changed until SD is restarted**. When SD
starts it swaps the restored account in, before any session exists. The account
as it was is kept beside it, in `.sdrestore.previous`, until the next restore, and
every step is written to `sdrestore.log` in the SD Core Solo folder.
**The audit trail** gets a line when the restore has been prepared,
`restore.account account=sduser archive=<backup file name>`, naming the file and
not its directory; the swap itself is in `sdrestore.log`, not in the audit trail.

**It does not:** carry a password; back up SD's own system files or the
settings; follow symbolic links in the account (any it finds are named and left
out); or restore a Solo backup onto the multi-user SD Core.

**`restore.account latest` restores the most recent backup without your
naming it.** `latest` stands where the archive name goes: SD looks in the
directory saved by `set.backup.directory`, picks **the newest backup made on this
computer that really holds your account**, prints which one it chose (*The most
recent backup is …*), and restores from it exactly as if you had typed its name.
SD decides by *looking inside*: a backup is called
`SD-<computer>-sduser-<yyyymmdd-hhmmss>.zip`, or the same with `all`, but a
backup made with `all` names no account, so SD opens each backup the name might
fit and reads its list of accounts (nothing is unpacked).

**If a newer backup made on this computer does not hold your account** — or
cannot be read — SD says so before it asks you to go ahead: *The most recent
backup made on this computer, …, does not hold … (or cannot be read). The newest
backup that does is …*. You can still answer `n`. If **no** backup holds it, SD
says *No backup of sduser made on this computer was found in …* and changes
nothing. Whichever backup is picked is still checked against its own contents
in full before anything is changed, and one made on another computer is never
picked. `no.query` skips the questions, as with a named archive.

**`latest` only considers backups SD named itself**, `SD-<computer>-sduser-<yyyymmdd-hhmmss>.zip`
or the same with `all`. A backup you have renamed is never picked: restore it by
giving its name (`restore.account myfile.zip`). A bare name is looked for in the
saved directory; a name with a directory in it is used as given.

## `set.backup.directory`

```
set.backup.directory directory
set.backup.directory
```

After `admin`, saves the directory that `backup.account` writes to and
`restore.account` reads from, so it need not be typed each time. **It creates the
directory if it is not there**, checks that you can write to it, and keeps it in
`sd.conf` (`BACKUPDIR=`), where it takes effect at once — no restart. On its own
it shows the saved directory and changes nothing.

With a directory saved, `backup.account` no longer needs `to`; with none saved it
asks for one and saves the answer exactly as this verb would. `restore.account`
does the same for an archive named without a directory (`restore.account` with
`no.query` has nobody to ask and refuses instead). `to directory`, or an
archive name that carries a directory, still works and changes nothing that is
saved.

**The directory must be a full path** — a relative one is refused. It may hold
only letters, digits and `. _ @ + = , : / -` and a space, with no `//` and no `.`
or `..` part, and is refused with the reason if it does not. **It is made readable
by you only**, because a backup holds your account's files. An earlier release
refuses to start if `sd.conf` holds a `BACKUPDIR` line, so take the line out
before going back to one.

## Backups and restores on one screen: `tool.backups`

```
tool.backups
```

A full-screen page for the administrator (`admin` first; a window of 140 columns by 43 lines or
more). The top box names the saved backup directory, says how many archives it holds, and says
**whether another session is signed on**: `backup.account` and `restore.account` both run only when
every other session has logged out, so the page lists the other sessions (the same check the verbs
make) and **shows no box for a command while any is there**. The list is the archives of the
directory (the files `SD-host-...-yyyymmdd-hhmmss.zip`), newest first: when each was made (from its
name), its size, how many accounts it holds and for which product (from its manifest). The box at
the foot shows the manifest of the one under the cursor, read without unpacking anything: the
product, the SD version, the host and the date, and the account with its files, bytes and
directories.

| Key | Runs |
|---|---|
| `B` | `backup.account all` |
| `A` | `backup.account name ...`, for the account names you type (Solo has one, `sduser`; a name that is not an account is refused by the verb) |
| `R` | `restore.account archive all no.query`, the archive under the cursor |
| `D` | `set.backup.directory path`, for the absolute path you type |

Each shows its command and what it does in a box answered Run or Cancel, **and Cancel is what Enter
chooses**. The page runs those four commands and no other; the verbs ask for the administrator again
and write their own audit lines. **A restore replaces the account's data**, so its box says so and
wants the word `restore` typed (in either case). The verb's own question (Enter is No) cannot be answered through a
page, so the page gives `no.query` after its box has asked; it does not give it any other way. An
archive that holds an account other than `sduser` is refused by the page, which names it. As at
the prompt, the restore of the account you are using is prepared and happens the next time SD
starts. Where a command cannot be done, the verb says why and the page shows its last line on the
status line. `F5` reads the directory, the archives and the sessions again, `F1` explains the page
and `Ctrl-Q` leaves. Opening the page writes the audit line `tool start app=tool.backups`.

## `settings.report`

```
settings.report {directory}
```

Writes, or shows, a **plain-text record** of the settings — `sd.conf`, the
policy, the mode, the API and ssh, the service — to keep for reference. Nothing
reads it back, and it never contains a password or a private key. Its `[mode]` section says
whether the computer is managed; when the register of passwords cannot be opened it says so
(*unknown (the credential register could not be read, status N)*) rather than claiming
*not managed*.

## See also

[Administrator commands](06-administrator-commands.html) · [Configuration](16-configuration.html) for `BACKUPDIR` · [Known limits](19-features-the-developers-could-not-test.html).
