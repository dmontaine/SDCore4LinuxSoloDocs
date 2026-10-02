Title: Start here
Subtitle: What SD Core for Linux Solo is, what this set covers, and what it deliberately leaves out.

You already know MultiValue. This set does not teach it.

**SD Core for Linux Solo is a personal SD for one Linux user.** Everything it
has — programs and data — lives in one directory in that user's home,
`~/SDCoreSolo`, and it runs as that user. It has **one SD account, `sduser`**,
and no way to make another. It never runs as root and refuses to start as
root. Apart from that it has every feature of the multiuser SD Core for Linux
it was made from.

SD Core is a version of SD, with elements found in the main SD version and in
ScarletDME. ScarletDME was a fork of the original GPL release of OpenQM 2.6.6.

**That lineage matters when you go looking for documentation.** Not all the
features of the **commercial** OpenQM 2.6.6 were in the GPL release, and **no
documentation specific to the GPL version was ever released**. So even the
OpenQM documents are not authoritative here.

**The OpenQM 2.6.6 documents can be used as a reference**, but SD Core has
additions, changes and deletions — of features, of structure, of security and
of commands. **These pages cover those changes.**

## Two ways to use it

**The installer asks which, and the answer is fixed until you reinstall.**

| | |
|---|---|
| **Standalone** | a single-user database on this computer, in the way SQLite is. Nothing else manages it |
| **Managed** | a local database that an **SD Core for Linux** server also manages — one of several computers it looks after. The server signs in with a **global password** set when this computer was installed |

Only an SD Core for Linux server manages a Solo computer. What the server can
do today is on [Managed mode](15-managed-mode.html), and it will grow as
management features are added to SD Core for Linux.

## What "used to" means in these pages

These pages describe changes, so **used to**, **no longer** and **now** run all
through them. **The comparison is against the multiuser SD Core for Linux
L1.1-1 this release was made from** — see
[Differences from multiuser SD Core for Linux L1.1-1](01b-differences-from-multiuser-l1-1-1.html)
— and, behind it, upstream `sdb64`, ScarletDME and OpenQM 2.6.6.

## The pages

**Read them in numerical order the first time; after that they stand alone.**
**A number with a letter after it continues the page before it**, split so
that no page runs longer than a reader will scroll.

| | | |
|---|---|---|
| **00** | Start here | this page |
| **00a** | [Copyright and licence](00a-copyright-and-licence.html) | The copyright and the licence for this set, in full and in one place |
| **01** | [Installing](01-installation.html) | What the installer asks, what it puts where, and the control file for installing many computers |
| **01a** | [Upgrading and uninstalling](01a-upgrading-and-uninstalling.html) | Installing a new release over an existing one, and taking SD off the computer |
| **01b** | [Differences from multiuser SD Core for Linux L1.1-1](01b-differences-from-multiuser-l1-1-1.html) | What Solo leaves out, adds and does differently |
| **02** | **[Your first thirty minutes](02-first-run.html)** | **Start here if you just want it working** |
| **03** | [Running SD](03-running-sd.html) | How SD starts, stopping it, and the command line |
| **04** | [Scheduled jobs](04-scheduled-jobs.html) | Running an SD command on a timer |
| **05** | [The account and its passwords](05-account-types.html) | `sduser`, the account password, the administrator and global passwords |
| **06** | [Administrator commands](06-administrator-commands.html) | The commands that need `ADMIN` first |
| **06a** | [Sessions and locks](06a-sessions-and-locks.html) | Who is signed in, what is locked, and clearing it |
| **06b** | [Operating system access](06b-operating-system-access.html) | Reaching Linux from inside SD |
| **07** | [Development and file commands](07-programmer-commands.html) | Compiling, editing, and the verbs that maintain files and indexes |
| **08** | [ssh access](08-ssh-access.html) | Reaching SD on this computer over ssh |
| **09** | [API access](09-api-access.html) | The client API, its port, and its login |
| **10** | [Client distribution](10-client-distribution.html) | Which client library an application needs |
| **11** | [Lower case](11-lower-case.html) | Case in commands, file names and record ids |
| **12** | [Security](12-security.html) | What protects the database, and what does not |
| **12a** | [Security and the operating system](12a-security-and-the-operating-system.html) | Reaching the computer from inside SD, and the audit trail |
| **13** | [Other hardening](13-hardening.html) | The global catalogue, the logs, and the rest |
| **14** | [Not in SD Core](14-not-in-sd-core.html) | What has been removed, and what to use instead |
| **15** | [Managed mode](15-managed-mode.html) | What an SD Core for Linux server can do to a managed computer |
| **16** | [Configuration](16-configuration.html) | `sd.conf` and its parameters |
| **16a** | [System limits](16a-system-limits.html) | The sizes and counts SD works within |
| **17** | [The installed scripts](17-the-installed-scripts.html) | The scripts in the installed directory |
| **17a** | [The scripts SD runs itself](17a-scripts-sd-runs-itself.html) | The ones the installer and SD call, which nobody types |
| **18** | [Encryption and the SDEXT interface](18-encryption.html) | What encryption SD has, and the interface behind it |
| **19** | [Features the developers could not test](19-features-the-developers-could-not-test.html) | What nobody has watched work |

## The five things most likely to surprise you

**1. Every session asks for the account password.** At the keyboard, over ssh
and through the API. Being signed in to Linux is not enough. A command given
on the command line — `sd-solo list customers` — uses a copy of the password kept
for you in your own home directory, so a script or a scheduled job does not
have to type it. See [The account and its passwords](05-account-types.html).

**2. Administrator commands need `ADMIN` first.** Type `ADMIN` and the
administrator password (or, on a managed computer, the global password) and
they work for the rest of that session. See
[Administrator commands](06-administrator-commands.html).

**3. There is one account and you cannot make another.** It is called
`sduser` on every computer, whatever your Linux user is called. There is no
`sdsys` Linux user, no groups to join and no `sudo` helper. The account and
grant commands are gone. See
[Differences from multiuser SD Core for Linux L1.1-1](01b-differences-from-multiuser-l1-1-1.html).

**4. Commands and names are lower case, completely.** Typing in upper case
still works — SD converts it — but nothing can exist in two casings. See
[Lower case](11-lower-case.html).

**5. An ssh key can land inside SD, and the API login is SCRAM.** With a key
line the installer adds to your `~/.ssh/authorized_keys`, ssh lands at SD's
prompt, asked for the account password. Clients built against the old
cleartext API login will not connect. See [ssh access](08-ssh-access.html) and
[API access](09-api-access.html).

## What this release is

**LS1.1-2.** Linux only, English only, and not yet released. **It is a hobby
project with no release schedule.**

This set covers installing and running SD Core for Linux Solo, administering
it, managed mode, and what differs from the multiuser SD Core for Linux. The
reference for the language and the command processor is the separate User set.

## Where the source is

**Both repositories are public, and everything in them is open source.** SD is
GPL software — `config gpl` at an `sd-solo` prompt displays the licence, and the
installed directory carries it as the file `licence`.

| | |
|---|---|
| The server, the client libraries and the installer | <https://github.com/dmontaine/SDCore4LinuxSolo> |
| These pages | <https://github.com/dmontaine/SDCore4LinuxSoloDocs> |

**Neither repository contains a built binary, deliberately** — no executable,
no `.so`, no object files. A clone builds, and that is why installing means
building: the installer downloads the source and compiles it in your home
directory. The documentation repository holds the **Markdown only**; the HTML
and PDF you are reading are generated from it.

## Reporting what you find

**Open an issue on the server repository** —
<https://github.com/dmontaine/SDCore4LinuxSolo/issues> — for anything about SD
itself, and on the documentation repository for an error in these pages. If
you are not sure which, the server one is the right guess.

**Issues rather than pull requests.** Both repositories are writable only by
the author, so a change cannot be merged from outside; a clear issue is worth
more than a patch nobody can apply. A patch attached to an issue is welcome.

The two things worth reporting in most detail are **anything that behaves
differently from OpenQM and is not described here**, and **anything in these
pages that turns out not to be true of the build you are running**.

**Quote the version as `LS1.1-2`** — the string in the header bar of every page
here, in the sign-on banner, and in what `sd-solo --version` reports.
