Title: SD Client API
Subtitle: The sdclilib client library, its functions across four language bindings, and the server status codes.

The SDClient API lets an external application connect to SD, open
files, read and write records, execute commands, call subroutines,
and manage select lists — all through a single shared library.

**The API is a normal way to use SD**, not a facility reserved for developers
and administrators. A person running a custom GUI program that talks to SD needs
API access and may need nothing else. **On SD Core for Linux Solo the API is off
unless you chose it when installing** (always on in managed mode), and it signs
in to the one account, `sduser`, with the account password — see *API access* in
the GettingStarted set.

## The library

Two names are installed, built from one source in one step:

| | |
|---|---|
| `sdclilib.so` | what existing applications ask for |
| `libsdcli.so` | the ordinary Unix `lib*.so` convention (`-lsdcli` at link time) |

**This port carries no 32-bit build** — a difference from SD Core for
Windows, which ships a separate pair for QM-heritage applications.

### Where it is, and how to use it

Built to `bin/` under the source tree, installed alongside the server
under `~/SDCoreSolo/bin`. Link against it (`-lsdcli`) or load it at
runtime (`dlopen("sdclilib.so", ...)`) the ordinary way for a Linux shared
library — there is no PATH-order concern the way a Windows DLL search has,
because Linux resolves a shared library by its own rules (`rpath`,
`LD_LIBRARY_PATH`, or the standard library paths), not by which directory
happens to be searched first.

The header, `sdclilib.h`, ships with the source tree at
`sdb_ai/sd64/gplsrc/sdclilib`.

## Connection

| | |
|---|---|
| `SDConnect(host, port, user, pass, account)` | over the network, to port **4249**. Use `127.0.0.1` for this computer; `user` is `sduser` |
| `SDConnectLocal(account)` | **disabled on Solo.** Returns false at once |

> **`SDConnectUDS` (Unix Domain Socket) is not available, on either
> port.** It sent its credential over the socket in a way that predates
> SCRAM, and was removed for the same reason the old cleartext API login was.

### Why SDConnectLocal is disabled

`SDConnectLocal` sends no password, and every SD Core for Linux Solo session
proves the account password. It also could not work: it `fork()`s and `exec()`s
the server at a path it finds from `/etc/sd.conf` (or `$SD_CONFIG`), which a Solo
tree does not have — so before it was disabled it **hung for ever** waiting on a
child that never started. It now returns false immediately and the error text
(`SDError()`) reads *SDConnectLocal is not available in SD Core for Linux Solo -
connect with SDConnect and the account password*.

**The login is SCRAM-SHA-256.** A client that sends a password in
clear is refused. The server sets a puzzle only someone who knows the
password can answer, and the password itself is never sent in any
form. The server also proves itself to the client — another program
that grabbed the port before SD started cannot pretend to be SD.

**A session is confined to its own account** on the multiuser product: an API
session can open everything inside its own account and the shipped SDSYS files
every account needs, but cannot open, rename, delete or list anything else. **That
confinement has not been measured on Solo**, so do not rely on it there — see
*API access* in the GettingStarted set.

## Server status codes

| Constant | Value | Meaning |
|---|---|---|
| `SV_OK` | 0 | success |
| `SV_ON_ERROR` | 1 | an `ON ERROR` clause fired |
| `SV_ELSE` | 2 | an `ELSE` clause fired |
| `SV_ERROR` | 3 | an error occurred |
| `SV_LOCKED` | 4 | the record is locked by another session |
| `SV_PROMPT` | 5 | the command issued a prompt and is waiting for input |

## Function reference

### Connection management

| Function | Returns | Description |
|---|---|---|
| `SDConnect(host, port, user, pass, account)` | Boolean | connect over TCP |
| `SDConnectLocal(account)` | Boolean | **disabled on Solo** — always false |
| `SDConnected()` | Boolean | is a session active |
| `SDDisconnect()` | none | end the current session |
| `SDDisconnectAll()` | none | end all sessions |
| `SDGetSession()` | Integer | get the current session number |
| `SDSetSession(session)` | Boolean | switch to a session |
| `SDLogto(account)` | Boolean | switch to another account |

### File operations

| Function | Returns | Description |
|---|---|---|
| `SDOpen(filename)` | Integer (file number) | open a file |
| `SDClose(fileNo)` | none | close a file |
| `SDRead(fileNo, id, err)` | String | read a record |
| `SDReadl(fileNo, id, wait, err)` | String | read with a shared lock |
| `SDReadu(fileNo, id, wait, err)` | String | read with an update lock |
| `SDWrite(fileNo, id, data)` | none | write a record |
| `SDWriteu(fileNo, id, data)` | none | write with an update lock |
| `SDDelete(fileNo, id)` | none | delete a record |
| `SDDeleteu(fileNo, id)` | none | delete with an update lock |
| `SDRecordlock(fileNo, id, updateLock, wait)` | none | set a record lock |
| `SDRelease(fileNo, id)` | none | release a record lock |
| `SDMarkMapping(fileNo, state)` | none | enable/disable mark mapping |

### Record manipulation

| Function | Returns | Description |
|---|---|---|
| `SDExtract(src, fno, vno, svno)` | String | extract a field, value or subvalue |
| `SDIns(src, fno, vno, svno, newdata)` | String | insert into a dynamic array |
| `SDDel(src, fno, vno, svno)` | String | delete from a dynamic array |
| `SDReplace(src, fno, vno, svno, newdata)` | String | replace a field, value or subvalue |
| `SDLocate(item, src, fno, vno, svno, pos, order)` | Boolean | find a position for ordered insert |

### String functions

| Function | Returns | Description |
|---|---|---|
| `SDField(str, delim, first, occurrences)` | String | extract a field from a delimited string |
| `SDDcount(src, delim)` | Long | count delimiters |
| `SDChange(str, old, new, occurrences, start)` | String | replace substrings |
| `SDMatch(src, template)` | Boolean | match a string against a pattern |
| `SDMatchfield(src, template, component)` | String | extract a matched component |
| `SDSubstr(...)` | String | substring (language-specific) |

### Command execution

| Function | Returns | Description |
|---|---|---|
| `SDExecute(cmnd, err)` | String | execute a TCL command |
| `SDRespond(response, err)` | String | respond to a prompt from `SDExecute` |
| `SDEndCommand()` | none | end a multi-line command |
| `SDCall(name, argcount, ...)` | none | call a catalogued subroutine (pass by reference) |
| `SDCallx(name, argcount, ...)` | Integer | call a catalogued subroutine (pass by value) |

> `SDCall` and `SDCallx` are variadic: 0 to 20 arguments. Each binding
> provides per-arity wrappers — `SDCall0` through `SDCall20`, and
> `SDCallx0` through `SDCallx20`.

### Select lists

| Function | Returns | Description |
|---|---|---|
| `SDSelect(fileNo, listNo)` | none | select all records in a file |
| `SDSelectIndex(fileNo, indexName, indexValue, listNo)` | none | select by index value |
| `SDSelectLeft(fileNo, indexName, listNo)` | String | select left of an index cursor |
| `SDSelectRight(fileNo, indexName, listNo)` | String | select right of an index cursor |
| `SDSetLeft(fileNo, indexName)` | none | position cursor left |
| `SDSetRight(fileNo, indexName)` | none | position cursor right |
| `SDReadNext(listNo, err)` | String | read next select list id |
| `SDReadList(listNo, err)` | String | read the whole list as a dynamic array |
| `SDClearSelect(listNo)` | none | clear a select list |
| `SDRelease(fileNo, id)` | none | release a read lock |

### Session and package management

| Function | Returns | Description |
|---|---|---|
| `SDEnterPackage(name)` | Boolean | enter a package context |
| `SDExitPackage(name)` | Boolean | exit a package context |
| `SDGetArg(argNo)` | String | get a command argument |

### Error handling

| Function | Returns | Description |
|---|---|---|
| `SDError()` | String | get the last error message |
| `SDDebug(mode)` | none | enable/disable debug mode |
| `SDStatus()` | Long | get the last server status code |
| `SDFree(ptr)` | none | free a returned pointer |

## Language bindings

The API headers define bindings for four languages. The function
signatures are identical in meaning; the syntax differs.

### Gambas3

```
Library "./sdclilib"
Extern SDConnect(Host As String, Port As Integer, UserName As String,
  Password As String, Account As String) As Boolean
Extern SDRead(FileNo As Integer, Id As String,
  ByRef Errno As Integer) As String
```

### PureBasic

```
PrototypeC.l P_SDConnect(host.p-utf8, port.l, user.p-utf8,
  pass.p-utf8, account.p-utf8)
PrototypeC.i P_SDRead(fno.l, id.p-utf8, *err)
```

Per-arity prototypes are provided for `SDCall` and `SDCallx`:
`P_SDCall0` through `P_SDCall20`, `P_SDCallx0` through `P_SDCallx20`.

### Free Pascal / Lazarus

```pascal
type
  TSDConnectFn = function(Host: PAnsiChar; Port: LongInt;
    User, Pass, Account: PAnsiChar): LongInt; cdecl;
  TSDReadFn = function(FileNo: LongInt; Id: PAnsiChar;
    Err: PLongInt): PAnsiChar; cdecl;
```

A `LoadSdCliLib` function loads the library and resolves all
function pointers; `UnloadSdCliLib` releases it.

### Python (ctypes)

```python
import ctypes
_lib = ctypes.CDLL('./sdclilib.so')

SDConnect = _lib.SDConnect
SDConnect.argtypes = [ctypes.c_char_p, ctypes.c_int,
    ctypes.c_char_p, ctypes.c_char_p, ctypes.c_char_p]
SDConnect.restype = ctypes.c_int

SDRead = _lib.SDRead
SDRead.argtypes = [ctypes.c_int, ctypes.c_char_p,
    ctypes.POINTER(ctypes.c_int)]
SDRead.restype = ctypes.c_char_p
```

Per-arity `CFUNCTYPE` definitions are provided for `SDCall` and
`SDCallx` wrappers.

## What the API cannot do

| | |
|---|---|
| `sh` and `OS.EXECUTE` | **not refused over the API on this port** — unlike SD Core for Windows, both run at the account's own Linux permissions the same as any other session. `SDCLIENT` in `sd.conf` is the configurable control, if a site wants one; see *Configuration* in the GettingStarted set |
| Open files outside the account | refused on the multiuser product (status 3035 — *not permitted*); **not measured on Solo** |
| Reach the credential file | never, and cannot be added |
| Enumerate accounts | refused; all three failure cases give the same message |
