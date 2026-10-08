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

## An API session is you

**It runs as your Linux user**, in the account `sduser`, with the same rights as
a session at the keyboard — including `sh` and `OS.EXECUTE`, which a remote
client can therefore use. Records an API session creates are yours, and if your
Linux user may not read something, the API session may not read it either.

**A session signed in with the global password is a server session**, with the
administrator commands unlocked. See [Managed mode](15-managed-mode.html).

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
