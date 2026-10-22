Title: API access
Subtitle: The client API, the port it answers on, its login, and what an API session is.

**The client API lets a program on this computer, or another one, use SD as a
back-end data store.** Application code uses `SDConnect()` and the rest of the
client library as with any SD Core — see [Client distribution](10-client-distribution.html).

## Is it on?

| | |
|---|---|
| **Any computer** | only if you chose `local` or `open` when installing (`--api`, or `api=` in the control file). Off by default |

**Off means there is no listener at all**: the installer writes no socket unit,
and SD opens no port. **A computer that an SD Core for Linux server manages (one
with a global password) needs the API `open`, because the server connects through
it**; the installer warns you if you give it a global password and choose less.
Until 6 October 2026 a managed computer's API was forced open and not asked.

## Signing in

| | |
|---|---|
| **User name** | always `sduser` |
| **Password** | the account password — or, on a managed computer, the global password, which the server uses |
| **Account** | `sduser` |

**The login is SCRAM-SHA-256, inside TLS 1.3.** The password is never sent in
any form; the server sets a challenge only someone who knows the password can
answer, and then proves itself back, so a program that grabbed the port before
SD started cannot collect passwords by pretending to be SD.

**The client library pins the server's certificate (LS1.1-2).** The first
connection from a client to an address and port is trusted, and its certificate is
remembered — the SHA-256 of the whole certificate — in `~/.sdcore/known_servers`
(or the file named by the environment variable `SD_KNOWN_SERVERS`), one line,
`<host>:<port> <64 hex digits>`, per server. Any later connection to the same
address and port that presents another certificate is **refused before a single
byte of the login is sent**, with a message that names both fingerprints and the
line to remove. A reinstalled computer has a new certificate: remove its line,
connect again, and the new certificate is remembered. **A store the client cannot
read or write refuses the connection** rather than connecting unchecked. This is
done in the client library, so it applies to every program that uses `SDConnect`.
**The very first connection is trusted**, as with ssh: someone in the middle at
that moment would be remembered instead of the computer.

**A managed computer's server can ask this computer to install its ssh key** over
the API, once it has signed in with the global password: see
[Managed mode](15-managed-mode.html).

**A client that sends a password in clear is refused** with *"Cleartext login is
no longer supported; this server requires SCRAM authentication"*.

**A wrong password is refused**, and the refusal is written to the audit trail —
for example `api refused user=sduser reason=wrong password`. It is written
**before** the three-second wait SD adds to slow down guessing, so a program that
hangs up during the wait is recorded as well.

**Known issue: `TLS read failed`.** A refused login made through the C client
library can report `TLS read failed` instead of *Invalid username or password*.
The login is refused either way and nothing else is affected. It has been seen
only on test computers running under VirtualBox (Fedora and Debian guests), and
not on Windows.

**The only account an API session may enter is `sduser`.** Asking for SDSYS, or
any other name, is answered *User not allowed in requested account* — the same
wording for an account that does not exist, so the API cannot be used to find
out what accounts there are. **The administrator password is not an API login**:
it is refused.

**On a computer installed from a control file**, until the account password has
been chosen at the keyboard, only the global password is accepted.

**`SDConnectLocal` is disabled.** It sends no password, which Solo does not
allow. A client that calls it gets, at once, *SDConnectLocal is not available in
SD Core for Linux Solo - connect with SDConnect and the account password*. Use
`SDConnect` to this computer's own address instead (`127.0.0.1`).

## The port

**The API is a systemd socket unit**, `sd-solo-api.socket`, listening on TCP
port 4249, which is fixed: there is no option to move it. (SD Core for Linux
uses 4247, so the two can share a computer.) **Each connection starts one SD session**
(`sd-solo-api@.service`), so the API works whether or not anything else is
connected.

| | |
|---|---|
| `local` | listens on `127.0.0.1` — this computer only |
| `open` | listens on `0.0.0.0` — every address the computer has. A firewall on the computer must allow the port; the installer opens it with `ufw` if `ufw` is running, or with `firewall-cmd` if firewalld is (Fedora), and says so if it cannot |
| `off` | no socket unit |

**To change it afterwards**, run the service script again with the new choice —
it stops the old listener first:

```
bash ~/SDCoreSolo/tools/solo-service.sh install ~/SDCoreSolo --api open
bash ~/SDCoreSolo/tools/solo-service.sh install ~/SDCoreSolo --api local
bash ~/SDCoreSolo/tools/solo-service.sh install ~/SDCoreSolo --api off
```

Each of those was measured: `open` listens on `0.0.0.0`, `off` leaves nothing
listening and the unit not active, `local` is `127.0.0.1` again. **An `open` API
on a computer with a firewall still needs the firewall told**; that part is
yours.

> **`APILOGIN` is not an off switch.** It decides whether the API demands a
> password. `APILOGIN=0` is the **weaker** setting, not the safer one. Do not
> reach for it.

**Linger matters here too.** Without it the socket, like SD, stops when your last
session ends; see [Running SD](03-running-sd.html).

## What a connection costs, and how big a request may be

**Each API connection is one `sd-solo -n -q` process, running as your Linux user
(never root), and it is there before anybody has logged in.** Since 20 October
2026 it ends when the connection ends, and it refuses a request whose declared
length it cannot accept. Before that date a client that sent four bytes that read as a huge
length (`garb` is 1,651,663,207), or that declared a request and closed before
sending it, or that closed while its own command waited in a shell and then
cleared its input, could leave that process polling at **100% of a processor**
until somebody killed it. *Measured on Solo (20 October 2026, a real TLS and SCRAM
session):* 101 ticks of CPU in a second, for one of four `garb` clients, three of
four that declared 1,000,000,000 and had no more to send, and the client that
closed while its command waited in `SH sleep 2`; none after the change.

Every request starts with a 4-byte length that counts the whole packet, itself
included, and a 2-byte request code, so the smallest packet is 6 bytes. The
client library never sends one that breaks the rules, so a program using it does
not meet them; they matter to a program that speaks the protocol itself, and to
anyone probing the port.

| When | The length of a packet may be |
|---|---|
| before the client has logged in (the password proof not yet accepted) | 6 to 65,536 bytes. The login messages are all under 4 KB |
| after the login | 6 to 1,073,741,832 bytes: the largest SD string (1,073,741,822) plus the 10 bytes of header the client library allows for |

A packet that declares anything else (below 6, negative, or over the limit that
applies) is not read: the server ends the session at once and writes one line to
the system log (`API packet length ... refused (before login)` or `(after
login)`). The same test applies to the reply a client sends to an `INPUT` prompt.
*Run:* a first packet declaring 0, 3, 5, -1, -2147483648, 2147483647, `garb` or the
largest string plus 11 is refused and the session ends with the client still
connected; so is a reply to an `INPUT` prompt that declares `garb` or 5.

**How the server knows the login has happened, and how that differs from SD Core.**
SD Core knows "not yet logged in" because its server is still root until the
client has logged in. Solo's server is never root, so it keeps a flag that is set
when the password proof has been accepted (when the server gives the session its
user name); the limit before the login applies until then. This is a difference
from SD Core, made with the owner's permission on 20 October 2026. *Run, before
the login:* a first packet declaring 65,536 bytes is waited for (the boundary),
65,537 and 1,000,000,000 are refused and the session ends. *Run, after the login:*
a record of 3,075,000 bytes is written through the API and read back byte for byte,
and a packet declaring 1,000,000,000 is waited for. Before this change Solo applied
no limit before the login. The log line's wording *before login* was not read
from a log in this run; it is the source's.

**A file lock goes to the file the client names (20 October 2026).** An API
client that had opened three files and asked for the file lock on the first
locked the last one it had opened, and a release freed that one; the form that
waits for the lock waited for the wrong file too. *Run, with two sessions of the
account:* each lock and each release now acts on the file number asked for, and
a request with nothing to say no longer repeats the text of the last refusal.

## An API session is you

**It runs as your Linux user**, in the account `sduser`, with the same rights as
a session at the keyboard — and the same two switches: the API server reads
`sh` and `os` from the account's register record when it enters the account, so
`sh` and `OS.EXECUTE`, and a path outside the account, are **refused to a remote
client while the switches are off, as delivered**, and usable by it once the
administrator has turned them on (`set.os.access`; a client that is already
signed in sees the change at its next login). See
[Operating system access](06b-operating-system-access.html). Records an API
session creates are yours, and if your Linux user may not read something, the
API session may not read it either.

**A session signed in with the global password is a server session**, with the
administrator commands unlocked. See [Managed mode](15-managed-mode.html).

## Request 29, the NET gate and `NETFILES`

**Request 29 opens an `account;file` file for a client** — the one door of SDNet
that is still in the server. **It is shut as delivered, and two things have to be
turned on, by the administrator, before it opens anything:** bit 2 of `NETFILES`
in `sd.conf` (`NETFILES=2`; the default is 0) and **the account's NET gate**,
which `set.net.access` turns on (`set.net.access on` after `admin`; with no
argument it shows where the gate stands). The gate is off as delivered, is kept in field
15 of the register record, is read when the request is asked (so a change reaches
a client that is already signed in), and only the word `yes` means on.

With both on, the request still refuses, with no file number and a line in the
audit trail that says why: when `SDCLIENT` is not 0 (as an ordinary open is);
when the account is named by a **path** (a path names no register record, so
nothing says who may enter it); when it names **another account or SDSYS** (the
one account a session may enter is its own); and when the account is
**suspended**. A refusal of the account rule is *User not allowed in requested
account*, whatever the reason, as for request 3.

**Where the file is.** The request asks the account's `os` switch
(`set.os.access os on`) as `dir`, `openseq`, `openpath` and the OSCOPY verbs do
(changed on 21 October 2026, carried from SD Core's change of 20 October; until
then it was a documented limit of this page). A file the account's VOC names with an
F-pointer must lead **inside the account's own directory**, with `..` and symbolic
links resolved, unless the account has the `os` switch on: a pointer at an absolute
path outside the account, a relative path that climbs out, a link to the outside, the
directory next door whose name merely begins with the account's own, a system
directory, or a dictionary half that leads out (though its data half is inside), is
refused. The refusal is error ELSE with status 3035 (permission), no file number comes
back, and the audit trail gets `api refused request=29 reason=os gate off user=... account=... path=...`.
The switch is read from the register record when the request is asked, so taking it off
reaches a client that is already signed in. The pointers SD itself puts in the VOC
(`syscom`, `newvoc`, `sd.voclib` and the like) lead out of the account and are refused here
too: a client of request 29 has no use for them. A pointer that opened before and leads out
of an account whose `os` switch is off stops opening: turn the switch on if that was
intended. Solo has no SDSYS exemption, because SDSYS is never a target of the request.
Run in a private sandbox through a real API login (`sandbox-req29.py`, section X); a
real client on another machine was not run. **Before 20 October 2026 the bit was the
whole gate**, and a site that had set it let an API login open a file named in the VOC of
any directory that holds a `voc`.

**Not measured on Solo:** whether an API session is confined to the account's own
directory as the multiuser product's is. The multiuser API refuses `OPEN` of
anything outside the account, distinguishing a containment refusal from a missing
file; Solo runs the same login code, but that containment has not been tested on
it, so do not rely on it as a boundary. `SDCLIENT` in `sd.conf` is the
configurable control: `1` refuses `CALL` and `EXECUTE` from a client, and `2`
allows a call only to a subroutine compiled as callable from a client (`0`, the
default, allows everything).

## Client libraries

| | |
|---|---|
| The shared library | `sdclilib.so`, in `~/SDCoreSolo/bin` |
| Source | built as part of this release, or standalone at <https://github.com/dmontaine/linuxsdclilib> |

**Use a client library from this release or later.** One that predates SCRAM is
refused. See [Client distribution](10-client-distribution.html).

**Programs using the `!sdclient` class** — SD BASIC reaching another SD over the
API — speak the same login, from this release at both ends.
