Title: Sessions and Locks
Subtitle: Seeing who is signed in, ending a session that will not end itself, and inspecting or forcing the locks a session holds.

These are the verbs for looking at SD as a whole and intervening in it:
**which sessions are running, what they are holding, and how to take either
away.** Most need `ADMIN` first; the table at the end says which.

SD folds case, so a command may be typed in either case. Commands are shown here
in lower case. In the tables, *italics* mark something you supply and **bold**
marks a word typed as it stands; braces mark an optional part.

> **The listings on this page were produced by running the verbs** on the
> multiuser SD Core for Linux, whose code for them Solo shares apart from the
> gates, and re-checked on Solo. Paths are shown as they read on a Solo
> computer.

## Two things are not on this page, on purpose

**A programmer needs to understand locks without being able to force one**, so
the User set keeps that part of the subject: what a task lock is against a
database lock, and `release`, which gives back locks your own session holds.
The same goes for `status`, `phantom`, `pstat`, `pdebug` and `pdump` — a
programmer's view of their own processes.

## Who is signed in: `listu`

```
listu {no.page} {lptr {n}}
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

**`Origin` is blank for a local session and that is not a fault.** It reports
an address or a device name, and a console or piped session has neither. It
reads `Phantom` for a phantom and an IP address for a network client — the API,
or ssh.

**`(logout pending)`** after the name means somebody has asked that session to
end and it has not gone. See below.

**`sd -u`**, from a terminal, lists the same sessions without an SD session of
your own, and needs no `ADMIN`.

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

**`logout n` needs no `ADMIN`.** SD lets a session end any session running under
the same user name, and on a Solo computer every session is `sduser`.
**`logout all` needs `ADMIN`.** It leaves your own session alone.

**`sd -k n`** and **`sd -k all`**, from a terminal, do the same from outside SD.

### When a session will not end

`logout` **signals** a process. If the process has already gone — killed from
outside SD, or lost with a terminal — there is nothing to signal, and the entry
stays with **`(logout pending)`** beside it.

**The entry matters because it holds a slot and an exclusive-access claim.**
`NUMUSERS` counts it, and any verb that wants a file to itself — `build.index`
is the usual one — is refused while it is there. **Recovery is not another
`logout`:**

```
sd -cleanup
```

from a terminal — no `sudo`, no `ADMIN` — and `sd -stop` then `sd -start` if
that does not take it.

**Confirm the session is actually dead before clearing it.** `pstat` *n*
answers *(Not responding)* for a session with nothing behind it — it asks the
process and waits about four seconds — and that is the difference between a dead
entry and a busy one.

## What is locked: `list.readu`

```
list.readu {user.no} {detail} {wait} {no.page} {lptr {n}}
```

```
:list.readu
There are no active file, read or update locks held by any user
```

That answer is about **every session**, not only yours. With locks held:

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

**A dead session's locks are not released**, any more than its user-table entry
is, and everything wanting that record waits for a process that is not there.
**Take the user number to `pstat`.** *(Not responding)* means clear the session
rather than wait for it.

## Task locks: `list.locks`, `lock`, `clear.locks`

Task locks are **64 numbered flags with no connection to any data**. They mean
whatever the programs using them agree they mean — usually *only one session runs
this job at a time*, where the thing being protected is not a single file.

```
list.locks
lock n {no.wait}
clear.locks {n}
```

```
:list.locks
No task locks reserved by any user
```

and with lock 5 held by user 29, `*` marking your own session:

```
:list.locks
 0:       1:       2:       3:       4:       5: 29*   6:       7:
 8:       9:      10:      11:      12:      13:      14:      15:
16:      17:      18:      19:      20:      21:      22:      23:
```

*(64 numbers over eight rows; the rest are blank and are cut here.)*

### `lock` waits unless you tell it not to

```
:lock 5 no.wait
Set task lock 5
:lock 5 no.wait
Task lock already owned by this process
```

**Without `no.wait` it waits for ever.** It prints *Waiting for task lock to
become available* once, then retries every two seconds until it gets it — right
for a job that must run, wrong for anything unattended. **`no.wait` turns the
wait into a refusal** (*Task lock is already in use*), which a script can act on.

### `clear.locks` gives back your own, and only your own

```
:clear.locks 5
Released task lock 5
:clear.locks
All task locks released
```

**With no number it releases all 64 this session holds**, and says *all* whether
it held any or not.

| | |
|---|---|
| *Released task lock n* | it was yours and it is now free |
| *Task lock n is held by another process* | **`clear.locks` will not take it** |
| *Task lock n is not held by any process* | it was already free |
| *Task lock number must be in range 0 to 63* | from `clear.locks 99` |

## Forcing a lock open: `unlock`

```
unlock file n {user n} record.id {record.id …}
unlock file n {user n} all
unlock file n {user n} filelock
unlock tasklock n {n …}
```

**This is the only verb that takes somebody else's lock, and it needs `ADMIN`:**

```
:unlock
Command requires administrator privileges
```

**It names a file by number, not by name** — the `File` column of `list.readu`.
That is deliberate: the lock is on a file the computer has open, which may not
be in your VOC at all.

| | |
|---|---|
| **`file`** *n* | the file, by the number `list.readu` printed |
| **`user`** *n* | restrict to one session's locks |
| **`all`** | every record lock matching, rather than named ids |
| **`filelock`** | the file lock, which cannot be combined with record ids |
| **`tasklock`** *n* | force a task lock, including one held by another process |

**A file number or a user number is compulsory** — *Either a file number or a
user number must be specified* — so there is no `unlock` that means *everything
on the computer*.

> **Unlocking is not free and SD cannot make it so.** A lock is a promise its
> holder is relying on. Forcing one open while its owner is alive lets two
> sessions write the same record and tells neither. **Establish that the holder
> is dead first** — `pstat` on the user number, `listu` for *(logout pending)* —
> and prefer clearing the session to clearing the lock.

### `unlock tasklock` is the only way back from a killed holder

**Task locks are released when a session ends normally.** They are not part of
any file, so nothing has to be written back.

> **`sd -cleanup` does not give them back, and that is a defect.** It releases a
> dead session's record locks and file locks and leaves its task locks held, by
> a user number nothing is behind, until SD itself is restarted. `list.locks`
> shows the number with an owner and `clear.locks` refuses it because it is not
> yours. **`unlock tasklock` *n*, after `ADMIN`, is the way out** — that is what
> the forced form is for.

## Who needs `ADMIN`

| | |
|---|---|
| `listu`, `list.readu`, `list.locks`, `lock`, `clear.locks`, `unlock`, `logout all` | `ADMIN` first |
| `logout`, `logout n`, `sd -u`, `sd -k`, `sd -cleanup` | nothing |
