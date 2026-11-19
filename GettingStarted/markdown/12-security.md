Title: Security
Subtitle: Who can reach SD Core Solo for Linux, what each password guards, and what does not protect it.

SD Core Solo for Linux is built for **one Linux user on one computer**, and its
protection follows from that. Read this page before relying on it for anything
beyond that.

## The position in one paragraph

**Your Linux user owns everything**: the programs, the data, the account and the
credential files all live in `~/SDCoreSolo`, and every SD process runs as you.
**Linux's own file permissions are what keep other Linux users out** — the
installer makes the directory mode 0700, and every file in it private. **SD's
passwords keep other *people* out of SD** — at the keyboard, over ssh and through
the API — and its administrator gate keeps a session from doing more than it
should by accident. **Neither stops somebody who is already signed in to Linux as
you**, or `root`, from reading or changing the files directly.

## What each thing guards

| | Guards | Does not guard |
|---|---|---|
| **Linux** | your directory from other Linux users | it from you, and from `root`, who can read `~/SDCoreSolo` — the credential store included |
| **The account password** | every SD session: keyboard, ssh, API, and a command line | the files themselves |
| **`admin`** | the administrator commands and direct VOC edits, in a session | a program that reads the files |
| **The `sh` and `os` switches** | the way out of SD onto the computer from inside a session: the `sh` verb, `OS.EXECUTE`, files and directories outside the account, the embedded Python, `micro` — off as delivered | what a program that already runs as you does outside SD |
| **The global password** | a managed computer, for the SD Core for Linux server | — |
| **The deny list** | commands the user may not run on a managed computer | the same files, outside SD |

**The account password is the gate SD adds.** Being signed in to Linux as you is
not enough to get an SD session: SD asks. That matters when somebody else has
your Linux session for a moment, and above all for ssh and the API, which are
reached from other computers. See [The account and its passwords](05-account-types.html).

## What ships secured

| | |
|---|---|
| Every session | asks the account password |
| Administrator commands | refused until `admin`. The eight that had no check of their own — `config`, `listu`, `list.locks`, `list.readu`, `lock`, `clear.locks`, `set.date`, `clean.account` — are gated too. See [Administrator commands](06-administrator-commands.html) |
| The VOC | direct edits need `admin`; the global catalog is changed by nobody in a session |
| The shell and OS access | refused until the administrator turns the `sh` or `os` switch on with `set.os.access`: off on a new install unless you answered yes when the installer asked, and kept as they were on an upgrade — see [Operating system access](06b-operating-system-access.html) |
| `NETDIRS` | a door in `sd.conf`, **no line as delivered**: folders outside the account, separated by semicolons, that a session with the `os` switch off may still read and write by pathname. Listing one is a loosening for every session of the account, on every route — see [Configuration](16-configuration.html) |
| The NET gate | **off as delivered**: `set.net.access on` is what lets the account open a file held on another SD server (a VOC pointer of type `N`, to a server the administrator has defined with `set.server`). Turning it on lets the account's programs read that server's files, and change them only where the server is defined as `write`. `NETFILES` in `sd.conf` (0 as delivered) must have bit 2 set before an API client may open an `account;file` file here (request 29), and a VOC pointer that leads out of the account is refused unless the account has the `os` switch on — see [Not in SD Core](14-not-in-sd-core.html) and [API access](09-api-access.html) |
| File transfer | an sftp program on another computer can put text files into the account's file `xfer.in` and take them from `xfer.out`, over Solo's ssh (port 4251). **Off twice as delivered**: `XFER=1` in `sd.conf` (read when Solo starts) and `set.xfer.access on`. Opening it means a program that knows your Linux password can bring files into the account while you wait in `receive`; it sees only that one folder, only plain text, within the size limits (`XFERFILE`, `XFERSLOT`) and only while the transfer is open. **The SD account password is not asked of the sftp connection** (it has no terminal to give it): what stands in for it is that nothing moves unless your own session, which did give it, is waiting in `receive` or `send` at that moment — see *Moving Files In and Out* in the User documentation |
| Scheduled jobs | SD's own jobs (`set.job`) run one of your SD commands on a systemd timer of yours. **Off as delivered**: `set.jobs.access on`, and off refuses every job at its next run. A job is made or changed only with the account password typed at that moment; it runs as a one-shot command does, signed in with the kept copy of the password, so it adds no way in that `sd-solo` itself does not already give — see [Scheduled jobs](04-scheduled-jobs.html) |
| The daemon | runs as you, never as root; SD refuses to start as root |
| Files | mode 0600, directories 0700, from the installer and from every file SD creates (`umask 077`) |
| ssh | Solo's own listener on port 4251, run by you with no `sudo`; your Linux account name and password (inside ssh's encrypted channel, checked by PAM) or a key, forced into `sd-solo`, no shell, no forwarding; on by default for this computer only, and three wrong passwords from one address lock that address for ten minutes — see [ssh access](08-ssh-access.html) |
| The API | off unless chosen at install (managed or not); SCRAM inside TLS 1.3; only the one account — see [API access](09-api-access.html) |
| `sd-solo -internal` | closed once the installer has finished |

**The account is the same one for every session**, so the gates are about *how
you arrived and what you unlocked*, not about which account you are in.

## What SD keeps

| | |
|---|---|
| **The passwords** | none is stored as such. `$cred` holds a verifier for each — the account, the administrator and the global — that cannot be turned back into a password. It is mode 0700, its files 0600 |
| **The kept copy** | a copy of the account password **in clear**, in `$cred/$stored`, that lets SD's scheduled jobs (`run.job`) and the MCP door's calls (`mcp.call`) sign in without typing. **Restricted as delivered**: any other `sd-solo <command>` needs the password on its input, until the administrator says `set.stored.access on`. Linux has nothing that lets a job with no session unlock a secret for you, so it cannot be encrypted in a way the job could then open. **Anyone who can read your files can read it**, as they could read a `~/.pgpass`; it keeps it from other Linux users, not from your other programs |

**Whoever can replace a verifier can set a password they know**, and anyone who
is your Linux user, or `root`, can. That is the limit stated above, not a fault:
the passwords protect SD sessions, not the files under them.

## The installer's own door

**`sd-solo -internal` is how the installer runs SD's setup steps**, and it is admitted
only by a one-shot marker file the installer writes immediately before each step
and SD consumes — the audit trail shows *internal session admitted* with the name
of the writer. A marker older than ten minutes is refused, and consumed. It is
not a way into a running system: the installer checks, when it finishes, that a
plain `sd-solo -internal` is refused. **SDSYS, SD's own system account, is never
signed in to**: nobody logs in to it, and there is no `sdsys` Linux user.

## On a managed computer

**The SD Core for Linux server can add to what is locked** — the global
catalog, the list of denied commands — and it signs in with the global
password. **All of that is SD enforcing it, and none of it is Linux enforcing
it**: the user of the computer owns the files and can change them from outside
SD. What managed mode protects is what happens *inside* SD. See
[Managed mode](15-managed-mode.html).

**The server can also put an ssh key on the computer.** Only a session
signed in with the global password may, at most four keys are kept, each can start
`sd-solo` and nothing else, and every use is audited. That is the global password
reaching one step further — into Solo's own ssh key file — so **whoever
holds the global password can also reach this computer over ssh as `sduser`**.
It could already run the administrator commands. **On the client side, the library
pins the computer's TLS certificate on first use**, which stops someone posing as
the computer from collecting a login proof to crack offline; the first connection
is trusted. See [API access](09-api-access.html).

## What you can do further

**Lock a session into one application** by removing `basic` and `run` from the
VOC (which needs `admin`) and turning off its break key (`pterm break off`), so
it can neither compile nor run anything else and cannot interrupt out to a TCL
prompt. Neither is a setting; both are done by hand.

**Turn the API off** if you do not use it — it is off unless it was chosen at
install, with or without a global password; see [API access](09-api-access.html). **Restrict ssh to this
computer** (`remote.ssh local`) if other computers do not need it, or **leave it off**
(`remote.ssh off`) — see [ssh access](08-ssh-access.html). When Solo's ssh server is on it
runs on port 4251, your Linux account name and password land you in SD. Anyone who can reach the port can try
passwords for your Linux account, so keep it limited to this computer unless you need it
open. **Do not give the ssh key line to anything you would not give the account password
to** — see [ssh access](08-ssh-access.html).

**Enable full-disk encryption and a screen lock**, as you would for any computer
that holds data you care about: everything above assumes the computer itself is
in your hands.

## Continued in

[Security and the operating system](12a-security-and-the-operating-system.html) —
reaching the operating system from inside SD, and the audit trail.
