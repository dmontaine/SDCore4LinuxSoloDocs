Title: Showing a Document with pdf
Subtitle: The one bypass of the os switch: pdf, naming a reader, where the documents come from, and the context help.

This page continues [Operating System
Access](06b-operating-system-access.html).

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

A record that has more in the documentation names the document on its second line, as the
`pdf` directory holds it: the book first, then the chapter (`user-29a-sd-tcl-the-home-screen`;
the books are `user` and `gettingstarted`). The Documentation button runs `pdf` with that name.

## See also

[Administrator commands](06-administrator-commands.html) ·
[Sessions and locks](06a-sessions-and-locks.html) ·
[Security and the operating system](12a-security-and-the-operating-system.html).
