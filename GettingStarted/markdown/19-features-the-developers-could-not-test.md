Title: Known Limits
Subtitle: Parts of SD Core Solo for Linux whose behavior depends on your environment, and what to check before an application relies on them.

Nothing on this page is known to be broken. Each entry is a place where what
happens depends on conditions SD cannot establish for itself: the load on
your machine, the network between two servers, the program you run beside
SD.
Treat the page as a list of things to try on your own system before an
application depends on them.

## How to read an entry

| | |
|---|---|
| **What to expect** | how the feature behaves, as far as SD controls it |
| **Check before relying on it** | what to try, on your own system |

## Installing and running

### Solo's own ssh port, `sudo` and linger

**What to expect.** Solo's ssh listener (port 4251) is run for your own user by
two systemd user units, one `sshd` for each connection. Your Linux password, or a
key in Solo's key file, reaches the forced command and a command of the client's own
is not run; a key that is not in the file is refused, and so is a wrong Linux password; forwarding
is refused. Three wrong passwords from one address in ten minutes lock that
address for ten minutes. A guess spread over many addresses is not stopped. With
`linger` on, which is the default, your user manager and SD stay up after your
last sign-out; with it off, nothing runs until you sign in.

**Check before relying on it.** Whether another computer can reach port 4251
depends on your firewall (`ufw` or `firewalld`) and your network. Open the port
only if you need to, connect from another computer, then sign out with linger on
and connect again from elsewhere. Try this once on a computer you can afford to
lock yourself out of. See [ssh access](08-ssh-access.html).

### An install from the published branch

**What to expect.** The installer builds from the published source, so what it
does depends on the network, the packages your distribution offers and the
distribution itself. It is written for Debian and Ubuntu.

**Check before relying on it.** Run `installsdsolo.sh` from a fresh clone on the
distribution you will use, before an unattended install on many computers.

### A first install from a control file, over ssh and the API

**What to expect.** A computer installed from a control file has no account
password until the user chooses one at the keyboard. At the console the first
password is chosen and the refusals work, and afterwards every route works. Over a
piped session, and a session that says it is over ssh, only the global password
is accepted until then. The API serves the global password's proof while no
account password has been set.

**Check before relying on it.** Install from a control file, then sign in over the
API with the global password before choosing the account password.

## The API

### The server's ssh key and the pinned certificate

**What to expect.** On a managed computer the server's key is installed,
repeated, listed, removed, capped at four and refused for the account password;
shell syntax in a key is not run; the installed key reaches `sd-solo` through
Solo's own `sshd` with the global password. The client library remembers a
server's certificate, accepts the same one again, and refuses a replaced one
before any login byte is sent; the remedy in its message works. See
[Managed mode](15-managed-mode.html).

**Check before relying on it.** The address a client connects from, the route
between two machines and the SD Core for Linux server itself are conditions of
your own site. Have a second machine sign in with the global password, ask for
its key to be installed, and then connect over ssh with that key; reinstall the
managed computer and confirm the client is refused until its pin is removed.

## Sessions

### `sd-solo` started from inside a running session

**What to expect.** A shell that SD starts with `sh` or `OS.EXECUTE` is marked as
SD's own, and `sd-solo` run inside it refuses to start: *SD is already running in
this session - type exit to return to it.* That is a guard and not a boundary: the
mark is a variable of the shell, and the user can remove it there. See
[Running SD](03-running-sd.html).

**Check before relying on it.** Do not build a procedure on the guard. A shell
that you start yourself outside SD is not marked.

## Locking and contention

### Semaphores under heavy contention

**What to expect.** A semaphore is used on every record lock, so ordinary use
exercises it all the time.

**Check before relying on it.** If an application will have many sessions waiting
for the same lock, try that load on your own machine first and watch how long
the waiters wait.

### A lock held by a terminal session and wanted by an API connection

**What to expect.** Two local sessions compete correctly, and the API's own
session-confinement gate is described in [API access](09-api-access.html).

**Check before relying on it.** If an application mixes terminal sessions and API
clients on the same records, try an API client and a terminal session competing
for one record.

### Task locks taken twice by the same session

**What to expect.** Taking a task lock you already hold succeeds, and one
`unlock` releases it however many times you locked it: the test is "unowned or
mine", and `unlock` clears the slot outright. A program that takes the same lock
twice and expects to hold it after one `unlock` will be wrong.

## Scheduled jobs

### An administrator's scheduled job

**What to expect.** The units are written and the run is recognized as
[Scheduled jobs](04-scheduled-jobs.html) describes; the next and last run that
`set.job` and `tool.jobs` show are read from `systemctl --user show`.

**Check before relying on it.** Make a job that runs every five minutes, let it
run once, and read its lines in the audit trail and
`journalctl --user -u sd-solo-job-`*name*; then delete it.

## SDNet: remote files between SD servers

### Anything between two machines

**What to expect.** The route works between a client and a server: a TLS 1.3
handshake, the pinned key, the SCRAM login from the stored keys, and reads,
writes, locks and deletes through a VOC pointer of type `N`. `NETWAIT` and `NETREPLY` bound
how long a call waits for a server that never answers or goes silent after its
acknowledgement.

**Check before relying on it.** A firewall in between, a name that has to be
resolved, real network latency, a connection that a router drops, and the key
hash carried from one screen to the other are conditions of your own network.
Set up the server and a client on two machines and try each of them, including
a server that hangs, before depending on remote files in production.

### `MUSTLOCK` on a remote file

**What to expect.** With `MUSTLOCK=1` a write to a file needs the record lock
to be held first, and a remote file follows the same rule.

**Check before relying on it.** `MUSTLOCK` applies to every session, so set it
on a scratch installation first, with a write to a remote file with and without
the lock held.

## Application data

### Large and long-established data

**What to expect.** SD creates, writes, reads and deletes files, records,
indexes, select lists and sequential files, and the system files it starts with
are ordinary files of the same kind.

**Check before relying on it.** Before moving a production application, restore a
copy of its account and run it: a file of hundreds of thousands of records, a
deep dictionary or a schema built over years by someone else is the case worth
trying first.

## Sockets

### UDP and ICMP

**What to expect.** TCP is the supported protocol: listening, connecting,
accepting, reading, writing, the blocking and non-blocking modes and the error
codes. The flags `0x00010000` and `0x00020000` for UDP and ICMP exist in the
language, but SD makes no promise about sockets opened with them.

**Check before relying on it.** Use TCP. If you need a datagram socket, try a
datagram to a listener and back first.

## The MCP door

### An agent's MCP client through the door

**What to expect.** The launcher `solo-mcp.py` speaks MCP on its standard input
and output, and the rules inside `set.mcp` and `mcp.call` decide what a call may
do. See [The MCP door](25-the-mcp-door.html).

**Check before relying on it.** Turn the door on with `set.mcp on`, add the
launcher to your own MCP client, call each tool once, then `set.mcp off` and see
the next call refused. The audit trail shows each call and each refusal.

## SD BASIC statements that depend on something outside SD

| | needs |
|---|---|
| `sendmail` | a mail relay configured on the machine |
| `chgphant()` | a phantom process to change |
| `ccall()` | a C function registered into the executable |

**What to expect.** All three compile, and none of them does anything useful
without what it needs. Nothing else in the documentation depends on them.

**Check before relying on it.** Try each with what it needs, on your own system.

## The terminal editors' key bindings

**`micro`'s key bindings are the program's own, unmodified.** SD only launches it
and stages its syntax highlighting. If a binding surprises you, that is the
program's own documented behavior.

## What is not on this page

The places where SD Core Solo for Linux deliberately differs from OpenQM,
ScarletDME, upstream `sdb64` or the multiuser SD Core for Linux are documented as
differences, in the pages that describe the feature. This page is only about
behavior that depends on your own environment.

## See also

[Sessions and locks](06a-sessions-and-locks.html) covers the locking model that
some of the entries above qualify. [System limits](16a-system-limits.html) states
the figures SD works within.
