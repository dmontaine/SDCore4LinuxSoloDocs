Title: Client distribution
Subtitle: The shared library an application needs, why there are two names, and how to build one.

An application reaches SD Core through a shared library. **You do not have to
find it inside an installed SD system** — the source builds standalone as
well, because the person writing an application and the person running the
server are not always the same person.

## Two names, one source

```
sdclilib.so
libsdcli.so
```

Both are built from the same object files in the same step — `sdclilib.so` is
the name an application historically asks for; `libsdcli.so` follows the
ordinary Unix `lib*.so` convention (`-lsdcli` at link time). **They are not two
copies to keep in sync by hand** — one build target produces both, so there is
nothing to drift.

**This release carries no 32-bit build.** SD Core for Windows ships a separate
32-bit client for QM-heritage applications and a third-party tool that only runs
32-bit; neither concern applies here.

## Where it lands

Built to `bin/` under the source tree by `make`; an installed Solo tree has both
under `~/SDCoreSolo/bin`. There is no separate client-only package yet —
building from source, or copying the `.so` from an installed tree, is how an
application gets it today.

## The library must match the release

**The cleartext API login is gone.** A client that still sends a password in
clear is refused with *"Cleartext login is no longer supported; this server
requires SCRAM authentication"*. Use a client library from this release or
later.

## Connecting

Your application code does not change for `SDConnect()`, which takes the same
arguments and returns the same things it always did. **`SDConnectLocal()` is
different: it is disabled**, because it sends no password and every Solo session
proves one.

| | |
|---|---|
| `SDConnect()` | over the network, to port **4243** unless you chose another. Use `127.0.0.1` for this computer. Signs in as `sduser` with the account password |
| `SDConnectLocal()` | **not available.** It returns at once, with the error *SDConnectLocal is not available in SD Core for Linux Solo - connect with SDConnect and the account password*. (Before this was fixed it hung for ever on Solo, because the library looks for `/etc/sd.conf`, which a Solo tree does not have.) |

BASIC programs reaching another SD server use the `!sdclient` class, which
speaks the SCRAM login too — `connect()` takes the same arguments, but **the
program has to be running under this release at both ends.**

Everything about what a connected session may open and the identity it runs as
is on [API access](09-api-access.html).

## Building from source

The source lives in the server's own repository at `sdb_ai/sd64/gplsrc/sdclilib.c`,
and `make` produces it together with the server —
<https://github.com/dmontaine/SDCore4LinuxSolo>. **No built `.so` is committed**,
so a clone builds — the same no-binaries rule the whole repository follows.

A standalone copy of the same source, kept in sync for building the client
without the whole server tree, is at <https://github.com/dmontaine/linuxsdclilib>.
That copy is the multiuser product's and still has `SDConnectLocal`; against a Solo
computer use `SDConnect`.

**The C headers are shipped with the source tree.**
