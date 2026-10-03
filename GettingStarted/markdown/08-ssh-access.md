Title: ssh access
Subtitle: Reaching SD on this computer over ssh, on Solo's own port, what the installer sets up, and what it costs.

**An ssh sign-in on port 4251 lands inside SD.** SD Core for Linux Solo runs its own
small ssh server for itself. You sign in with a key, and instead of a shell you
get `sd-solo`, which asks for the account password.

```
ssh -p 4251 you@this-computer
Password:                            <- SD's account password
:
```

## How it is arranged (LS1.1-3)

| | |
|---|---|
| **Its own port, 4251** | fixed, not an option — like the API's port 4249. The computer's own ssh server, on port 22, is **not used and not touched** |
| **Run by you, with no `sudo`** | systemd starts one `sshd` for each connection, as your own user, from a socket on port 4251. Nothing stays running between connections |
| **Key login only** | an ssh server that is not root cannot check your Linux password, so there is no password login on this port. SD then asks for the account password, as it does everywhere |
| **Nothing else** | every sign-in runs `sd-solo`: no shell, no `scp` or `sftp`, no port forwarding |

**Because the two ssh servers are separate, you can have both products.** If you
also use SD Core on this computer, you reach that one on port 22 and this one on
port 4251. Before LS1.1-3 both went through the computer's ssh server and a person
who used both could not tell them apart.

## Turning it on

| | |
|---|---|
| **At install** | `--ssh off` (the default), `--ssh local` (this computer only) or `--ssh open` (reachable from the network). Giving a key with `--ssh-key FILE` turns it on, `local`, unless you say otherwise. A managed computer is always `open` |
| **Later** | `bash ~/SDCoreSolo/tools/solo-service.sh ssh ~/SDCoreSolo local` (or `open`, or `off`) |
| **After an upgrade** | ssh is on, `local`, only if you used ssh before; everything it keeps is below |

**The installer installs the ssh server package** when ssh is on (`openssh-server`, which provides
the `sshd` program, with your `sudo`), and on Debian and Ubuntu installing it also starts the computer's
own ssh server on port 22. That is the package's doing; the installer does not change it.

## Keys

```
bash ~/SDCoreSolo/tools/solo-ssh.sh key-add    ~/SDCoreSolo  ~/.ssh/id_ed25519.pub
bash ~/SDCoreSolo/tools/solo-ssh.sh key-list   ~/SDCoreSolo
bash ~/SDCoreSolo/tools/solo-ssh.sh key-remove ~/SDCoreSolo  ~/.ssh/id_ed25519.pub
```

**Solo has its own key file, `~/SDCoreSolo/sshd/authorized_keys`.** Your own
`~/.ssh/authorized_keys` is not used for this port, so a key you add for Solo gives
nothing on the computer's ssh server, and the other way round. Each line is the key with
two options in front:

```
restrict,pty ssh-ed25519 AAAA… you@laptop
```

| | |
|---|---|
| `restrict` | turns off every forwarding and agent facility for that key |
| `pty` | gives `sd-solo` the terminal it needs |

**Adding the same key twice adds one line.** The `sshd` directory is created by
`solo-ssh.sh setup` (the installer runs it), and holds the generated `sshd_config`, Solo's own
host key and the key file. **Do not edit `sshd_config`: it is rewritten whenever `setup`
runs.** What it says:

| | |
|---|---|
| `ForceCommand ~/SDCoreSolo/bin/sd-solo` | every sign-in runs `sd-solo`, whatever the client asks for |
| `PasswordAuthentication no`, `AuthenticationMethods publickey` | keys only |
| `AllowUsers you` | only your Linux user |
| `DisableForwarding yes` | no port forwarding, which `ForceCommand` alone would not stop |
| `StrictModes yes` | see below |

**`StrictModes`.** ssh refuses a key file when it, or a folder above it up to your home folder, is
owned by somebody else or writable by everybody. `solo-ssh.sh setup` checks the same path and says
which one to fix. (On Ubuntu with its one-person groups, a mode of `0664` or `0775` was accepted
and `0666` or `0777` was refused; a group with other members may be stricter.)

## On a managed computer: the server's key (LS1.1-2)

**You do not add the server's key.** On a managed computer the server installs it
itself, over the API, after signing in with the global password; the line it
writes goes in the same key file and is the same kind as the one above, so that key reaches
`sd-solo` and nothing else. At most four such lines are kept. **From LS1.1-3 the answer
names Solo's own ssh server: the fingerprint of its host key, and the port, 4251.** `key-list`
shows every key line SD wrote, the server's included. See
[Managed mode](15-managed-mode.html).

## If you upgrade from an earlier release

**Earlier releases had two other ways in, and both are gone:** a forced-command line in your
`~/.ssh/authorized_keys`, and an optional block in the computer's `sshd_config.d` that made every
ssh login of yours, password included, land in SD (`--ssh-match`). The upgrade:

- **moves the Solo key lines out of your `~/.ssh/authorized_keys`** into Solo's key file, after
  keeping a copy of the old file next to it (`authorized_keys.sdsolo-backup-<time>`). Your other keys
  are not touched;
- **turns the new listener on, `local`**, if you had used either way in;
- **tells you the one command that removes the old block**, which needs `sudo`:

```
bash ~/SDCoreSolo/tools/solo-ssh.sh match ~/SDCoreSolo --remove
```

**Remove it.** Solo no longer uses it, and while it is there it still sends your port-22 ssh
logins into Solo.

## The cost: no password, no shell, no scp or sftp

**On this port there is no password login, no shell and no file copy.** The command is
forced, so there is no file-transfer subsystem left to run and no shell. That is the accepted
cost of landing in SD.

**The cost is inbound only.** `scp` or `rsync` running **on** this computer, connecting
outward, is an ssh *client* and is not affected. So copy files by pulling them from here — or
use the computer's own ssh server, on port 22.

**To reach Linux from an ssh session, use `sh`** inside SD — see
[Operating system access](06b-operating-system-access.html).

## Reaching the computer from the network

| | |
|---|---|
| **`local`** | the listener is `127.0.0.1:4251`: `ssh -p 4251 localhost` works, nothing else can connect, and no firewall rule is needed |
| **`open`** | the listener is `0.0.0.0:4251`. **Allow TCP 4251 in the firewall.** The installer adds a `ufw` rule if `ufw` is active and you let it use `sudo`; otherwise it tells you |
| **Managed** | always `open`: the SD Core for Linux server has to reach port 4251 |

**The installer does not open port 22**, and Solo does not use it.

## What an ssh session is

**It is you**: the same account, `sduser`, and the same rights as a session at
the keyboard.

**On a computer installed from a control file**, before the account password has
been chosen at the keyboard, an ssh session is told:

```
This account has no password yet. Set it at this computer's keyboard first; until then only the global password is accepted.
```

**SD has to be running for the forced `sd-solo` to find it.** The ssh unit needs
`sd-solo.service`, and with linger on it always runs. Without linger your systemd user manager, and SD
with it, start when you first sign in — **that has not been measured for an ssh sign-in**: if a first
connection is told *SD has not been started*, connect again, and enable linger (see
[Running SD](03-running-sd.html)) if ssh is how you mostly reach this computer.

## Measured, and not measured

**Measured on one computer (Ubuntu 26.10, OpenSSH 10.5), 2 October 2026:**

- a key login through the generated configuration reaches the forced command and a command the client sends is not run;
  a key that is not in the file, and a password, are refused; forwarding is refused, by the configuration and by `restrict`
  each on its own, and a control with both removed lets it through; `StrictModes` refuses a key file at `0666`;
- the systemd units start one `sshd` per connection and leave no failed units;
- **a real `sd-solo` session over port 4251** on the installed Solo: the password was asked, `WHO` answered
  `sduser`, a stranger's key was refused, and port 4251 presented Solo's own host key while port 22 presented a
  different one;
- an upgrade from the previous release: the Solo key line moved to the new file with its backup, the units
  came up, and removing the old block was done by hand.

**Not measured:** reaching port 4251 from another computer (`open`) and the firewall rule; a sign-in after a
restart with linger off; a computer without the `ufw` firewall; a distribution other than Ubuntu.
