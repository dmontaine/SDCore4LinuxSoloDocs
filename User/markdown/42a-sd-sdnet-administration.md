Title: SDNet - Administering a Remote File
Subtitle: Run the administration verbs, FCONTROL and CREATING.SEQKEY on a file of another SD server, and query its indices; the waits, the audit and what is not here.

This page continues [SDNet - Files on Another SD Server](42-sd-sdnet.html).

## Administering a remote file

These do on the other machine what they do on a local file, **for your remote login and in the account that login entered
there**, so every switch that account has on that machine applies (on an SD Core server a file named `voc`, `*.bp` or
`*.dic` still needs that account's edit switch there). They need **a `write` definition here, and the other machine's
permission**: an SD Core server allows them only to a remote login its administrator has put in both its write group and
its administration group (off for every account until he does); a Solo server allows them only to a session signed in
with its **global password**, and refuses one signed in with the account password with *This session cannot administer
files here: sign in with the global password* (message 14964). A Solo with no global password (the delivered state) allows none of them.

| | |
|---|---|
| `write rec to f creating.seqkey id`, `writeu ...` | the other machine's **own counter for that file** gives the id and it comes back in `id`; the update lock on the new record is held there for `writeu` and given back for `write`. A directory file is refused as it is locally (message 11348). Not inside `begin transaction` (message 11510) |
| `fcontrol(f, key {, value})` | goes to the other machine with its key (`FC$SET.SEQ.KEY`, `FC$GET.SEQ.KEY`, `FC$VALID.ID`, `FC$SYNC`, `FC$CRLF`, `FC$MINMOD`) and returns what it returned; `status()` is what it left there. Asking whether a name is a valid record id (key 6) changes nothing, every other key is a change |
| `analyze.file`, `configure.file`, `create.index`, `build.index`, `delete.index`, `disable.index` on the VOC name of a remote file | run **on the other machine**, as the remote login's Linux user; **what the verb printed there is printed here**, and the verb's return code is the one the other machine's verb ended with. The file is the one the pointer opened (you cannot name another), and the rest of the command is passed on as it stands |
| `create.file name ... server srv {as remote}` | makes the file on server *srv* (as *remote* there when `as` is given) and, **only after that machine has confirmed it**, writes the `N` pointer *name* in your VOC: there is never a pointer to a file that is not there. `dict`, `data`, `using`, `pathname` and a comment mean nothing for a remote file and are refused (message 11528) before anything is sent; a name already in your VOC is refused (11539) before anything is sent |
| `delete.file name` where *name* is an `N` pointer | asks, as a local delete does (`no.query` does not), deletes the file on the other machine and, in the same act, the pointer here. If that machine says the file is not there (someone removed it) the pointer would dangle, so the verb offers to remove it (`no.query` removes it). If that machine refuses, the pointer stays |

**What the other machine accepts in the words.** Letters, digits and `. _ -`, single blanks, at most 200 characters.
`pathname`, `lptr`, `force`, `no.query`, a slash, a quote or two blanks in a row are refused with message 11530 and
nothing is run; `directory` is accepted only as the last word. The verb that asks questions is told not to (`create.file`
and `delete.file` get `no.query` there, since you answered here). The index verbs need the file to themselves: if another
process has it open on that machine the verb says "Cannot gain exclusive access" and does nothing, as it does locally.

**What it records.** The other machine writes `api admin request=70 op=create.index file=customers user=name` before it
runs the verb, and this machine writes `netfile admin server=nfa file=customers op=create.index` (and
`netfile fcontrol ... key=n`), so a killed command is still on both trails.

**Not available:** the statements that read or change the tree of an index (`akread`, `akwrite`, `akdelete`, `clear.ak`,
`akenable`, `akmap`) on a remote file are still refused, as are `openseq` and the other sequential-file statements. A
change is never sent twice: a lost reply is message 11509, and the program must look at the file to find out what
happened.

## Indices of a remote file

A remote file's indices are the other machine's, and the statements that *query* them work on a remote file the way
they do on a local one. They change nothing, so they need no write definition and no administration permission: a
program that may open the file may use them, on a read definition as well as a write one, and inside `begin transaction`.

| Statement | What it does on a remote file |
|---|---|
| `indices(f)`, `indices(f, name)` | the names of the indices of the file on the other machine, and the description of one (null for a name that is not an index) |
| `selectindex name from f to n` | the indexed values, as a select list; `status()` 1 and an empty list for an index that is not there |
| `selectindex name, value from f to n` | the ids that hold the value; `status()` 0 and an empty list for a value nobody holds |
| `setleft name from f`, `setright name from f` | put the scan at the left (lowest) or the right (highest) end |
| `selectright name from f setting key to n`, `selectleft ...` | the next key to the right (or left) of the scan, in `key`, with its ids as the select list; `status()` 1 and nothing when there is no further key |

The other machine keeps the position of a scan *for the open file*, so `setleft` followed by `selectright` walks the keys
in order, and `selectindex name, value` followed by `selectright` goes on from that value, as locally. Each call is one
request (a scan that finds a key makes a second one to fetch its ids), so a scan is as slow as the line between the
machines. A lost connection stops the statement with message 11497, as a lost `select` does; these statements have no
`on error` clause.

The `indices()` that every query asks of every file gets the other machine's answer, so `list`, `count` and
`select ... with` on a remote file use the other machine's index when the field in the `with` clause has one, and read
only the records the index names instead of every record. A file with no index is read as before.

## The two waits: `NETWAIT` and `NETREPLY`

Two `sd.conf` parameters, in seconds, 1 to 3600 (zero is refused): **`NETWAIT`** (5 unless you say otherwise) is how
long to wait for a connection to a remote server, **`NETREPLY`** (30) how long to wait for each reply to a request. `config('NETWAIT')`
and `config('NETREPLY')` read them.

**A server that goes silent is given up on at those times.** `NETWAIT` bounds the TCP connection and then the TLS handshake, each: a server that
accepts the connection and never answers the hello is given up on after `NETWAIT`. `NETREPLY` bounds each read, including the first reply of the login after the handshake: a server that
completes the TLS 1.3 handshake and then sends nothing, or only half of a TLS record, is waited for no longer than `NETREPLY`. A reply that stops in the middle of its body is a lost connection, like a reply that never starts: message 11497 for a read,
11509 for a change (whose outcome is then unknown), and the connection is dropped. A server that answers late but inside
`NETREPLY` is read as before. The limit is for each read, not for a whole reply: a server that sends one byte at a time and
never stops is still waited for one read at a time.

## What is audited

| line | when |
|---|---|
| `set.server name=... host=... port=... mode=... replaced=... pin=typed|confirmed` | a definition is made or replaced |
| `delete.server name=... host=... port=...` | a definition is removed |
| `set.netlogin server=... remote=...` | a login is stored |
| `delete.netlogin server=...` | a login is removed |
| `netfile open server=... file=...` (and `mode=write`) | a remote file is opened, once |
| `netfile refused server=... reason=...` | an open is refused, with the reason (gate, list, login, pin, limit) |
| `netfile clear server=... file=...` | a `clearfile` of a remote file |

No password, key, hash, record id or record is ever audited.

## What is not here

- **No files-only route on a Solo server.** SD Core has one (the Linux groups `sdfiles` and `sdfileswrite`); Solo has one
  account and one Linux user, and its API login is the account's own login. API request 29 is behind the NET gate and
  `NETFILES` bit 2, as the Getting Started set explains.
- **A remote file does not follow you across an account change**: the process's connections are dropped when the account
  is entered again (the NET gate is set), so a remote file variable from before is stale.

## See also

*API access* and *SDSYS and the administrator* (GettingStarted): `set.net.access`, API request 29 and the OS switch.
*Locks*, *Files and records* and *The query processor* (this set): the statements and verbs a remote file works with.
