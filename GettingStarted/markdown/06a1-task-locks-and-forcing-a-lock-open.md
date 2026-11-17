Title: Task Locks and the Sessions Screen
Subtitle: Task locks, forcing a lock open with unlock, the sessions and locks screen, and the paragraph every session runs first.

This page continues [Sessions and Locks](06a-sessions-and-locks.html).

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
| F6, Shift-F6 | change to the other list (and to the AI line, when the AI is on for the account) |
| L (sessions) | `logout` the selected session; the user number is typed again to confirm. For a session whose process is gone the entry is cleared and its locks are released |
| U (locks) | `unlock` the selected lock; the holder is not told |
| I | `inhibit.login on` when logins are allowed, `off` when they are inhibited |
| / | go to the next row of the list that has the focus that holds a text you type (a user number, a name, a record id; any case; round to the top) |
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

## The paragraph every session runs first: `MASTER.LOGIN`

A record named `MASTER.LOGIN` in the VOC of the SD system directory (`~/SDCoreSolo/voc`, not
the account's own VOC), if it is a paragraph (its first line is `PA`), is run **once by every
new session, before the account's own `LOGIN` paragraph**: an interactive session, a
phantom and an API session. There is none as delivered, so nothing runs by
default, and the account's own `LOGIN` paragraph is the ordinary place for what a session
should do when it starts; a session shows what it is by `@tty`. It runs in the account,
**under the account's own switches**: it can do no more there than the account can.

## Who needs `admin`

| | |
|---|---|
| `tool.sessions`, `listu`, `list.readu`, `list.locks`, `lock`, `clear.locks`, `unlock`, `logout all`, `lock.tracer`, `inhibit.login` | `admin` first |
| `logout`, `logout n`, `sd-solo -u`, `sd-solo -k`, `sd-solo -cleanup` | nothing |
