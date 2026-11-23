Title: Sessions and Locks
Subtitle: Seeing who is signed in, ending a session that will not end itself, and inspecting or forcing the locks a session holds.

These are the verbs for looking at the machine as a whole and intervening in it:
**whose sessions are running, what they are holding, and how to take either
away.** Most need `admin` first; the table at the end says which.

SD folds case, so a command may be typed in either case. Commands are shown here
in lower case. In the tables, *italics* mark something you supply and **bold**
marks a word typed as it stands; braces mark an optional part.

> Paths are shown as they read on an SD Core Solo for Linux computer.

## Two things are not on this page, on purpose

**A programmer needs to understand locks without being able to force one**, so
the *user* documentation keeps the part of the subject that is theirs: what a
task lock is against a database lock, and `release`, which gives back locks your
own session holds and needs no privilege at all. The same goes for `status`,
`phantom`, `pstat`, `pdebug` and `pdump` — a programmer's view of their own
processes.

**This page is the other half**: the verbs that see and act across the machine.

## Who is signed in: `listu`

```
listu {detail} {no.page} {lptr {n}}
```

```
:listu
  User  Pid          Puid  Login time    Origin : Username
    12  1789               27 Aug 13:14  : sduser
    19  1808               27 Aug 13:53  : sduser
*   27  1828               27 Aug 13:59  : sduser
```

| | |
|---|---|
| **`*`** | the session you are typing in |
| **`User`** | SD's user number — what `logout`, `pstat` and `pdump` all take |
| **`Pid`** | the Linux process id |
| **`Puid`** | the user number of the **parent**, filled in for a phantom and blank otherwise |
| **`Origin`** | where the session came from |
| **`Username`** | always `sduser` on a Solo computer |

```
listu detail {no.page} {lptr {n}}
```

`detail` adds a line above the report: the interactive users, the API sessions,
the phantoms that count as licensed, the other phantoms, and the most users
there have been at once since SD started. `system(1056)` gives the same five
numbers. **SD has no license count**, so the third number is whatever the system
marks as chargeable.

**`Origin` is blank for a local session and that is not a fault.** It reports
an address or a device name, and a console or piped session has neither. It
reads `Phantom` for a phantom and an IP address for a network client — the API,
or ssh.

`SDNet` is also among the values it can print, and you will not normally see
it. The mark behind it is set in only one place, the API server's request 29,
"open an SDNet file", which is refused unless `NETFILES` has bit 2 set (see
*Configuration*). The SDNet client, where a program opens a file on another SD
server through an `N` pointer, does not use request 29, so a session that comes
from it shows an address like any other API session.

**`(logout pending)`** after the name means somebody has asked that session to
end and it has not gone. See below.

**`sd-solo -u`**, from a terminal, lists the same sessions without an SD session of
your own, and needs no `admin`.

## Ending a session: `logout`

```
logout                  end this session
logout n {n …}          end another session, by user number
logout all              end every session but this one
```

**`logout` with no argument ends your own session.** It is `quit` under another
name, which is worth knowing before typing it intending to list something.

```
:logout 27
Force logout initiated for user 27
```

**`logout n` needs no `admin`.** SD lets a session end any session running under
the same user name, and on a Solo computer every session is `sduser`.
**`logout all` needs `admin`.** It leaves your own session alone.

**`sd-solo -k n`** and **`sd-solo -k all`**, from a terminal, do the same from outside SD.

### When a session will not end

`logout` **signals** a process. **If the process has already gone** — killed from
outside SD, or lost with a terminal — there is nothing to signal, and `logout`
finds that out first: it **clears the entry and releases every file, record,
group and task lock the session still held**, and says so (then the text of
message 10167, *User n was not running, so there was nothing to log out*):

```
LOGOUT reaped user 43 (pid 1799435, sduser) - process was gone.
```

For a session killed from outside with `kill -9` while it held a record lock,
`logout 43` prints the line above, and afterwards `listu` does not list the user
and `list.readu` does not show the lock. The entry matters because **it holds a slot
and an exclusive-access claim**: `NUMUSERS` counts it, and any verb that wants a
file to itself — `build.index` is the usual one — is refused while it is there.
So **the way to clear a dead session is `logout` *n*** — no `admin`, since every
session on a Solo computer is `sduser`.

A session that is **alive and does not answer the signal** is the other case,
and it keeps its entry with **`(logout pending)`** beside it; asking again
signals it again.

```
sd-solo -cleanup
```

from a terminal — no `sudo`, no `admin` — is the remaining way to clear entries
of sessions that are not going to end, and `sd-solo -stop` then `sd-solo -start`
if that does not take them.

**Confirm the session is actually dead before clearing it.** `pstat` *n*
answers *(Not responding)* for a session with nothing behind it — it asks the
process and waits about four seconds — and that is the difference between a dead
entry and a busy one. `pstat` is a programmer verb and is documented in the user
set. (`logout` itself only clears an entry whose process is not there; it
signals a live one.)

## Keeping new sessions out while you work: `inhibit.login`

```
inhibit.login on        no new session until it is switched off
inhibit.login off       allow them again
inhibit.login           say which it is, and who set it
```

For maintenance that needs the system quiet. **Needs `admin`.** A person or program
that tries to sign in while it is on — at the keyboard, over ssh or through the
API — is refused with *SD is not accepting new sessions: logins have been
inhibited by* name *(inhibit.login). Try again later.* **Sessions already running
are not touched**, and neither are the phantoms the session that set it starts.
The refusals, and `inhibit.login on` and `off` themselves, are on the audit trail.

**On Solo the session that set it is the only one that can switch it off, and
nobody new gets in, you included.** On SD Core for Linux, the multiuser product, the administrator's
own new session is let in; on Solo the administrator is a state of a session (you
type `admin` after signing on), so at the door no new session is one. That cannot
lock you out: **if the session that set it ends or is killed, the inhibit is ignored
from then on** — the record names a user number *and* a process id, and a process
that is not there holds nothing — and the next `inhibit.login on` replaces it.
`inhibit.login on` while another live session holds it is refused (*Logins are
already inhibited by* name*. Switch it off first (inhibit.login off)*).

The record is the file `login.inhibit` in the SD system directory. **It is not the
same thing as the hold that `backup.account` and `restore.account` take**, which
keeps *everybody* out, the setter's new sessions included, while they run.

## The paragraph every session runs first: `MASTER.LOGIN`

A record named `MASTER.LOGIN` in the VOC of the SD system directory (`~/SDCoreSolo/voc`, not
the account's own VOC), if it is a paragraph (its first line is `PA`), is run **once by every
new session, before the account's own `LOGIN` paragraph**: an interactive session, a
phantom and an API session. There is none as delivered, so nothing runs by
default, and the account's own `LOGIN` paragraph is the ordinary place for what a session
should do when it starts; a session shows what it is by `@tty`. It runs in the account,
**under the account's own switches**: it can do no more there than the account can.

## What is locked: `list.readu`

```
list.readu {user.no} {detail} {wait} {no.page} {lptr {n}}
```

```
:list.readu
There are no active file, read or update locks held by any user
```

That answer is about the **whole system**, not about you. With locks held:

```
User File Path........................... Type Id..............................
  23    1 /home/you/SDCoreSolo/user_ac RU   R1
          counts/sduser/zzlk31a
  23    1 /home/you/SDCoreSolo/user_ac RL   R2
          counts/sduser/zzlk31a
```

| | |
|---|---|
| **`User`** | the SD user number holding it — `listu` turns that into a session |
| **`File`** | SD's internal file number, and **this is the number `unlock` wants** |
| **`Path`** | wrapped over as many lines as it needs |
| **`Type`** | `RU` update · `RL` read · `FX` exclusive file lock · `SX` shared file lock · `WAIT` a session waiting for one |
| **`Id`** | the record id; blank for a file lock |

**An id is text a program wrote, and it is shown safely.** A program may lock a record
whose id holds an escape sequence, and the listing, the deadlock report and `tool.sessions` would have sent that
sequence to **your** terminal. Now a control character, `del`, a byte from 128 to 159 and a
mark character in an id, a path or a user name are each shown as `?`. The same holds for `getlocks()` and
`system(1083)` in a program. The locks themselves are not changed (`unlock` and `release` act on the real id),
and `fileinfo(f, 1039)` still gives a program its own ids exactly.

A file lock has no id:

```
User File Path........................... Type Id..............................
  23    1 /home/you/SDCoreSolo/user_ac FX
          counts/sduser/zzlk31a
```

### The keywords

**`detail`** prints the record-lock budget above the listing:

```
:list.readu detail
Record lock limit (NUMLOCKS) = 100, Current = 0, Peak = 1
There are no active file, read or update locks held by any user
```

**`Peak` is the one to watch.** `Current` describes this instant, which is rarely
the instant that mattered. A peak near the limit is how you find out `NUMLOCKS`
needs raising **before** a program fails with the lock table full. `config`
reports the configured limit — see [Configuration](16-configuration.html).

**A user number** restricts the listing to one session. **`wait`** adds the
sessions *waiting* for a lock, as `WAIT` rows — left out by default, so the plain
listing shows the cause of a hold-up and not its victims. **Ask for `wait` when
something is stuck.**

### A lock outliving its session

**A dead session's locks are not released by its dying**, any more than its
user-table entry is removed, and everything wanting that record waits for a
process that is not there. **They are released when the entry is cleared**,
which is `logout` *n* for a session whose process is gone (see *When a session
will not end*), or the `tool.sessions` page's L key.

**Take the user number to `pstat`.** *(Not responding)* means clear the session
rather than wait for it.

## Continued in

[Sessions and Locks: Lock Tracing, Task Locks and the Sessions
Screen](06a1-task-locks-and-forcing-a-lock-open.html) — lock.tracer, task locks,
unlock and tool.sessions.
