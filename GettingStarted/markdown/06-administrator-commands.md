Title: Administrator commands
Subtitle: admin, the commands that need it, and the maintenance verbs.

**The administrator commands are in your own account, and they need `admin`
first.** There is no separate administrator account to sign in to, and no
`sdsys` Linux user to become.

## `admin`

```
:admin
Administrator password:
Administrator commands unlocked for this session
```

**Type the administrator password** — or, on a managed computer, the global
password. **It lasts until you leave SD or type `admin OFF`**:

```
:admin off
Administrator commands locked
```

| | |
|---|---|
| *Wrong password - administrator commands stay locked* | one try; type `admin` again |
| *Administrator commands are already unlocked* | nothing to do |
| *No administrator password is set on this system* | the installation did not set one; install again |

**A session signed in with the global password is already unlocked.**

**Every attempt is recorded** in the audit trail, unlocked or refused. The
password is never displayed, stored or logged.

## What needs it

**Refused without `admin`, with *Command requires administrator privileges*:**

| | |
|---|---|
| `set.password admin` | change the administrator password — [The account and its passwords](05-account-types.html) |
| `CONFIG` | report or set configuration — except `CONFIG GPL` and `CONFIG CONTRIB`, which need nothing |
| `SET.DATE` | set the session's date |
| `CLEAN.ACCOUNT` | empty the account's scratch files |
| `update.accounts` | refresh the account's VOC |
| `set.os.access` | show, or turn on and off, the shell and OS access switches — [Operating system access](06b-operating-system-access.html) |
| `set.net.access` | show, or turn on and off, the NET gate (API request 29, with `NETFILES=2`) — [API access](09-api-access.html) |
| `set.start.default`, `set.tile`, `set.pins`, `tool.menus` | where a session starts, which tiles the home screen has, whether the account may add its own — [The home screen's settings](#the-home-screens-settings) below |
| `set.viewer`, `tool.viewers`, `docs.install`, `load.help` | the readers `pdf` may start, the documents it shows, the context help — [Operating system access](06b-operating-system-access.html) |
| `set.ai`, `tool.ai` | whether the AI may be used (`set.ai gate on`; off as delivered), its provider, model and key, and the page that does it — [The AI helper](24-the-ai-helper.html) |
| `tool.gates` | all four gates on one page, and the page that changes them (any session sees it; Enter changes a gate after `admin`) — [Operating system access](06b-operating-system-access.html) |
| `tool.sessions` | who is signed on, what is locked, and `logout`, `unlock` and `inhibit.login` on one screen — [Sessions and locks](06a-sessions-and-locks.html) |
| `LISTU`, `LOGOUT ALL` | [Sessions and locks](06a-sessions-and-locks.html) |
| `LIST.READU`, `LIST.LOCKS`, `LOCK`, `CLEAR.LOCKS`, `UNLOCK` | [Sessions and locks](06a-sessions-and-locks.html) |
| anything on the deny list | managed mode only — [Managed mode](15-managed-mode.html) |

**Refused without `admin`, with *The VOC can only be changed after admin*:**
editing the VOC directly — `ED VOC`, a program's `WRITE` or `DELETE` to the
VOC, `COPY` into it — and saving or deleting a sentence with `.S` and `.D`.
What SD writes to the VOC as a side effect of an ordinary command —
`CREATE.FILE`'s entry, the command stack — is not gated.

**Refused even with `admin`**: changing the global catalogue (`CATALOG ...
GLOBAL`, `DELETE.CATALOG` of a global entry, a write to `global.bp.out`), and
the commands that are the SD Core for Linux server's — see
[Managed mode](15-managed-mode.html).

**`LOGOUT` on its own needs nothing** — it ends your own session, like `QUIT`
— **and nor does `sd-solo -k`**, because it is a switch on the program, not a
command in a session. **`sh` needs no `admin`, but it needs its switch**: it is
refused (*is not permitted to use the operating system shell*) until the
administrator has turned `sh` on with `set.os.access`; see
[Operating system access](06b-operating-system-access.html). **Nor does
`set.password`** for your own account password — it asks for the current one
instead; see [The account and its passwords](05-account-types.html).

## The maintenance verbs

### `CONFIG`

```
config                     report every setting
config lptr                the same, to the default printer
config param value         set one, for this session only
config gpl                 display the licence
config contrib             display the contributors
```

**`config param value` sets a private, session-local value, not the
installation's.** The installation's settings live in `sd.conf` and are read
when SD starts — see [Configuration](16-configuration.html). This form
overrides one for the session you are in: the right tool for trying a value,
the wrong one for changing the installation.

| | |
|---|---|
| *New parameter value required* | `config numlocks` with nothing after it. **The report form is `config` alone** |
| *Not a recognised private configuration parameter name* | the name cannot be set per session |
| *Invalid value for this parameter* | it can, and the value is wrong |

### `SET.DATE`

```
set.date date
```

**Sets the date SD reports in this session, not the computer's clock.** SD
keeps an offset from the real date for the session; `DATE()` and `TIMEDATE()`
return the new date until the session ends. Other sessions and Linux are not
affected. The argument goes through SD's `D` conversion, so anything
`iconv(…, 'D')` accepts will do:

| | |
|---|---|
| *Date required* | nothing after `set.date` |
| *Invalid date format* | the argument is not a date SD can read |

It is for testing date-dependent code without touching the clock.

### `CLEAN.ACCOUNT`

```
:clean.account
Cleaned $COMO
Cleaned $hold
Cleaned $savedlists
```

Empties the account's captured transcripts (`$COMO`), its hold file of reports
(`$hold`) and its saved select lists (`$savedlists`). **Nothing else is
touched** — no data file, no program, no dictionary. A como capture that is
running is left alone and says so.

### `update.accounts`

```
update.accounts {all}
```

**Copies SD's shipped command definitions into your VOC**, adding what is
missing and leaving your own VOC records alone. An upgrade runs it for you, so
you will not normally type it. With one account, `all` and no keyword do the
same thing.

**It never takes anything away.** A record you removed stays removed. To keep
your own version of one of SD's records, put `[locked]` in field 1 **after the
type code** — `V[locked]`, `PA[locked]` — and it is left alone. **A verb is
updated anyway**, because a locked verb would go on naming a program this
release replaced; you are told which ones.

**Your own changes to the VOC: `newvoc.mods`.** An optional directory file called
`newvoc.mods` in the SD system directory holds VOC records of your own — a verb
withdrawn, a Q-pointer to a shared file, a paragraph the account should have. Each
record in it is written to the account's VOC **after** the standard records, so it
has the last word, except over a record marked `[locked]`. The file is a directory
file: one record per file, the record id as the file name, the VOC record as its
contents. There is no file until you make one, and nothing is created for you. It
is applied when the VOC is built at install and at every `update.accounts`, which
an upgrade runs, and **an upgrade leaves the file where it is.** `deletesdsolo.sh
--keep-data` keeps it too: the account, `sd.conf` and a `newvoc.mods` directory are
what is left beside the `.sdcore-kept` stamp, and a new install that reloads the
saved data puts the directory back before it refreshes the VOC, so your records are
applied again at once. (Only a real directory is kept; a link or a plain file of that
name is removed with the rest, and `--delete-data` removes it as it removes everything.) To see it work, put a record in the
file and run `update.accounts`; then `ct voc` the id.

### `backup.account` and `restore.account`

```
backup.account {to directory}
restore.account archive
```

There is one account, so **neither command needs a name**: with none, they fill
in the account's name, `sduser`, exactly as if you had typed it (`all`, and the
name, are still accepted). After `admin`, `backup.account` writes
**one zip file**, named for the computer, the account and the time, holding the account's
files and a plain-text description of it. `restore.account archive` puts the
account back, on this computer or another one. **A backup from another person's SD Core Solo replaces
this account's data**, so it says what will be replaced and asks first. A backup
made here restores on SD Core Solo for Windows, and the other way round.

**Every backup is checked as it is made**, and one that does not match the
account is deleted rather than kept. A backup or restore starts only when no
other session is logged in, and no one can log in — at the terminal, over ssh or
through the API — until it has finished. **A backup never overwrites a file:** if
the name is taken, SD says so and writes nothing. **An archive that lies is
refused with nothing changed** — counts that disagree with the manifest, an
entry path with `..` in it or an absolute path, no manifest or two of them.

**A restored account comes back as it was:** each directory has the permissions
it was backed up with (a directory that was `0700` stays `0700`).

**A restore of the account you are using cannot happen while SD is running.**
`restore.account` checks the archive, prepares it, and says so; the restore
happens the next time SD starts (`systemctl --user restart sd-solo.service`, or
stop SD and start it). The account as it was is kept beside it, in
`.sdrestore.previous`, until the next restore, and every step is written to
`sdrestore.log` in the SD Core Solo folder.

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

### `set.backup.directory`

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
does the same for an archive named without a directory. `to directory`, or an
archive name that carries a directory, still works and changes nothing that is
saved.

**The directory must be a full path** — a relative one is refused. It may hold
only letters, digits and `. _ @ + = , : / -` and a space, with no `//` and no `.`
or `..` part, and is refused with the reason if it does not. **It is made readable
by you only**, because a backup holds your account's files. An earlier release
refuses to start if `sd.conf` holds a `BACKUPDIR` line, so take the line out
before going back to one.

### `settings.report`

```
settings.report {directory}
```

Writes, or shows, a **plain-text record** of the settings — `sd.conf`, the
policy, the mode, the API and ssh, the service — to keep for reference. Nothing
reads it back, and it never contains a password or a private key.

## The home screen's settings

```
set.start.default {menu | terminal}
set.tile sduser tile {on | off}
set.tile
set.pins {on | off}
tool.menus
```

**The delivered start is the home screen.** A session on a terminal opens it (tiles for the
editor, the IDE, the record editor, a terminal, the keyboard test, and Sign out), and the
command prompt is what is left when it returns. `set.start.default terminal` makes the prompt
the start, `set.start.default menu` the home screen again, and without a word the verb says
what it is now. It is read when a session starts. A piped session, a window under 140 by 43,
a phantom and an API server are never given the screen. **After an upgrade, if you want the
old start, type `admin` and then `set.start.default terminal`.** Starting in the menu confines
nobody: the Terminal tile is there unless you turn it off.

**`set.tile sduser tile off`** takes a tile off the page, and `on` puts it back (every tile is on as
delivered; the ids are `editor`, `ide`, `ced`, `terminal`, `keys` and `signout`). `set.tile`
alone says what is off. **Sign out cannot be turned off.** With **Terminal** off and a start in
the home screen, the account is never given the prompt: when the screen ends it signs the
person out. That is a wall only for the prompt the screen gives; the editor, IDE and record
editor tiles run the account's own programs, so keep the account to the screen by turning those
off as well, and a shell still needs `set.os.access sh on`.

**`set.pins on`** lets the account add programs of its own to its home screen (a group *My
programs* and the `my.menu` verb); the permission is **off as delivered**, `set.pins off` takes
it back (the person's list stays in their file), and without a word the verb says which it is. A
program of the account's own runs the command the person could type at the prompt, so it adds
no access: the switches and the gates apply as they do there.

**`tool.menus`** is the same settings as a page: the *Accounts* box and the *Menu* box (where the
account starts, whether it may add its own programs, and a line for each tile), with a `[?]`
on each row (F1, `?` or a click) that explains it. **It writes nothing itself**: Enter on a row
shows the command it would run (`set.tile sduser editor off`) with its effect in a box answered
Run or Cancel (Cancel is what Enter chooses), and the line under the page is the verb's own
answer. A change reaches the person at their next sign-on, or F5 in the home screen. The three
verbs write the audit trail, in lower case; the page only runs them.

## Not here

There is no `APPEND.SD.PATH`: the installer links `sd-solo` into `~/.local/bin`, and
whether that directory is on your PATH is between you and your shell's startup
file. There are no `create.account`, `delete.account`, `modify.account`,
`GRANT`, `REVOKE` or `LIST.GRANTS` — see [Not in SD Core](14-not-in-sd-core.html).
