Title: SD Core - Introduction and Getting Started
Subtitle: What a multivalue database is, what SD is, the four components, and your first session.

This page orients you to SD Core for Linux Solo: what a multivalue database is,
where SD came from, what the pieces are, and how to take your first steps. It
is the only page in this set that assumes nothing.

## What is a multivalue database?

A multivalue database stores data in records made of **fields**, where
each field can hold **more than one value** — and each value can hold
**more than one subvalue**. A single field in a customer record can
therefore carry every phone number the customer has, without a separate
table or a join.

The model was designed by Dick Pick in the 1970s as the Pick Operating
System. It has been through PI/open, UniVerse, Unidata, D3, jBASE,
QM, and ScarletDME — and SD is one of its direct descendants.

The three delimiters that make it work are **field marks**, **value
marks** and **subvalue marks** — control characters that separate the
levels inside a single string. A dynamic array in SDBasic is a string
that carries these marks, and `extract`, `insert`, `delete` and
`replace` work on them directly.

## What SD is

SD Core for Linux Solo is built from upstream `sdb64`, a MultiValue database
with elements found in the main SD version and in ScarletDME. ScarletDME
was a fork of the original GPL release of OpenQM 2.6.6.

**That lineage matters when you go looking for documentation.** Not all
the features of the *commercial* OpenQM 2.6.6 were in the GPL release,
and no documentation specific to the GPL version was ever released. The
OpenQM 2.6.6 documents can be used as a reference, but SD Core has
additions, changes and deletions — of features, of structure, of
security and of commands. This documentation set covers those changes.

If you have used OpenQM, or upstream `sdb64`, much of SD Core will still be
familiar: the same data model, the same query processor, the same
BASIC.

**SD Core for Linux Solo is Linux only, and for one user.** It is the
single-user edition of SD Core for Linux: one account, `sduser`, installed in
your home directory and run as you. There are no `#ifdef` branches keeping
Windows alive in this source — SD Core Solo for Windows is a separate project,
kept in behavioural parity by deliberate policy, and this is not a build of it.

SD Core is free software under the GNU General Public Licence v3. `config gpl`
displays the licence and `config contrib` the list of contributors. Installing
means cloning the source and building it — see the GettingStarted set.

## The four components

| | |
|---|---|
| **The command processor (TCL)** | reads what you type at the `:` prompt and dispatches it to a verb, a program, a paragraph or a query |
| **The query processor** | runs `list`, `select`, `count`, `sort` and the rest — the reporting language |
| **SDBasic** | the programming language: a compiled BASIC with dynamic arrays, file I/O, and the multivalue string functions |
| **The SDClient API** | a C client library (`sdclilib.so`) that lets an external application connect to SD, read and write records, execute commands and call subroutines |

## Signing in

```
sd
```

**You land in the one SD account, `sduser`, after the account password** — the
one you chose when installing. Being signed in to Linux is not enough: SD asks.
If the computer was installed from a control file there is no password yet, and
`sd` asks you to choose one. The GettingStarted set's *Your first thirty
minutes* walks through it.

SD is already running. It is your own systemd user service, `sd-solo.service`,
so you do not type `sd -start`. Open a new terminal after installing: one that
was open before the install may not have `~/.local/bin` on its PATH yet.

## Your first file and record

```
create.file customers
ed customers 1001
```

`ed` is the line editor, and it needs nothing installed. In `ed`: `i`
to insert, type your lines, a full stop on its own line to stop
inserting, then `fi` to file and exit.

You can also use `nano` or `micro` (both full-screen editors; `micro`
highlights SD BASIC) — with nothing to unlock first. `edit` aliases `ed` here,
not a full-screen editor.

```
list customers
count customers
```

Commands are lower case now. Typing `LIST` still works — SD tries what you
typed, then lower case, then upper, and finally with any hyphens changed to
dots, so `clear-select` reaches `clear.select` too.

## Writing a program

A program lives in a `bp` file — a directory file, which is an ordinary
Linux directory with one file per program. You can write it in `ed`,
in `nano`, in `micro`, or in any text editor you like — the folder is on
disk at:

```
~/SDCoreSolo/user_accounts/sduser/bp
```

Compile and catalogue it from inside SD:

```
basic bp myprog
catalog bp myprog
```

Then run it by name:

```
myprog
```

## Administrator commands

**There is no separate administrator account.** A few commands — `config`,
`listu`, editing the VOC directly — refuse with *Command requires
administrator privileges* until you unlock them for the session:

```
admin
```

Type the administrator password chosen at installation, or, on a managed
computer, the global password. It lasts until you leave SD or type
`admin off`. See *Administrator commands* in the GettingStarted set.

## What is not in SD Core

The following were in OpenQM, in ScarletDME, or in upstream `sdb64`, and
are not in SD Core for Linux Solo:

| Gone | Why |
|---|---|
| SDNet (remote files) | Removed; the API is the supported way to reach another SD server |
| `ENCRYPT.FIELD` verb | Removed; `sdencrypt()` and `sddecrypt()` in SDBasic are the supported route |
| `sed`, `update.record`, `modify` editors | Gone; use `nano`, `micro` or `ed` |
| PROC language | Removed; use paragraphs instead |
| `SET.LANGUAGE`, `LOAD.LANGUAGE` | Removed; SD Core is English only (NLS, for currency and separators, is kept) |
| Accounts other than `sduser` | Solo has one account. `create.account`, `grant` and the rest are gone |

**Embedded Python is not on this list** — a real difference from SD Core
for Windows, which dropped and later restored a narrower form of it. This
port never removed it.

## Document conventions

| | |
|---|---|
| **bold** | a word typed as it stands |
| *italics* | something you supply |
| braces `{ }` | an optional part |
| `code` | a command, a function name, or something you type |
