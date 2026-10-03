Title: The scripts SD runs itself
Subtitle: What the installer calls in turn, and what does not exist on Solo.

This page continues [The installed scripts](17-the-installed-scripts.html).

**SD itself runs no script.** The daemon, the verbs and the API never call
`sudo` or a shell script of their own. What runs scripts is the installer, in
this order:

| Step | What runs | What it does |
|---|---|---|
| 1 | `installsdsolo.sh` | asks the questions, or reads the options and the control file; refuses before changing anything if it should |
| 2 | `git clone`, `make` | downloads the source to `~/.sdsolotmp` and builds it there |
| 3 | `gplbld/solo-stage.sh` | lays the tree out in your home directory and runs the bootstrap: `sd-solo -i`, `SECOND.COMPILE`, the dictionaries, `THIRD.COMPILE`, then makes the account, sets the passwords, sets the deny list, and runs `SYNC.GLOBAL.CATALOG`. On an upgrade, `--upgrade`, which first makes the safety copy — see [Upgrading and uninstalling](01a-upgrading-and-uninstalling.html) |
| 4 | `tools/solo-service.sh install` | the systemd user units |
| 5 | `tools/solo-ssh.sh key-add`, `match --apply` | only if you asked |
| 6 | the self-check | a session as `sduser`, and that `sd-solo -internal` is closed |

**Every step that talks to SD does it through `sd-solo -internal`, one session at a
time.** Each session needs a marker file — `$internal` in the tree — that the
installer writes immediately before starting it and that SD deletes on admission.
A marker older than ten minutes is refused, and consumed. **That is why the door
is closed when the installer has finished:** there is no marker, and nothing but
an installer step writes one. The audit trail records each admission, with the
writer.

## What Windows automates that Solo does not

| Windows Solo | Here |
|---|---|
| a scheduled task registered in an elevated step | a systemd user unit, and no elevated step at all |
| `solo-machine.ps1`, the one administrator consent prompt | nothing: `sudo` is used for packages, a firewall rule and linger, and each is optional or checked first |
| PowerShell scripts for the firewall and ssh rules | `ufw` if it is active, and otherwise a message; `solo-ssh.sh` and `solo-service.sh ssh` for Solo's own ssh listener |
| `install-summary.log` | the installer's own output, and `journalctl --user -u sd-solo.service` for the service |

## What the multiuser product has that Solo does not

`sd-elevate`, `ssh-forcecommand` and `sd-reconcile-accounts` — the `sudo` helper,
the `sshd_config` block for the group, and the account-register checker — are
gone with the accounts, the groups and root. See
[Not in SD Core](14-not-in-sd-core.html).

## See also

[Installing](01-installation.html) covers what the installer asks and puts where.
[ssh access](08-ssh-access.html) and [API access](09-api-access.html) cover the
two scripts that change either setting.
