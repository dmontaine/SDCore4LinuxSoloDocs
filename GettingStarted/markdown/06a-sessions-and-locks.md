Title: Sessions and Locks
Subtitle: Seeing who is signed in, ending a session that will not end itself, and inspecting or forcing the locks a session holds.

These are the verbs for looking at SD as a whole and intervening in it:
**which sessions are running, what they are holding, and how to take either
away.** Most need `admin` first; the table at the end says which.

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

*Run on 24 October 2026 on Solo, in a sandbox, against a session killed from
outside with `kill -9` while it held a record lock: `logout` on its number
printed the line above, and afterwards `listu` did not list the user and
`list.readu` did not show the lock.* The entry matters because **it holds a slot
and an exclusive-access claim**: `NUMUSERS` counts it, and any verb that wants a
file to itself — `build.index` is the usual one — is refused while it is there.
So **the way to clear a dead session is `logout` *n*** — no `admin`, since every
session on a Solo computer is `sduser`. (Earlier editions of this page said
`logout` could not clear it and that `sd-solo -cleanup` was the way; the run
above is why that was taken out.)

A session that is **alive and does not answer the signal** is the other case,
and it keeps its entry with **`(logout pending)`** beside it; asking again
signals it again. *That case was not run for this page.*

```
sd-solo -cleanup
```

from a terminal — no `sudo`, no `admin` — is the remaining way to clear entries
of sessions that are not going to end, and `sd-solo -stop` then `sd-solo -start`
if that does not take them.

**Confirm the session is actually dead before clearing it.** `pstat` *n*
answers *(Not responding)* for a session with nothing behind it — it asks the
process and waits about four seconds — and that is the difference between a dead
entry and a busy one. (`logout` itself only clears an entry whose process is not
there; it signals a live one.)

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

**A dead session's locks are not released by its dying**, any more than its
user-table entry is removed, and everything wanting that record waits for a
process that is not there. **They are released when the entry is cleared**,
which is `logout` *n* for a session whose process is gone (see *When a session
will not end*), or the `tool.sessions` page's L key.

**Take the user number to `pstat`.** *(Not responding)* means clear the session
rather than wait for it.

## Who took which lock, and where: `lock.tracer`

`list.readu` shows what is locked *now*. When the question is who took a lock, in
which program and on which line, or what locked what in the second before a
hold-up, switch the tracer on, let the system run for a short while, and read the
log. **Needs `admin`.**

```
lock.tracer on
lock.tracer off
lock.tracer {user n} {lptr {n}} {no.page}
```

While it is on, every lock any session takes or lets go is added, as one line, to
the file **`locktrace` in the SD system directory**. `on` empties that file
first, so a log is one switching-on's worth. The report lists the lines, oldest
first, with the **date and time**, the **user number**, the **action**, the
**program** that was running and the **line number** in it, the **file** and the
**record id**. `user n` shows one session's lines; `lptr` prints them and
`no.page` stops the pause at the end of a page. An ordinary session is told
*lock tracing is for the administrator*.

| action | means |
|---|---|
| `Get RU` | a record update lock taken |
| `Get RL` | a shareable read lock taken |
| `RL->RU` | a read lock raised to an update lock |
| `RU->RL` | an update lock lowered to a read lock |
| `Get FX` | a file lock taken |
| `Rel RX` | a record lock let go |
| `Rel FX` | a file lock let go |

**It logs everything, SD's own locks included**: a sign-on takes `$ipc`, the
command processor takes `$command.stack`, and reading the log reads a file. The
volume is large and every lock costs a file write, so switch it on for a short
time and off again.

> **Only a lock that is granted is logged**: one that is refused, or has to wait,
> is not, until the moment it is granted. A session that is killed has its locks
> cleared by the cleanup, and that is not logged either, so a `Get RU` with no
> `Rel RX` after it may be a lock the cleanup gave back.

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
nobody new gets in, you included.** On the multiuser SD Core the administrator's
own new session is let in; on Solo the administrator is a state of a session (you
type `admin` after signing on), so at the door no new session is one. That cannot
lock you out: **if the session that set it ends or is killed, the inhibit is ignored
from then on** — the record names a user number *and* a process id, and a process
that is not there holds nothing — and the next `inhibit.login on` replaces it.

The record is the file `login.inhibit` in the SD system directory. **It is not the
same thing as the hold that `backup.account` and `restore.account` take**, which
keeps *everybody* out, the setter's new sessions included, while they run.

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

**This is the only verb that takes somebody else's lock, and it needs `admin`:**

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

> **`sd-solo -cleanup` does not give them back, and that is a defect.** It releases a
> dead session's record locks and file locks and leaves its task locks held, by
> a user number nothing is behind, until SD itself is restarted. `list.locks`
> shows the number with an owner and `clear.locks` refuses it because it is not
> yours. **`unlock tasklock` *n*, after `admin`, is the way out** — that is what
> the forced form is for.

## Sessions and locks on one screen: `tool.sessions`

```
tool.sessions
```

**A full-screen page for the administrator** — type `admin` first; without it the page is
refused before it reads anything. It needs a window of 140 columns by 43 lines. The top list
is every session — user number, Linux process, when it signed on, its kind (user, phantom,
API or SDNet), the login user (always `sduser` here), the account it is in, where it came
from, and whether it is yours (a star) or a logout is pending. The lower list is every lock —
the user, the file number and path, the type (`FX` and `SX` lock a whole file, `RU` is read
and update, `RL` is read, `WAIT` is a session waiting for a lock), the record id, since when,
and whether a session with that user number is still signed on. Under the sessions the page
says whether logins are inhibited and by whom, and under the locks it gives the lock limit,
the number held now and the most there have been. **Both lists are read again every three
seconds, with no key pressed, and on F5, and a refresh keeps the cursor on the same user or
the same lock, not on the same row number.** A box under the lists explains the row under the
cursor in plain sentences and shows the exact command its key would run.

| Key | What it does |
|---|---|
| arrows, Page Up, Page Down, Home, End, a click | move in the list that has the focus |
| Tab | change to the other list |
| L (sessions) | `logout` the selected session; the user number is typed again to confirm. For a session whose process is gone the entry is cleared and its locks are released |
| U (locks) | `unlock` the selected lock; the holder is not told |
| I | `inhibit.login on` when logins are allowed, `off` when they are inhibited |
| F5 | read again |
| F1, ?, a click on the [?] | explain the page |
| Ctrl-Q | leave |

**The page changes nothing by itself.** Each of the three keys builds one command from values
the page has read and checked, shows it in a box with what it does, and runs it only when you
answer Run; Cancel is what Enter chooses. Those three commands are the only ones the page can
run. **`logout` and `unlock` write no audit line of their own, so the page writes one for
every command it ran** (`tool run app=tool.sessions command=... status=...`), as well as the
line that says it was started (`tool start app=tool.sessions`); `inhibit.login` writes its
own as well.

What each key refuses, in words, before any box opens: `L` on the session the page runs in
(leave it with Ctrl-Q); `U` on a `WAIT` row, which is a session waiting for a lock and not a
lock; `U` on a record id with a character the page will not put in a command (only lower-case
letters, digits, dot, dash and underscore go through) or an id that is one of the words
`unlock` reads as a keyword (`all`, `file`, `user`, `filelock`, `tasklock`) — release those at
the prompt with `unlock`. The lock table keeps record ids in upper case and shows them so; the
page sends the id in lower case, which the verb folds.

**`logout` ends a session by signalling its process**, so work that session has not saved is
lost; the box says so before you answer Run. **While `inhibit.login` is on, only this session
and the phantoms it starts can sign in, and if this session is lost the hold lapses** — the
page's box says that too.

**Not on the page:** a message to a user, `logout all`, and `unlock tasklock`.

## Who needs `admin`

| | |
|---|---|
| `tool.sessions`, `listu`, `list.readu`, `list.locks`, `lock`, `clear.locks`, `unlock`, `logout all`, `lock.tracer`, `inhibit.login` | `admin` first |
| `logout`, `logout n`, `sd-solo -u`, `sd-solo -k`, `sd-solo -cleanup` | nothing |
