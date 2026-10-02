Title: ssh access
Subtitle: Reaching SD on this computer over ssh, what the installer sets up, and what it costs.

**An ssh sign-in can land inside SD.** ssh checks who you are as it would for
any sign-in; then, instead of a shell, you get `sd-solo`, which asks for the account
password.

```
ssh you@this-computer
Password:                            <- SD's account password
:
```

## Two ways to set it up

| | Needs `sudo` | Whose logins land in SD |
|---|---|---|
| **The key line** — the default | no | logins with that key only. Every other way in, and every other key you own, is unchanged |
| **The `Match` block** — optional | yes | **every** ssh login of your user, key or password, and your user then has no shell over ssh at all |

**Both are done by `tools/solo-ssh.sh`**, which the installer runs when you give
it a key (`--ssh-key FILE`, or the `ssh-public-key-file` line of a control file)
and, for the block, `--ssh-match`. It can be run again at any time.

### The key line

```
bash ~/SDCoreSolo/tools/solo-ssh.sh key-add    ~/SDCoreSolo  ~/.ssh/id_ed25519.pub
bash ~/SDCoreSolo/tools/solo-ssh.sh key-list   ~/SDCoreSolo
bash ~/SDCoreSolo/tools/solo-ssh.sh key-remove ~/SDCoreSolo  ~/.ssh/id_ed25519.pub
```

`key-add` appends the public key to your `~/.ssh/authorized_keys` with these
options in front:

```
command="/home/you/SDCoreSolo/bin/sd-solo",restrict,pty ssh-ed25519 AAAA… you@laptop
```

| | |
|---|---|
| `command=` | **that key runs `sd-solo` and nothing else.** Whatever the client asks to run — a shell, `scp`, `sftp` — it gets `sd-solo` instead |
| `restrict` | turns off every forwarding and agent facility for that key |
| `pty` | gives `sd-solo` the terminal it needs |

**It never touches your other keys.** A key without the forced command still
gets a shell, exactly as before — which is the point of choosing this route by
default. **Adding the same key twice adds one line**, because the lines are
recognised by their command string, and `key-remove` and `key-list` find those
lines and no others.

**This is the first thing to check if a key still gives you a shell**: a copy
of the key that is also in `authorized_keys` *without* the forced command wins,
because ssh takes the first line that matches.

### The `Match` block

```
bash ~/SDCoreSolo/tools/solo-ssh.sh match ~/SDCoreSolo              # print what it would write
bash ~/SDCoreSolo/tools/solo-ssh.sh match ~/SDCoreSolo --apply      # write it (needs sudo)
bash ~/SDCoreSolo/tools/solo-ssh.sh match ~/SDCoreSolo --remove
```

It writes one file, `/etc/ssh/sshd_config.d/50-sd-solo-<your user>.conf`:

```
Match User you
    ForceCommand /home/you/SDCoreSolo/bin/sd-solo
    DisableForwarding yes
```

| | |
|---|---|
| `Match User` | **only your Linux user** is affected. Everyone else signs in as before |
| `ForceCommand` | every ssh login of yours runs `sd-solo` and nothing else |
| `DisableForwarding` | no port forwarding for you, which `ForceCommand` alone would not stop |

**The change is checked before it stays**: `sshd -t` must accept the new
configuration before the ssh server is reloaded, and if it does not, the file is
put back. `--apply` refuses if `sshd_config` has no `Include
/etc/ssh/sshd_config.d/*.conf` line, since the block would then do nothing. The
uninstaller removes the block, and leaves the rest of the sshd configuration as
it was.

> **Measured, and not measured.** The text of the block and its syntax (`sshd -t`)
> are tested, and **writing it into a real `/etc/ssh` through the installer's
> `--ssh-match` has been run once** (30 Sep 2026): the file appeared, and an ssh
> login with a key reached `sd-solo` with no shell. **Removing it with `--remove`, and a
> *password* login landing in SD, have not been run.** Check that your ssh password
> login lands in SD before relying on it, and keep another way into the computer
> open while you do.

## On a managed computer: the server's key (LS1.1-2)

**You do not add the server's key.** On a managed computer the server installs it
itself, over the API, after signing in with the global password; the key line it
writes is the same kind as the one above (`command="…/bin/sd-solo",restrict,pty`), so
that key reaches `sd-solo` and nothing else. At most four such lines are kept, and the
keys you added yourself are never touched. `key-list` shows every key line SD wrote,
the server's included. See [Managed mode](15-managed-mode.html).

## The cost: no scp or sftp to that key, and no shell

**A key with the forced command cannot copy files in or give you a shell.** The
command is forced, so there is no file-transfer subsystem left to run and no
shell. That is the accepted cost of landing in SD.

**The cost is inbound only.** `scp` or `rsync` running **on** this computer,
connecting outward, is an ssh *client* and is not affected. So copy files by
pulling them from here — or use another key for file transfer.

**To reach Linux from an ssh session, use `sh`** inside SD — see
[Operating system access](06b-operating-system-access.html).

## Reaching the computer from the network

**The installer does not install or start an ssh server, and does not open port
22.** Those are the computer's, not yours. If none is running it warns and prints
the command (`sudo systemctl enable --now ssh`, or `sshd` on some distributions).

| | |
|---|---|
| **Standalone** | ssh is optional. `ssh localhost` works whenever an ssh server is running, with no firewall rule |
| **Managed** | ssh must be reachable from the SD Core for Linux server, so the server's address has to be able to reach port 22. Managed mode needs the ssh server running; the installer says so if it is not |

## What an ssh session is

**It is you**: the same account, `sduser`, and the same rights as a session at
the keyboard.

**On a computer installed from a control file**, before the account password has
been chosen at the keyboard, an ssh session is told:

```
This account has no password yet. Set it at this computer's keyboard first; until then only the global password is accepted.
```

**SD has to be running for the forced `sd-solo` to find it.** With linger on it always
is. Without linger your systemd user manager, and SD with it, start when you
first sign in, and an ssh sign-in counts — but **that has not been measured**: if
a first connection is told *SD has not been started*, connect again, and enable
linger (see [Running SD](03-running-sd.html)) if ssh is how you mostly reach this
computer.
