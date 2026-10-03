Title: ssh access
Subtitle: Reaching SD on this computer over ssh, on Solo's own port, what the installer sets up, and what it costs.

**An ssh sign-in on port 4251 lands inside SD.** SD Core for Linux Solo runs its own
small ssh server for itself. You sign in with your Linux account name and password,
and instead of a shell you get `sd-solo`, which asks for the SD account password.

```
ssh -p 4251 you@this-computer
you@this-computer's password:        <- your Linux password
Password:                            <- SD's account password
:
```

**Your password is not sent in clear text.** ssh agrees a key exchange and a cipher with the
server first (on the test computer, `mlkem768x25519-sha256` and `chacha20-poly1305@openssh.com`),
and the password travels inside that encrypted channel. **The first time you connect, ssh shows
the server's host key fingerprint: check it** against `~/SDCoreSolo/sshd/ssh_host_ed25519_key.pub`
(`ssh-keygen -lf` on that file), because that is what stops someone posing as your computer.
The API is different: it uses SCRAM, where the server never receives the password at all.

## How it is arranged (LS1.1-3)

| | |
|---|---|
| **Its own port, 4251** | fixed, not an option — like the API's port 4249. The computer's own ssh server, on port 22, is **not used and not touched** |
| **Run by you, with no `sudo`** | systemd starts one `sshd` for each connection, as your own user, from a socket on port 4251. Nothing stays running between connections |
| **Your Linux account name and password** | checked by the computer's own login check (PAM) even though this ssh server is not root: it can check the password of the user it runs as. **It is your Linux password, nobody else's.** SD then asks for the account password, as it does everywhere. A key is an optional extra |
| **Nothing else** | every sign-in runs `sd-solo`: no shell, no `scp` or `sftp`, no port forwarding |

**Because the two ssh servers are separate, you can have both products.** If you
also use SD Core on this computer, you reach that one on port 22 and this one on
port 4251. Before LS1.1-3 both went through the computer's ssh server and a person
who used both could not tell them apart.

## Turning it on

| | |
|---|---|
| **At install** | **on, `local` (this computer only), unless you say otherwise**, as SD Core Solo for Windows does, **and the ssh server is installed for you if the computer does not have it** (as the Windows installer ticks "Install the OpenSSH server" by default). `--ssh off` declines both, `--ssh local` or `--ssh open` (reachable from the network) answers it, and the question the installer asks says plainly when answering would install the package. A managed computer is always `open`. The API is different: it is off unless you ask for it |
| **Later** | `bash ~/SDCoreSolo/tools/solo-service.sh ssh ~/SDCoreSolo local` (or `open`, or `off`) |
| **After an upgrade** | ssh is on, `local`, only if you used ssh before (an upgrade keeps what you had); everything it keeps is below |

**The installer installs the ssh server package when ssh is on and the computer does not have it** (`openssh-server`,
which provides the `sshd` program, with your `sudo`), and on Debian and Ubuntu installing it also starts the
computer's own ssh server on port 22. That is the package's doing; the installer does not change it. **If you do not
want the ssh server installed, say `--ssh off`** (or answer `off`). With `--skip-packages` the installer installs
nothing: with no `sshd` on the computer ssh is then left off, and `--ssh local` or `--ssh open` is refused at the
start, before anything is built.

## Keys (optional)

**You do not need a key.** Add one if you would rather sign in without typing the Linux
password, or on a computer that cannot type it (the SD Core server's key, on a managed computer,
is added this way too).

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
| `UsePAM yes`, `PasswordAuthentication yes` | your Linux password, checked by PAM. `PubkeyAuthentication yes` as well: a key in the key file also works |
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

## The cost: no shell, no scp or sftp

**On this port there is no shell and no file copy.** The command is
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

**`open` lets anyone who can reach port 4251 try your Linux password.** Solo's ssh checks the password of
the user who owns it (measured on the Ubuntu test computer: no lockout in that computer's own PAM setup,
each wrong try cost one to three seconds, and systemd allows 64 connections at a time and none-per-address
by default, `MaxConnections=64`, `MaxConnectionsPerSource=0`). **Solo therefore has a lockout of its own**
(below). **Use `local`** unless something has to reach this computer from elsewhere; if you must `open`
it, **restrict the firewall rule to the addresses that need it**, choose a strong Linux password, and
consider keys. The same exposure exists for the computer's own ssh server on port 22.

## The lockout: three wrong passwords lock the address

**Three wrong Linux passwords from one address within ten minutes lock that address for ten minutes.**
A connection from it is closed at once, before any password is asked, and the lock lifts by itself.

| | |
|---|---|
| **Per address, not per account** | the Linux account is never locked, so someone guessing cannot lock **you** out of your own account (which is what Windows' account lockout can do), and the computer's own login and ssh server on port 22 are not touched |
| **Only a wrong password counts** | not a key the server does not know: an ssh agent that offers several keys is not punished for it |
| **A sign-in forgets the failures** | and a connection that makes the third wrong try is cut at once, whatever the client says it may try |
| **Who is locked** | `bash ~/SDCoreSolo/tools/solo-ssh.sh locked ~/SDCoreSolo` lists the addresses; `unlock ~/SDCoreSolo ADDRESS` (or no address, for all) lets them back in. The state is `~/SDCoreSolo/sshd/guard.json`, mode `0600` |
| **How** | systemd starts `tools/solo-sshguard.py` for each connection, which runs the `sshd` itself and reads its log; it reads the address from the connection, never from anything the client sends. Every line is in the journal: `journalctl --user -u 'sd-solo-ssh@*'` |

**What it does not stop:** a guess spread over **many addresses** (each gets its three), and connections
already open when the lock engages: by reading the guard's code (not measured), each of them is cut at its
next wrong password, so each gets one more guess. On a computer where everyone shares one address (behind one NAT), they share the lock.
**Measured** with a fake `sshd` for the rules and with the real `sshd` end to end (the first two wrong
passwords get the ordinary refusal, the fourth connection is refused, `locked` names the address and
`unlock` lets it in again). **Not measured:** through the installed Solo's own systemd unit.

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
  a key that is not in the file is refused; forwarding is refused, by the configuration and by `restrict`
  each on its own, and a control with both removed lets it through; `StrictModes` refuses a key file at `0666`;
- **the Linux password**: a private `sshd` run as the user, not root, with the same settings, checked a **wrong** password
  through PAM's own helper and refused it (also through the real systemd unit), and **the owner's correct password was
  accepted** and the forced command ran as him; the key exchange and cipher were agreed before the password was sent;
- the systemd units start one `sshd` per connection and leave no failed units;
- **a real `sd-solo` session over port 4251** on the installed Solo with a key: the password was asked, `WHO` answered
  `sduser`, a stranger's key was refused, and port 4251 presented Solo's own host key while port 22 presented a
  different one;
- an upgrade from the previous release: the Solo key line moved to the new file with its backup, the units
  came up, and removing the old block was done by hand.

**Also measured, the same evening:** on the installed Solo, `ssh -p 4251 -o PubkeyAuthentication=no` with the
owner's Linux password was accepted (`Accepted password`) through Solo's own systemd unit and landed in SD, which then
asked for the SD account password.

**Not measured:** the lockout through the installed Solo's own systemd unit; the install default (ssh on, `local`) on a fresh install; reaching port 4251 from another computer (`open`) and the firewall rule; a sign-in after a restart with linger off; a computer without
the `ufw` firewall; a distribution other than Ubuntu, and a PAM setup other than Ubuntu's (the check uses the
`sshd` PAM service; it logs two harmless refusals for a process that is not root, and a stricter stack may
refuse the session).
