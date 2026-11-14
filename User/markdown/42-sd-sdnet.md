Title: SDNet - Files on Another SD Server
Subtitle: Open a file that lives on another SD server and use it with READ, WRITE, SELECT and LIST; what the administrator turns on, and what stays off.

**Read this first.** SDNet lets a program open a file on **another SD server** through an
ordinary `OPEN` and then use it with the ordinary file statements. In SD Core Solo for Linux
it is **delivered off, in four places**: the account has no NET gate, no server is defined,
nobody has a login on one, and no VOC entry points at one. The administrator turns each on,
deliberately, and each can be turned off again. **It has been run in private test
sandboxes against a test server; it has not yet been run between two real machines.**

Solo is a **client** of SDNet here, and it is also a **target**:

| | |
|---|---|
| Solo reaches a server (**Solo to Core**, **Solo to Solo**) | **limited**: a program may read, select and query the remote files and their indices; and, **only for a server the administrator defined `write`**, write, delete and lock, and administer a file where the other machine allows it (below). Nothing else crosses: no other commands, no programs, no operating system paths. |
| A SD Core server reaches Solo | through Solo's API login, as any API client does; see *API access* in the GettingStarted set for what that login gives and for API request 29. A session signed in with the **global password** (managed mode) may also administer the account's files; one signed in with the account password may not. |

Commands are shown here in lower case. *Italics* mark something you supply.

## What has to be true before a program can open a remote file

All of these, checked in this order, before anything is dialled:

1. **The account has the NET gate.** `set.net.access on` (the administrator, after `admin`).
   The gate is **off as delivered** and a blank or missing field is off. It is the same gate
   that lets an API client open a file here with request 29.
2. **A server is defined** with `set.server`, by the administrator, and the account is named
   in the definition's accounts list (Solo has one account, `sduser`, so that is `accounts sduser`).
3. **There is a stored login** for that server: `set.netlogin`.
4. **The key the server presents is the one the definition pins.** A server that presents
   another key is sent nothing at all.
5. **A VOC entry names the file**: a record whose field 1 is `N`, field 2 the server's name and
   field 3 the file's name on that server. The administrator writes it (it is a VOC record).

The administrator is exempt from the gate and from the accounts list. Every other check applies to it.

## Defining a server: `set.server`, `list.servers`, `delete.server`

```
set.server name host port account {pin | connect} {read | write} {accounts a,b,c} {no.query}
list.servers {name}
delete.server name
```

The three are the administrator's: each stops with "Command requires administrator privileges"
for anyone else. A definition is one line of a file, `netservers`, in the SD system directory:
the name, the host, the port, the account to enter on the remote server, the hash of the
server's public key (the *pin*), the mode, the accounts that may use it and the date. **The file
never holds a secret**, and it is readable by whoever can read that directory.

| field | rule |
|---|---|
| `name` | a lower case letter, then up to 15 lower case letters or digits (upper case is converted) |
| `host` | a DNS name, an IPv4 address or an IPv6 literal; no one but the administrator ever chooses a host |
| `port` | 1 to 65535, the port of the other server's API: **4247** for SD Core, **4249** for a Solo |
| `account` | the account to enter on the remote server (1 to 32 letters, digits, `.` `_` `-`) |
| pin | exactly 64 hexadecimal digits, **or the word `connect`** (below) |
| `read` / `write` | what a program may do with the server's files. `read` unless you say `write` |
| `accounts` | the accounts here that may use the server. **Left out, none may**: the definition says so when it is made |

A definition that is wrong is refused with a message of its own for each field, and nothing is
written. At most ten servers can be defined. Replacing a name asks first; `no.query` skips that
question and **never the key question below**.

**The pin is the hash of the server's public key** (its SubjectPublicKeyInfo), not of its
certificate, so a certificate renewed over the same key keeps it. Two ways to get it:

- **Typed**: the 64 digits the other server's administrator gives you. Nothing is dialled.
- **`connect`**: every other field is checked first; then one TLS connection is made to *host:port*
  (it waits `NETWAIT` seconds), the hash of the key the server presents is **shown on a line of
  its own**, and you are asked `Trust this key (y/<n>)?`. Only `y` writes the definition. The
  connection is a handshake and sends nothing. **Compare the hash by a way that does not use
  this network**: if it differs, something may be standing between you and that server.

Each of the three verbs writes a line in the audit trail (`set.server name=...`, `delete.server name=...`),
which never carries the pin.

## Your login on a server: `set.netlogin` and `delete.netlogin`

```
set.netlogin server
delete.netlogin server
```

`set.netlogin` asks for the **user name and the password you have on the remote server** (the
password is typed hidden), runs the SCRAM-SHA-256 exchange with that server once, and keeps **the values
derived from the password**: the salt and the iteration count the server gave, the *ClientKey* and
the *ServerKey*. That is enough to log in later and not enough to learn the password or try it
anywhere else. **The password is never stored, printed, logged, audited or written**; it is held in one
variable and spent on the derivation.

- It needs the NET gate **and** the account named in the server's list, or the administrator.
  (SD Core refuses the administrator here for want of an operating system login of its own. In Solo the
  administrator is the same Linux user as the account, so it keeps a login too.)
- **Where it is kept**: one file for each server in `netlogins` in the SD system directory, mode 600, in a
  directory made mode 700 the first time it is needed. It is outside every account's directory, so a
  program whose **OS switch is off** cannot reach it with `openseq`. **Every use is preceded by a test
  that the file is a plain file, not a link, owned by you with no group or world bits, in a directory that is
  the same.** A file that fails the test is refused and nothing in it is used.
- **The honest limit.** A session that holds the `os` switch (or the `sh` switch) can read anything the Linux
  user can read, this directory included, exactly as it can read `~/.ssh`. Leave the switches off on an account that
  does not need them.
- **The key is checked first.** The key the server presents is compared with the definition's pin **before a
  single byte is sent**; a different key ends the command with nothing sent, and a login made for another key is
  refused when it is used.
- On any failure nothing is written and an existing login is untouched.

`delete.netlogin` removes it. Both write an audit line (`set.netlogin server=... remote=...`,
`delete.netlogin server=...`), with no password, key or hash in it.

## The VOC pointer

```
N
nfa
customers
```

A VOC record with `N` in field 1, the server's name in field 2 and the file's name on that server in field 3
(lower case; up to 63 letters, digits and `. _ - $`). Opening it, `open 'customers' to f`, gives a file variable
that these statements work on: `read`, `matread`, `readv`, `select`, `selectv`, `sselect`, `readnext`,
`fileinfo` (type 7), `close`, and `getlocks()`. `open 'dict', 'customers' to d` opens the dictionary of the remote
file from the same server, so `list`, `count`, `sort`, `select`, `list.item` and `list dict` work on the VOC name.
A `select` takes every record id of the remote file in one reply; each record a query then needs is one request, so
a query that reads every record of a big file costs one round trip per record.

**A pointer someone wrote for themselves gives them nothing they were not granted**: the gate, the list, the login
and the pin are asked every time, **an `$internal` program included**.

## Writing and locking: a server defined `write`

For a server defined `write`, a file opened through it also takes `write`, `writeu`, `matwrite`, `writev`,
`writevu`, `delete`, `deleteu`, `clearfile`, `readu` and the other locking reads, `recordlocku`, `recordlockl`,
`release`, `filelock` and `fileunlock`, and `recordlocked()` asks the server. A lock is the **server's**: it is held
there, for this session, and `release` or a `close` or the end of the process gives it up. A statement that changes
or locks a file of a server defined `read` is refused with message 11501 (or sets `status()` under `on error`) and
**nothing is sent**.

**`recordlocked(f, id)` needs the same two things as a lock** (changed on 21 October 2026, carried from SD Core's rule of
20 October): the server must be defined `write` here, and the login on the other machine must be allowed to ask. A
`read` definition stops the program with message 11501 (`recordlocked is refused`) and nothing is sent; the function has no
`on error` clause, so a refusal always stops the program. A SD Core server answers request 34, the one `recordlocked()`
sends, only to a login that is in its write group; to any other it replies with an error, and the program stops with
message 11500 and that machine's words, which end `(request 34)`. The answer names the holder of the lock by his user number
on the other machine, which is why a login that can only read is not given it. Inside `begin transaction` it still works
for a `write` definition, because it changes nothing. `getlocks()` of a remote file is the local view only.

- The mode is the **client's** setting, set only by the administrator. The server it points at is not told: **what the
  login may do there is that server's decision**. A SD Core server's files-only route lets a login write only if the
  administrator of that server has put the remote user in its write group, and refuses the change otherwise; the refusal
  comes back as an `on error` reply with the server's own text.
- A change is **never repeated** and **never made inside a transaction**: `begin transaction` with a change to a remote file
  stops with 11510. A connection that is lost *after* a change was sent says so (11509) and does not say whether the
  change was made: read the record to find out.
- A `read` can be up to **2 seconds old**: the process keeps the last 64 records it read from remote files for 2 seconds.
  A record over 32,768 bytes is not kept; `readv` is not cached; a write-mode handle is never cached.
- **Solo does not ask an "edit" question** (SD Core asks one for a remote file whose *name* says it is a VOC, a dictionary
  or a program file): Solo has no edit gates, so for such a name the other server's own gates decide.

## Administering a remote file

These do on the other machine what they do on a local file, **for your remote login and in the account that login entered
there**, so every switch that account has on that machine applies (on an SD Core server a file named `voc`, `*.bp` or
`*.dic` still needs that account's edit switch there). They need **a `write` definition here, and the other machine's
permission**: an SD Core server allows them only to a remote login its administrator has put in both its write group and
its administration group (off for every account until he does); a Solo server allows them only to a session signed in
with its **global password**, and refuses one signed in with the account password with *This session cannot administer
files here: sign in with the global password*. A Solo with no global password (the delivered state) allows none of them.

| | |
|---|---|
| `write rec to f creating.seqkey id`, `writeu ...` | the other machine's **own counter for that file** gives the id and it comes back in `id`; the update lock on the new record is held there for `writeu` and given back for `write`. A directory file is refused as it is locally (message 11348). Not inside `begin transaction` (message 11510) |
| `fcontrol(f, key {, value})` | goes to the other machine with its key (`FC$SET.SEQ.KEY`, `FC$GET.SEQ.KEY`, `FC$VALID.ID`, `FC$SYNC`, `FC$CRLF`, `FC$MINMOD`) and returns what it returned; `status()` is what it left there. Asking whether a name is a valid record id (key 6) changes nothing, every other key is a change |
| `analyze.file`, `configure.file`, `create.index`, `build.index`, `delete.index`, `disable.index` on the VOC name of a remote file | run **on the other machine**, as the remote login's Linux user; **what the verb printed there is printed here**, and the verb's return code is the one the other machine's verb ended with. The file is the one the pointer opened (you cannot name another), and the rest of the command is passed on as it stands |
| `create.file name ... server srv {as remote}` | makes the file on server *srv* (as *remote* there when `as` is given) and, **only after that machine has confirmed it**, writes the `N` pointer *name* in your VOC: there is never a pointer to a file that is not there. `dict`, `data`, `using`, `pathname`, a comment and `keepcase` mean nothing for a remote file and are refused (message 11528) before anything is sent; a name already in your VOC is refused (11539) before anything is sent |
| `delete.file name` where *name* is an `N` pointer | asks, as a local delete does (`no.query` does not), deletes the file on the other machine and, in the same act, the pointer here. If that machine says the file is not there (someone removed it) the pointer would dangle, so the verb offers to remove it (`no.query` removes it). If that machine refuses, the pointer stays |

**What the other machine accepts in the words.** Letters, digits and `. _ -`, single blanks, at most 200 characters.
`pathname`, `lptr`, `force`, `no.query`, a slash, a quote or two blanks in a row are refused with message 11530 and
nothing is run; `directory` is accepted only as the last word. The verb that asks questions is told not to (`create.file`
and `delete.file` get `no.query` there, since you answered here). The index verbs need the file to themselves: if another
process has it open on that machine the verb says "Cannot gain exclusive access" and does nothing, as it does locally.

**What it records.** The other machine writes `api admin request=70 op=create.index file=customers user=name` before it
runs the verb, and this machine writes `netfile admin server=nfa file=customers op=create.index` (and
`netfile fcontrol ... key=n`), so a killed command is still on both trails.

**Not available:** the statements that read or change the tree of an index (`akread`, `akwrite`, `akdelete`, `clear.ak`,
`akenable`, `akmap`) on a remote file are still refused, as are `openseq` and the other sequential-file statements. A
change is never sent twice: a lost reply is message 11509, and the program must look at the file to find out what
happened.

## Indices of a remote file

A remote file's indices are the other machine's, and the statements that *query* them work on a remote file the way
they do on a local one. They change nothing, so they need no write definition and no administration permission: a
program that may open the file may use them, on a read definition as well as a write one, and inside `begin transaction`.

| Statement | What it does on a remote file |
|---|---|
| `indices(f)`, `indices(f, name)` | the names of the indices of the file on the other machine, and the description of one (null for a name that is not an index) |
| `selectindex name from f to n` | the indexed values, as a select list; `status()` 1 and an empty list for an index that is not there |
| `selectindex name, value from f to n` | the ids that hold the value; `status()` 0 and an empty list for a value nobody holds |
| `setleft name from f`, `setright name from f` | put the scan at the left (lowest) or the right (highest) end |
| `selectright name from f setting key to n`, `selectleft ...` | the next key to the right (or left) of the scan, in `key`, with its ids as the select list; `status()` 1 and nothing when there is no further key |

The other machine keeps the position of a scan *for the open file*, so `setleft` followed by `selectright` walks the keys
in order, and `selectindex name, value` followed by `selectright` goes on from that value, as locally. Each call is one
request (a scan that finds a key makes a second one to fetch its ids), so a scan is as slow as the line between the
machines. A lost connection stops the statement with message 11497, as a lost `select` does; these statements have no
`on error` clause.

The `indices()` that every query asks of every file gets the other machine's answer, so `list`, `count` and
`select ... with` on a remote file use the other machine's index when the field in the `with` clause has one, and read
only the records the index names instead of every record. A file with no index is read as before.

## The two waits: `NETWAIT` and `NETREPLY`

Two `sd.conf` parameters, in seconds, 1 to 3600 (zero is refused): **`NETWAIT`** (5 unless you say otherwise) is how
long to wait for a connection to a remote server, **`NETREPLY`** (30) how long to wait for each reply to a request. `config('NETWAIT')`
and `config('NETREPLY')` read them.

**A server that goes silent is given up on at those times, in every case we know of** (changed on 21 October 2026, carried
from SD Core's change of 20 October). `NETWAIT` bounds the TCP connection and then the TLS handshake, each: a server that
accepts the connection and never answers the hello is given up on after `NETWAIT`, not after the 15 seconds that were fixed
before. `NETREPLY` bounds each read, including the first reply of the login after the handshake. Before this, a server that
completed the TLS 1.3 handshake and then sent nothing, or only half of a TLS record, held the session far beyond `NETREPLY`
(the server's session tickets, or the half record, ended the wait on the socket, and the read behind it then had no limit of
its own); with `NETREPLY=3` the test server was waited for 5 seconds and 12 seconds in those two cases, and 3 seconds now.
A reply that stops in the middle of its body is a lost connection, like a reply that never starts: message 11497 for a read,
11509 for a change (whose outcome is then unknown), and the connection is dropped. A server that answers late but inside
`NETREPLY` is read as before. The limit is for each read, not for a whole reply: a server that sends one byte at a time and
never stops is still waited for one read at a time.

## What is audited

| line | when |
|---|---|
| `set.server name=... host=... port=... mode=... replaced=... pin=typed|confirmed` | a definition is made or replaced |
| `delete.server name=... host=... port=...` | a definition is removed |
| `set.netlogin server=... remote=...` | a login is stored |
| `delete.netlogin server=...` | a login is removed |
| `netfile open server=... file=...` (and `mode=write`) | a remote file is opened, once |
| `netfile refused server=... reason=...` | an open is refused, with the reason (gate, list, login, pin, limit) |
| `netfile clear server=... file=...` | a `clearfile` of a remote file |

No password, key, hash, record id or record is ever audited.

## What is not here

- **No files-only route on a Solo server.** SD Core has one (the Linux groups `sdfiles` and `sdfileswrite`); Solo has one
  account and one Linux user, and its API login is the account's own login. API request 29 is behind the NET gate and
  `NETFILES` bit 2, as the Getting Started set explains.
- **A remote file does not follow you across an account change**: the process's connections are dropped when the account
  is entered again (the NET gate is set), so a remote file variable from before is stale.
- **Not confirmed on two real machines.** The paths above were measured in private sandboxes against a test server (the
  channel binding of the login is the one thing the test server cannot compute); a real Solo or SD Core server on the
  other end is the next measurement.

## See also

*API access* and *SDSYS and the administrator* (GettingStarted): `set.net.access`, API request 29 and the OS switch.
*Locks*, *Files and records* and *The query processor* (this set): the statements and verbs a remote file works with.
