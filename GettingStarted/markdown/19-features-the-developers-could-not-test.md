Title: Features the developers could not test
Subtitle: The parts of SD Core for Linux Solo that were built and reasoned about but never exercised, what is known about each, and what it would take to settle it.

Everything else in this documentation describes behaviour that was run and
watched. **This page is the exception, and it exists so that the exception is
visible in one place rather than scattered through the reference as footnotes.**

Nothing here is known to be broken. Each entry is something that **compiles, or
exists, or follows from the source, and was never put under load or into the
condition that would prove it.** Treat them as the list of things to pilot before
an application depends on them. **This list is drawn from this release's own build
and witness record**, not carried over from the multiuser product's version of
this page: the two differ in which parts have actually been run, not just in which
parts exist.

## How to read an entry

| | |
|---|---|
| **Known** | what was actually run and observed |
| **Not known** | the specific gap — usually narrower than the heading suggests |
| **To settle it** | what would have to be done |

## Installing and running

### The `Match` block, `sudo`, and linger

**Known.** The text of the `sshd_config.d` block is what `solo-ssh.sh match`
prints, and `sshd -t` accepts it. The key route was run against a private `sshd`:
a key with the forced command gets SD and no shell, and a key without it gets a
shell.

**Run once, on one computer (30 Sep 2026, Ubuntu).** An install with `--ssh-match`
and `--enable-linger` ran the package installation (`apt`), wrote the block into
the real `/etc/ssh/sshd_config.d`, checked it with `sshd -t` and reloaded `sshd`;
an ssh login with the key then reached `sd` with no shell, and linger was on.

**Not known.** Removing the block again (`match --remove`); a **password** login
landing in SD through the block; the package installation on `dnf`, `zypper` and
`pacman`; the `ufw` rule. And **linger**: that it is on is measured, but whether SD
really survives your last sign-out and is there for a cron job or an ssh login that
arrives afterwards is not.

**To settle it.** Run each once on a computer you can afford to lock yourself out
of; sign out with linger on, and connect again from elsewhere.

### An install from the published branch

**Known.** The installer was run end to end many times, including interactively at
a terminal, from a local clone of the committed source, and its upgrade and
uninstall were run against what it made.

**Not known.** An install of the published `main` branch on GitHub, over the real
network, on any distribution other than the one it was developed on.

**To settle it.** Run `installsolo.sh` from a fresh clone on each distribution
family.

### A first install from a control file, over ssh and the API

**Known.** At the console the first password is chosen, the refusals work, and
afterwards every route works. Over a piped session and a session that says it is
over ssh, only the global password is accepted until then.

**Not known.** The **API** while the account password is unset — the code path
that serves the global proof when there is no account record was reasoned from the
source and not exercised.

**To settle it.** Install from a control file, then sign in over the API with the
global password before choosing the account password.

## The API

### Confinement of an API session to the account

**Known.** The multiuser product's API refuses to open anything outside the
account, and Solo runs the same login code.

**Not known.** That refusal has not been measured on Solo, so it is not documented
as a boundary. `sdclilib.so` and the server were run end to end against SCRAM, the
account-only rule and the wrong-password refusals.

**To settle it.** An API client that tries to `OPEN` a file outside the account.

### The server's ssh key and the pinned certificate (LS1.1-2)

**Known.** On a managed scratch tree, over a real SCRAM and TLS login: the server's
key is installed, repeated, listed, removed, capped at four, and refused for the
account password; shell syntax in a key is not run; the installed key reaches `sd`
through a private `sshd` with the global password. The client library remembers a
server's certificate, accepts the same one again, and refuses a replaced one before
any login byte is sent, and the remedy in its message works.

**Not known.** All of it was measured between programs on **one computer**, over
the loopback address: the address recorded in the audit trail was `127.0.0.1`, and
a client and a server on different computers have not been tried. The **SD Core for
Linux server itself** is not in this product, so no real server has installed a key
or been pinned — what was run is the client library every `SDConnect` caller uses
and a test client. A login through the computer's own `sshd` with the server's key,
and Windows Solo's side of the same request, were run by its own tests, not
together with this one.

**To settle it.** Have a second computer sign in with the global password, ask for
its key to be installed, and then connect over ssh with that key; reinstall the
managed computer and confirm the client is refused until the pin is removed.

## Sessions

### `sd` started from inside a running session

**Known.** From `sh`, a command that runs `sd` — for example `sh /full/path/sd who`
— starts a **separate, nested one-shot session** and returns its output. There is
no guard against it, unlike SD Core for Windows, which refuses. There is no
interactive `sh` from TCL to be left in (a bare `sh` is refused), so the
interactive case cannot arise.

**Not known.** A nested session left waiting at a prompt.

## Locking and contention

### Semaphores under contention

**Known.** The semaphores are exercised on every record lock.

**Not known.** Whether a semaphore has been observed genuinely blocking — one
session waiting on another under real contention, as opposed to two sessions each
getting an uncontended lock in quick succession.

**To settle it.** Enough concurrent sessions to make one wait, and a watch on what
it does while it waits.

### Contention between an API session and a local one

**Known.** Two *local* sessions compete correctly.

**Not known.** A lock held by a local terminal session and contested from an API
connection specifically.

**To settle it.** An API client and a terminal session competing for one record.

### Task locks taken twice by the same session

**Known.** From the source: taking a task lock you already hold succeeds, and one
`unlock` releases it however many times you locked it.

**Not known.** It was **read rather than run**.

**To settle it.** Four lines of SD BASIC.

## Application data

### A real application's data

**Known.** SD creates, writes, reads and deletes files, records, indexes, select
lists and sequential files, and the system files it bootstraps with are real ones.

**Not known.** **No production application's data has been loaded into this
release.** Nothing here has met a file of hundreds of thousands of records, a deep
dictionary, or a schema built over years by somebody else.

**To settle it.** Restore an existing account and run it.

## Sockets

### UDP and ICMP

**Known.** TCP works: listening, connecting, accepting, reading, writing, the
blocking and non-blocking modes, and the error codes — exercised directly by the
TLS/SCRAM interop work.

**Not known.** The `0x00010000` and `0x00020000` flags are named in the
documentation because they are **in the compiler**, not because a datagram was ever
sent. No UDP or ICMP socket has been opened.

**To settle it.** A datagram to a listener and back.

## SD BASIC statements that compile but were never run

| | why not |
|---|---|
| `sendmail` | needs a mail relay configured |
| `chgphant()` | needs a phantom process to change |
| `ccall()` | needs a C function registered into the executable |

**Known.** All three compile.

**Not known.** What any of them does. Nothing else in the documentation depends on
them.

## The terminal editors' key bindings

**`nano` and `micro`'s key bindings are each program's own, unmodified — this
release does not implement or alter either editor**, only launches it and, for
`micro`, stages its syntax highlighting. If a binding surprises you, that is each
program's own documented behaviour, not something to report against SD.

## What is NOT on this page, and why

**Anything that was tested and failed is a defect, not a gap**, and does not belong
here — it is either fixed or it is a known issue.

**Anything a reader might merely find surprising is not a gap either.** The places
where this release deliberately differs from OpenQM, ScarletDME, upstream `sdb64`,
or the multiuser SD Core for Linux are documented as differences, in the pages that
describe the feature. This page is only about what nobody has watched happen.

## See also

[Sessions and locks](06a-sessions-and-locks.html) covers the locking model that some
of the entries above qualify. [System limits](16a-system-limits.html) states which
of its figures come from the source.
