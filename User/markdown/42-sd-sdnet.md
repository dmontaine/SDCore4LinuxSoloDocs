Title: SDNet - Files on Another SD Server
Subtitle: Open a file that lives on another SD server and use it with READ, WRITE, SELECT and LIST; what the administrator turns on, and what stays off.

**Read this first.** SDNet lets a program open a file on **another SD server** through an
ordinary `OPEN` and then use it with the ordinary file statements. In SD Core for Linux Solo
it is **delivered off, in four places**: the account has no NET gate, no server is defined,
nobody has a login on one, and no VOC entry points at one. The administrator turns each on,
deliberately, and each can be turned off again. **It has been run in private test
sandboxes against a test server; it has not yet been run between two real machines.**

Solo is a **client** of SDNet here, and it is also a **target**:

| | |
|---|---|
| Solo reaches a server (**Solo to Core**, **Solo to Solo**) | **limited**: a program may read, select and query the remote files; and, **only for a server the administrator defined `write`**, write, delete and lock. Nothing else crosses: no commands, no programs, no operating system paths. |
| A SD Core server reaches Solo | through Solo's API login, as any API client does; see *API access* in the GettingStarted set for what that login gives and for API request 29. |

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
- Select by index, and a server's own indices, are not used for a remote file.
- **Not confirmed on two real machines.** The paths above were measured in private sandboxes against a test server (the
  channel binding of the login is the one thing the test server cannot compute); a real Solo or SD Core server on the
  other end is the next measurement.

## See also

*API access* and *SDSYS and the administrator* (GettingStarted): `set.net.access`, API request 29 and the OS switch.
*Locks*, *Files and records* and *The query processor* (this set): the statements and verbs a remote file works with.
