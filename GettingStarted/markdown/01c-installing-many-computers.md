Title: Installing Many Computers
Subtitle: The control file for installing many computers, what the installer checks when it finishes, the editors and the mouse, and changing any of it afterwards.

This page continues [Installing](01-installation.html).

## Installing many computers: the control file

**A file passed as `--control-file` answers the installer's questions**, for an
install nobody sits at, and it is how one USB stick sets up several computers.
It is for any computer, managed or not: its presence does not make the install
managed — a global password in it does.

```sh
bash installsdsolo.sh --control-file /media/stick/sd-solo-setup.conf
```

| | |
|---|---|
| `admin-password=` | the administrator password |
| `global-password=` | the global password. **Blank or absent means none**: the installer says *The control file gives no global password, so this computer will NOT be managed by an SD Core server* and asks nothing |
| `api=` | `off`, `local` or `open` (port 4249). Blank takes the default, off; a managed computer needs `open` |
| `ssh=` | `off`, `local` or `open` (port 4251). Blank takes the default, `local`; a managed computer needs `open`. `off` with `ssh-public-key-file=` is refused |
| `deny-verbs=` | a comma-separated list of commands the user of the computer may not run without the administrator or global password. Only a session signed in with the global password can change the list afterwards, so on a computer with no global password it is fixed until a reinstall. See [Managed mode](15-managed-mode.html) |
| `ssh-public-key-file=` | a public key whose owner may ssh straight into `sd-solo`, on port 4251 (turns ssh on, `local`). `ssh-match=` is gone: a control file that still names it is warned about and the line is ignored |
| `os-access=` | `none`, `sh`, `os` or `both`: which of the shell and OS access to turn on. Blank asks on a terminal and means `none` without one; both are off unless named |
| `start=` | `menu` or `terminal`: where new sessions start, the home screen or the command prompt. Blank takes the home screen; an upgrade or a reload of kept data keeps what the installation had |
| `viewer=` | the name of a PDF reader to allow for the `pdf` command (`evince`, `okular`, `zathura`, `atril`, `xreader`, `mupdf`, `qpdfview` or `xpdf`), or `none`. Blank allows none |
| `enable-linger=` | `yes` or `no`. Blank is yes: `loginctl enable-linger` is run unless this says `no` (`--enable-linger` or `--no-linger` on the command line wins); anything else is refused |
| `reload-data=` | `yes` or `no`: what to do when the install directory holds data and configuration an earlier uninstall kept — use them, or move them aside and install clean. Blank asks on a terminal and reloads when there is none; it does nothing when nothing was kept. See [Upgrading and uninstalling](01a-upgrading-and-uninstalling.html) |

**The account password is deliberately not in it.** On a computer installed
from a control file **that gives a global password**, the user sets the account
password **the first time they run `sd-solo` at that computer's keyboard**.
Until then ssh and the API accept only the global password — the server can reach
the computer, and nobody else can. (Give `--account-password-file` as well if you
would rather set it at install time.) **With no global password in the file the
installer asks for the account password as usual**, so an unattended install of a
computer with no global password needs `--account-password-file`.

**A blank answer is asked for** (except `global-password=`, above, and `api=` and
`ssh=`, which take their defaults when nobody is at the keyboard), and one that
breaks the password rules is refused. The repository carries `sd-solo-setup.conf.sample`, which explains each
item and shows a sample answer, commented out, above the line for your answer;
copy it and fill it in. **A commented sample is never taken as an answer.**

**The file holds passwords in clear text.** Keep the stick safe, and do not
leave the file on a computer after installing.

## What the installer checks when it finishes

It signs in as `sduser` and runs `WHO`, and then confirms that **`sd-solo -internal`
is closed**: the door the installer itself uses is a one-shot, opened only by a
marker file the installer writes immediately before each of its own steps, and
it is shut when the install ends. The last line of a good install is:

```
SOLO INSTALL COMPLETE /home/you/SDCoreSolo
```

**If a step fails the installer stops and says which**, with the end of its
log. Nothing is put back — a failed first install leaves whatever it had made;
remove it with `deletesdsolo.sh` and start again.

## The full-screen editors

**The installer installs `micro` as an ordinary package**, because the
`micro` verb runs it. It is not offered as a choice — an editor verb that does
nothing is worse than either answer. **`ed`**, the line editor, and **`sed`**, the
full-screen editor with menus, are part of SD and need nothing installed; they
always work. `micro` is one of the packages the install requires: if it cannot be
installed, the install stops.

## The mouse on the Linux console

**The installer installs `gpm`**, the daemon that reads the mouse on a Linux text
console with no desktop, so that the full-screen pages, the IDE and the debugger
have a mouse there too. The Debian and Ubuntu package enables and starts the
daemon itself; the installer does not. **If `gpm` cannot be installed** (in a
container, for example) **the installer prints a warning and goes on**: SD runs
without it, and there is then no mouse on a bare console. Install it later with
`sudo apt-get install gpm`. Removing SD leaves the package installed. A session in
a terminal emulator or over ssh does not use it. See *SD Terminal Information*
in the User set.

## Changing any of it afterwards

| | |
|---|---|
| Whether there is a global password (managed or not) | a new installation: uninstall (keeping your data if you want it), then install |
| The API or ssh choices | uninstall and install again, or use the scripts in `~/SDCoreSolo/tools` — see [The installed scripts](17-the-installed-scripts.html) |
| The passwords | `set.password`, `set.password admin`, and on a managed computer `set.password global` from the server. See [The account and its passwords](05-account-types.html) |

## Continued in

[Upgrading and uninstalling](01a-upgrading-and-uninstalling.html) —
upgrading an existing installation, and uninstalling.
