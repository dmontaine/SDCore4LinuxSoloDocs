Title: SD TCL - Locks
Subtitle: The two kinds of lock, giving back the ones your session holds, and what you cannot do without admin.

**There are two unrelated things called a lock, and knowing which is which is
most of what this page has to say.** They do not interact, and the names do not
make that obvious.

| | |
|---|---|
| **database locks** | on a record or on a whole file. Taken by `readu`, `readl` and `filelock` in a program, and by `ed` while it holds a record |
| **task locks** | 64 numbered flags, 0 to 63, with **no connection to any data**. A program takes one by agreement to keep other programs out of something |

The statements a program uses to take either are in
[SD Basic - Locks and Transactions](14-sd-basic-locks-and-transactions.html),
and this page does not repeat them.

SD folds case, so a command may be typed in either case. Commands are shown here
in lower case. In the tables, *italics* mark something you supply and **bold**
marks a word typed as it stands; braces mark an optional part.

## Only one lock verb needs no `admin`

**Inspecting locks and forcing them open need `admin`.** `list.readu`,
`list.locks`, `lock`, `clear.locks` and `unlock` are in your account, and each
one refuses with *Command requires administrator privileges* until you type
`admin`. They are described under *Sessions and locks* in the GettingStarted set.

**What needs nothing is `release`**, which gives back locks this session holds.
That is the whole of the TCL lock interface without `admin`, and the reason is
defensible: a lock you took is yours to give back, and looking at the table or
taking somebody else's is an administrator's act.

## Giving your own locks back: `release`

```
release filename id {id …}
release filelock filename
```

It is the TCL form of the BASIC statement of the same name, needs no special
rights, and acts **only on locks this session holds**. The file is named the way
you would name it to any other verb.

```
:release
File  not found
:release voc
Record id required
```

**A record id is compulsory in the first form** — there is no *release
everything* at the prompt. `release filelock` *name* gives back a file lock,
which has no id to give.

**A `write` outside a transaction already releases the record lock**, so
`release` at the prompt is for the case where a program took a lock and ended
without writing — usually because you stopped it — and for a file lock, which
nothing releases implicitly.

## What to do when something is stuck

You cannot see the lock table without `admin`, so the useful sequence is:

| | |
|---|---|
| 1 | **`release`** anything you know your own session took |
| 2 | `status` and `pstat`, on [SD TCL - Processes and Phantoms](30-sd-tcl-processes-and-phantoms.html), to see whether the program you think is holding it is still alive |
| 3 | if it is not yours, type `admin`, then `list.readu` to see who holds it and, once you know the holder is dead, `unlock` to force it open |

**A dead session's database locks are not released.** A session killed from
outside SD keeps both its user-table entry and its record and file locks, so
everything wanting that record waits for a process that is not there. **`release`
will not clear that**; the recovery is `sd-solo -cleanup`, typed at a terminal — it
needs no `sudo` and no `admin`.

## Task locks, and why you will rarely meet one

A task lock protects something that is **not a file** — a nightly job, an
external resource, a sequence that must not run twice at once. Two programs
agree that lock 7 means *this job*, and the number carries no other meaning.

**Taking and releasing one is `lock` and `clear.locks`, both of which need
`admin`**, so in practice task locks are taken from **inside a program** with the
BASIC `lock` and `unlock` statements, which you may compile and run. That is the
route to reach for; the TCL verbs exist for inspecting or clearing the table by
hand.

> **A task lock held by a killed session is not given back by `sd-solo -cleanup`.**
> It stays held, by a user number nothing is behind, until SD itself is
> restarted — a defect, and it is recorded in the project's fix lists. If a job
> guarded by a task lock will not start again after a crash, that is the first
> thing to suspect, and clearing it needs `admin` and `unlock tasklock`.

## Which verbs need `admin`

| | |
|---|---|
| **no `admin`** | `release` |
| **`admin` first** | `list.readu` `list.locks` `lock` `clear.locks` `unlock` |

**The split is between your locks and everybody's.** Giving back what you hold
is something any session may do. Looking at the computer's lock table, taking a
numbered flag, or forcing another session's lock open are administrator acts.

## See also

[SD Basic - Locks and Transactions](14-sd-basic-locks-and-transactions.html) ·
[SD TCL - Processes and Phantoms](30-sd-tcl-processes-and-phantoms.html) ·
[SD TCL - Files and Records](20-sd-tcl-files-and-records.html).
