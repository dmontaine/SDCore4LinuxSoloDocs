Title: Running SD
Subtitle: How SD starts, stopping it, the command line, and what to do when the last shutdown was not a clean one.

**SD is your own systemd user service, and it is already running.** You do
not type `sd-solo -start` after every sign-in.

| | |
|---|---|
| Unit | `sd-solo.service` starts the daemon; with an API, `sd-solo-api.socket` is the listener and `sd-solo-api@.service` runs one API session per connection |
| Where | `~/.config/systemd/user/` — **user** units, started by your own systemd user manager and running as you |
| Start type | enabled — your systemd user manager starts them when it starts |
| Created by | the installer (`tools/solo-service.sh install`) |
| Removed by | the uninstaller |

**The user manager starts when you first sign in and stops when your last
session ends — unless linger is on.** With linger enabled, SD starts at boot
and keeps running while you are signed out, which is what remote access
(ssh or the API when nobody is at the keyboard) needs. It is a persistent
setting of your account, so the installer asks rather than assuming; to turn it
on later, as an administrator of the computer:

```
sudo loginctl enable-linger $USER
```

**Without linger**, `sd-solo` still works while you are signed in — SD is started
with your user manager — but a scheduled job or an API client that arrives
after you have signed out finds nothing running.

## Starting and stopping

```
systemctl --user stop sd-solo.service
systemctl --user start sd-solo.service
```

```
sd-solo -stop
sd-solo -start
```

**Stopping SD ends every session**, including API and ssh sessions. **Neither
needs `sudo`**: SD, its data and its programs are yours. If you stop it with
`sd-solo -stop` while systemd thinks it is running, `systemctl --user status`
still says *active* (the unit is a one-shot that remains after exit); use
`systemctl --user stop` when you want systemd to know.

## `sd-solo -start` and `sd-solo -stop` check the real process, not just the segment

SD's shared state lives in a System V shared-memory segment, which — unlike a
file — can outlive the daemon process that created it if SD is killed rather
than stopped cleanly. `sd-solo -start` and `sd-solo -stop` validate the daemon's actual
process id rather than trusting the segment's mere existence:

**"SD is already started" is only said when the daemon really is running**, and
it names the process id — the one `ps` and `kill` use.

**If the segment is there but the daemon is not** — what a killed or crashed SD
leaves behind — `sd-solo -start` says so rather than silently reporting success
against a segment nothing is serving. Clearing it is `sd-solo -stop`'s job.

> Check the daemon by hand at any time:
>
> ```
> ps -u $USER -o pid,cmd | grep '[s]dlnxd'
> ```

**A System V segment does not survive a reboot** — the kernel clears its IPC
state on every restart. A reboot always leaves a clean slate; the "segment
present, daemon dead" case above is about a crash the computer did **not**
restart from.

**Only one SD can run on a computer.** The shared-memory name is machine-wide,
so a second SD — Solo for another user, or the multiuser SD Core for Linux —
would attach to the first one's segment. That is why the installer refuses to
run beside the multiuser product.

## The command line

```
sd-solo                  enter the account, after the account password
sd-solo <command>        run one command and return, using the kept password
sd-solo -a               the same as sd-solo: there is one account
sd-solo -asduser         enter that account (any other name is refused)
sd-solo -quiet           suppress the displays on entry
sd-solo -u               list current sessions
sd-solo -k <n> | -k all  end session n, or every session
sd-solo -start           start SD
sd-solo -stop            stop SD
sd-solo --version        report the version
sd-solo --help           a summary of these
```

**`sd-solo <command>` runs and exits, with no prompt.** It uses a copy of the
account password kept for you (see
[The account and its passwords](05-account-types.html)), which is what lets a
script or a scheduled job use SD. **If that copy is missing or no longer
matches**, what happens depends on where the input comes from:

| | |
|---|---|
| typed at a terminal | refused: *A command given on the sd command line needs the account password on its input* |
| piped in | the first line of the input is taken as the password, once — so a job can supply it itself |

`SET.PASSWORD` keeps the copy up to date when you change the password.

**`sd-solo -a` and `sd-solo -a<name>` have nothing to choose between**: there is one
account, `sduser`, and every session lands in it. `sd-solo -asdsys` is refused,
saying that SDSYS is not entered in Solo.

**`sd-solo -u` and `sd-solo -k` need no `ADMIN`.** They are switches on the program,
outside any SD session, and like `-start` and `-stop` they are the business of
the user who owns SD. Inside a session, the same jobs are `LISTU` and
`LOGOUT`, which do need `ADMIN` — see
[Sessions and locks](06a-sessions-and-locks.html).

**`sd-solo` refuses to run as root**, and says so before anything else. Solo has no
use for root; running it as root would leave files in your tree that you could
not then manage.

## Where things are

Everything is under `~/SDCoreSolo` (or the `--home` you gave the installer):

| | |
|---|---|
| Programs | `bin/` — `sd-solo`, `sdlnxd` (the daemon), `sdtic`, `sdconv`, `sdfix`, `sdidx`, `sdclilib.so` |
| The changelog | `changelog` |
| Configuration | `sd.conf` |
| SD's own files | the rest of the directory: `gcat/`, `messages/`, `voc`, `$cred/`… |
| Your account | `user_accounts/sduser/` |
| Audit trail | `audit` |
| Error log | `errlog` |
| What was installed | `.sdcore-install` |

**The whole directory can be moved; its parts cannot be separated.** SD finds
its files from where its programs are (the directory that holds `bin/`, and
that holds the marker file `.sdcoresolo`), so a copied tree works from another
place — but the service units, the `~/.local/bin/sd-solo` link and any ssh key line
name the old path and must be redone.

## Checking the service

```
systemctl --user status sd-solo.service
journalctl --user -u sd-solo.service
```

`systemctl` reports whether systemd thinks the unit is running; `sd-solo -u` reports
who is actually connected.
