Title: Operating System Access
Subtitle: The two switches, sh and os, that decide whether SD can reach the operating system: off as delivered, turned on by the administrator.

`sh` runs a Linux command from the SD prompt. `!` is the same verb under a
shorter name. `OS.EXECUTE` is the way from a program, and a program can also
reach the operating system by naming a file or directory outside the account.
**All of it is off as delivered**, and the administrator turns it on, in two
independent switches, with `set.os.access`.

## The two switches

| Switch | What it lets the account do |
|---|---|
| **`sh`** | the `sh` verb and `!` |
| **`os`** | `OS.EXECUTE`; `OPENSEQ`, `OPENPATH`, `DIR()`, `DELETESEQ`, `OPENTEMP`, `DIGEST` of a file and the file-system keys of `OSPATH()` on a path **outside the account's own directory**; a print unit's pathname (`SETPTR ... AS PATHNAME`, `SETPU`, a CSV report's `TO` pathname) and its `SPOOLER` command outside it; the embedded Python; the editors `nano` and `micro`; and a VOC pointer to a file outside the account |

**Both are off as delivered, on a new install and on an upgrade unless you said
yes when the installer asked.** The account's own directory is never asked
about: inside it everything works whatever the switches say. **The
administrator needs no switch**: after `admin` the session passes both for as
long as it lasts.

**What they run as, when they are on:** your Linux user, with your own
permissions. SD never runs as root, so nothing it starts can be root either. A
command that needs `sudo` fails as it would in any script run without a terminal
to ask for the password. Turning a switch on is therefore not a small thing:
whoever signs in to SD, at the keyboard, over ssh or through the API, can do
what your Linux user can do.

## Looking and changing: `set.os.access`

An administrator command (type `admin` first):

```
:set.os.access
Account sduser has SH no, OS.EXECUTE no
:set.os.access sh on
Account sduser has SH yes, OS.EXECUTE no
:set.os.access os on
Account sduser has SH yes, OS.EXECUTE yes
:set.os.access sh off
Account sduser has SH no, OS.EXECUTE yes
```

With no argument it shows the state and changes nothing. **One switch, one
action, per command**: a command that names both, one word, or anything but `sh`
or `os` followed by `on` or `off` prints `Syntax: set.os.access ...` and changes
nothing. Saying it a second time says `sduser already had that access; nothing
changed`. The answer always gives **both** switches, because `sh off` says
nothing about `os`. The change is written to the audit trail
(`set.os.access sh on account=sduser`), and it is kept in fields 7 and 8 of the
account's register record (`yes` is on; anything else is off, and a record with
fewer fields is off).

**A change is seen at the next sign-on, never by a session that is already
running**: sign out and in again, or open a new ssh or API session. The API
server reads the switches when it enters the account, as the sign-on does.

## All the gates on one page: `tool.gates`

```
tool.gates
```

**A full-screen page, in any session** (140 columns by 43 lines at least). It
lists the four gates this account has - **Shell** (`sh`), **OS.EXECUTE** (`os`),
**Remote files** (`net`, [API access](09-api-access.html)) and **AI**
([The AI helper](24-the-ai-helper.html)) - with, for each: whether it is ON or
off **for this session**, what the register holds (shown to the administrator
only), how it is delivered (off, every one), whether it reaches beyond SD (every
one does) and when a change reaches you. A box under the list explains the row
under the cursor in plain sentences: what ON lets the account do, what it can
still do with it off, and the command that changes it.

| Key | What it does |
|---|---|
| Up, Down, Page Up, Page Down, Home, End, a click | move to a gate |
| Enter, the space bar (**after `admin`**) | turn the gate on the cursor's row ON, or off, through its own verb |
| F5 | ask the system again |
| F1, ?, a click on the [?] | explain the page |
| Ctrl-Q | leave |

**What the columns say.** *This session* is what the system holds for the
session you are typing in: `sh`, `os` and `net` were loaded when you signed on,
so a change made later shows there **after the next sign-on**, which is why the
*Register* column is there (what the next sign-on will load). The AI's gate is
the helper's own setting, looked at at every request, so the two columns agree;
if the helper is not running both show a dash and the row says so. The page
**asks and does not try**: it never tests an operation to find out whether it
works.

**What Enter does (administrator only).** It builds one command - `set.os.access
sh on`, `set.os.access os off`, `set.net.access on`, `set.ai gate on` and so on -
shows it in a box with what it will do, and runs it only when you answer Run;
Cancel is what Enter chooses. **Turning a gate ON asks you to type its name**
(`sh`, `os`, `net` or `ai`), because every one of them reaches beyond SD, and the
box says that a gate off as delivered is being turned on and that leaving it off
is the tighter choice. The verb asks for the administrator itself and writes its
own audit line (`set.os.access sh on account=sduser`); the page writes only the
line that says it was started (`tool start app=tool.gates`), and only for the
administrator. After the command it reads again and shows the verb's own last line
in the status bar. **There is no key that turns everything on, and nothing is
on as delivered.** A session that has not typed `admin` sees the page and cannot
change anything: Enter says so and opens no box.

## What a refusal looks like

With both switches off:

```
:sh echo hi
sduser is not permitted to use the operating system shell
:! echo hi
sduser is not permitted to use the operating system shell
```

A program's `OS.EXECUTE` is a run-time error, and a path outside the account is
the `ELSE` branch with status 3035 (the number of "permission"; it is the same
number as the VOC refusal, and the statement says what was refused):

```
<address>: sduser is not permitted to use OS.EXECUTE at line 2
```

```
   openseq '/etc/hostname' to f then crt 'opened' else crt 'refused ' : status()
refused 3035
   x = dir('/etc') ; crt dcount(x, @fm) : ' ' : status()
0 3035
```

`DIGEST` of an outside file, `OSPATH()` and `OPENTEMP` give nothing and status
3035; a print file or a `SETPTR ... AS PATHNAME` outside the account says
`sduser has no OS access, so a file or directory outside its own account cannot
be reached by pathname (the administrator can give it: set.os.access os on)`;
`nano` and `micro` say that they need OS.EXECUTE permission. `ed` and `edit`, SD's own line editor,
need none of it.

## The two verbs

```
sh command
! command
```

With `sh` on, everything after the verb is handed to the shell as typed:

```
:sh echo hello-from-the-shell
hello-from-the-shell
:! echo via-the-bang-form
via-the-bang-form
:sh pwd
/home/you/SDCoreSolo/user_accounts/sduser
```

**The shell is `bash -c`.** The command starts in the account's directory,
`~/SDCoreSolo/user_accounts/sduser`, and runs to completion with its output
shown; there is no terminal for it to take over. Your PATH is the one SD was
started with — under systemd that is the user manager's, which normally does not
include `~/.local/bin`, so `sd-solo` itself is not found by name inside `sh` (use
the full path).

## A filter on the command, not on you

**Both verbs refuse a command that contains a shell metacharacter or a line
break** — `;` `|` `&` `$` `` ` `` `<` `>`:

```
:sh echo alpha-beta | grep alpha
Error 2 executing operating system command
:! echo hi; ls /somewhere
Error 2 executing operating system command
```

**That is a sanitizer, not a permission.** It stops a one-line command being
turned into two by accident — a record id or a variable pasted into it — and it
is applied the same way to everyone. A plain command with arguments is fine:
`sh ls -la`, `sh grep -c error /var/log/messages`. **A bare `sh`, with nothing
after it, is refused the same way**, so there is no interactive shell to be left
in from TCL: to get a shell, use your own terminal.

## `OS.EXECUTE` is not filtered

**A program's `OS.EXECUTE` runs the string it is given as written**, pipes and
all, with the same rights as `sh`, once the `os` switch is on:

```
   os.execute 'echo alpha-beta | grep alpha'
   crt 'status=':status()
```

```
alpha-beta
status=0
```

That is the way to run a command SD's filter refuses. **The form `execute 'sh …'`
from a program goes through the `sh` verb itself**, and is filtered, and asked
about the `sh` switch, like a typed one.

## The editors

The full-screen editors `nano` and `micro` run as ordinary Linux programs, in
your terminal, with your permissions, and are **behind the `os` switch** (they
are a way out of SD); `ed` and `edit` are SD's own line editor and are not. See
[Development and file commands](07-programmer-commands.html#editors).

## Over ssh and the API

**The switches are loaded the same way for a remote session.** An ssh or API
session signs in to the account, and the sign-on reads the register: with a
switch off, it is off there too. With one on, anyone who has the account
password (and, over ssh, your key or Linux password) can run Linux commands as
your Linux user. That is one reason every session asks for the account password;
see [Security](12-security.html).

## At install and at upgrade

**The installer asks**, once, on a terminal, with the default no for each:
`Turn on the shell (sh and !) for your SD account? [y/N]` and `Turn on OS access
(os.execute, outside files, Python, nano, micro) for your SD account? [y/N]`. An
upgrade asks the same, because **before this release the shell and OS access
were always on**, and an existing Solo would otherwise lose them without a word;
say yes to each you want to keep. The answer can be given on the command line,
`--os-access none|sh|os|both`, or in a control file (`os-access=`); with no
terminal, or with `--yes` and no answer, nothing is turned on here, which on a
new account is off and on an upgrade leaves the switches as they were.

## On a managed computer

**The SD Core for Linux server can put `sh` on the list of denied commands** (see
[Managed mode](15-managed-mode.html)). It takes `!` with it, because SD denies a
verb by what it runs and both are the same command, and both then need `admin`
first. That list is separate from the switches: a denied `sh` needs `admin` even
when the switch is on, and `set.os.access` is itself an administrator command.

## The one bypass: showing a document with `pdf`

```
pdf
pdf name
set.viewer
set.viewer add pdf program
set.viewer remove pdf program
tool.viewers
docs.install directory
load.help
```

**`pdf name` shows a documentation PDF in a PDF reader, whatever the `sh` and `os`
switches say.** It is the one place where SD starts a program outside itself for an
account that may not run programs, so it is held narrow and it is **off as delivered**:
until you name a reader, `pdf` says that no viewer has been allowed and starts nothing.
The installer asks once, if it finds a known reader (evince, okular, zathura, atril,
xreader, mupdf, qpdfview, xpdf), whether to allow it; the answer is **No** unless you
type y (`--viewer NAME` or `--viewer none` on the command line answers it for an
unattended install).

**What `pdf` starts, and with what.**

* **The program** is the first one on *your* list for `pdf` that passes the test below
  **at the moment it is started**, not only when it was added. The list is the file
  `viewers` in the SD system directory, one line to a program:
  `pdf /usr/bin/evince`.
* **The document** is a regular file of the `pdf` directory of the SD system directory,
  found by its name in lower case letters, digits, dot, dash and underscore. The caller
  never supplies a path; a link or a directory in that directory is not a document.
* **The command** is built from those two parts alone, each in quotes, started in the
  background with no input or output of SD's, so a reader that stays open does not hold
  the session. The reader is given the document as its one argument.
* **A session with no graphical display** (`DISPLAY` and `WAYLAND_DISPLAY` both empty, as
  over plain ssh) is told so, and nothing is started.
* **One audit line**: `pdf open file=name viewer=/usr/bin/evince`.

**The test a program must pass** (made by `set.viewer add` and again by `pdf`): an
absolute path of letters, digits and `/ . _ + -` with no `..`, not under `/tmp`,
`/var/tmp`, `/dev`, `/home`, `/run`, `/proc`, `/sys`, `/mnt` or `/media`; an existing
regular file (a link is followed to its target); owned by root; without the setuid,
setgid or sticky bit; not writable by group or others; executable by everyone. That is
the test `sudo` makes of what it runs, less the parent directories. **It does not say a
program is safe.** A graphical reader can open other files its user may read and follow
links, so allow only readers you trust, and never a shell or an editor.

### Naming a reader: `set.viewer` and the Bypass programs page

```
set.viewer                           says which programs are allowed
set.viewer add pdf evince            allows it (a bare name is looked for in /usr/bin, /usr/local/bin, /bin)
set.viewer add pdf /opt/x/bin/reader an absolute path works too
set.viewer remove pdf evince         stops allowing it
```

An administrator command (`admin` first; without it: *Command requires administrator
privileges*). **`tool.viewers`**, also an administrator command and a tile (*Bypass
programs*) on the administrator's home screen, is the same list as a page: *Allowed for
pdf* (each program and whether it still passes the test) and *Found on this machine* (the
known readers that are installed, whether each is allowed and whether it could be). Like
the menu management page it **writes nothing itself**: Enter on a reader shows the command
(`set.viewer add pdf evince`) with its effect in a box answered Run or Cancel (Cancel is
what Enter chooses), allowing one **asks for its name to be typed**, and the line under
the page is the verb's own answer. A program on the list under a path outside the three
directories is shown and removed at the prompt, because the commands the page may run
carry no slash. F1, `?` or a click on a `[?]` explains each box.

### Where the documents come from: `docs.install`

The PDFs are generated from the documentation and are not part of the source tree, so
they reach the `pdf` directory by the administrator:

```
docs.install /path/to/the/pdf/files
```

copies every `*.pdf` of that directory into the `pdf` directory of the SD system
directory (which the installer makes). A file is copied only if its name is lower case
letters, digits and `. _ -`, ends in `.pdf` and does not begin with a dot, and the file
begins with `%PDF-`; anything else is skipped and named. A document already there is
replaced; the copies are mode 0644. `pdf` alone lists what is installed. `docs.install` is
an administrator command (`admin` first).

### The context help: `load.help`

The `[?]` button on the administrator's pages shows help kept in the file `context.help`
of the SD system directory, one record to a setting. The installer fills it from the text
under `helptext` there; `load.help` does the same by hand, after that text changes. **It
replaces the shipped records and keeps any you added** under another id (the count says
how many). Reading the help is not gated: the text is the product's own documentation of
its screens.

## See also

[Administrator commands](06-administrator-commands.html) ·
[Sessions and locks](06a-sessions-and-locks.html) ·
[Security and the operating system](12a-security-and-the-operating-system.html).
