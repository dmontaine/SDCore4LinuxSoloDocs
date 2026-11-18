Title: Encrypting the data SD keeps on disk
Subtitle: Putting SD's folder on an encrypted disk, what that protects, what it does not, and how it fits a service that starts when you sign in.

This page is about the bottom layer in the table on the
[encryption page](18-encryption.html): the disk. SD itself is unchanged by it.
The encryption is done by the operating system, to the volume your SD folder
lives on. The encryption page explains the other two layers, which are the
application's, and the functions and key ring they use.

SD folds case, so a command may be typed in either case. Commands are shown
here in lower case.

## The short version

**Encrypt the whole disk, or at least your home directory.** Solo keeps
everything in one folder, `~/SDCoreSolo`, under your own home, so the simple and
complete answer is the one the operating system already offers: full-disk
encryption chosen when the distribution is installed, or an encrypted home
directory. A stolen or discarded computer, or a copy of the disk, then holds
ciphertext, and nothing about SD has to change.

**It protects nothing once the volume is open.** While you are signed in, or SD
is running on an unlocked volume, anyone who can sign in to Linux as you, or as
`root`, reads the data as plaintext. The same is true of anyone who mounts the
volume.

If you cannot encrypt the disk or the home directory, encrypt a volume that
holds only `~/SDCoreSolo` (the last method below). Whichever you use, set
`SORTWORK` and encrypt or switch off swap, as described next.

## What needs protecting

Everything SD keeps is under `~/SDCoreSolo`, which the installer makes private
to you (mode 0700).

| Path | What is there |
|---|---|
| `~/SDCoreSolo/user_accounts/sduser` | the account, your data |
| `~/SDCoreSolo/$cred/` | the credential store: the verifiers for the passwords, and the kept copy of the account password, **in clear** |
| `~/SDCoreSolo/sd.conf` | the configuration |
| `~/SDCoreSolo/audit` and `errlog` | the audit trail and the error log |
| `~/SDCoreSolo/sd-tls` | the API's TLS key, when you chose an API |
| `~/SDCoreSolo/sshd/` | Solo's own ssh host key and your key file, when ssh is on |
| `~/SDCoreSolo/ai` | the AI settings and keys, when you use the helper |
| `~/SDCoreSolo/netlogins` | stored SDNet logins |

**The credential store matters most.** The kept copy of the account password is
a plain file, as the [Security](12-security.html) page says, so a copy of the
folder is a copy of the password. That alone is a reason not to leave a laptop
unencrypted.

**Backups go where `BACKUPDIR` points.** It is not set until you run
`set.backup.directory`, which saves the directory in `sd.conf`
([Backup and restore](06c-backup-and-restore.html)). `backup.account` writes
plain zip files there. A backup is protected only while that directory is on
an encrypted volume, and not at all once the file is copied away.

**The sort work area is a place of its own.** A sort that does not fit in memory
writes its work files to `SORTWORK`, and when that is not set, to `TEMPDIR`,
and when that is not set, to the system's default place for temporary files
([The parameters](16c-the-parameters.html)). Those files can hold the keys being
sorted, in plain text. On a volume that is not wholly encrypted, put them on the
encrypted one:

```
# in ~/SDCoreSolo/sd.conf
SORTWORK=/home/you/SDCoreSolo/sortwork
```

Create the directory first, as yourself (`mkdir -m 700 ~/SDCoreSolo/sortwork`),
and replace `/home/you` with your own home. If the directory does not exist
when SD starts, SD falls back to `TEMPDIR` without saying so. `TEMPDIR` itself,
for SD's other temporary files, is not covered by this page. A change to either
parameter needs a restart ([Configuration](16-configuration.html)).

**Swap.** SD's shared memory can be paged out to swap. Encrypt the swap or turn
it off; with full-disk encryption it is usually covered already, and the
distribution's installer is the place to check.

## Method one: full-disk encryption

Choose encryption when you install Debian or Ubuntu: the installer offers an
encrypted disk (LUKS) and asks for a passphrase that is typed at every boot.
Nothing else is needed for SD. The folder is on the disk, the disk is unlocked
before anyone signs in, and your user service starts as it always does.

Do this before the data exists. Turning an existing, populated disk into an
encrypted one is a reinstall in practice, and the old blocks of the unencrypted
disk are exposed until they are overwritten.

## Method two: an encrypted home directory

If the disk is not encrypted but your home directory can be, put `~/SDCoreSolo`
inside it. Use what your distribution provides for a home directory encrypted
with your sign-in password, and follow its own instructions: SD has no part in
it, and this page does not repeat them.

**What to check for Solo.** A home directory that is unlocked when you sign in
is not there before you sign in. [Running SD](03-running-sd.html) describes
what that means: your systemd user manager starts when you first sign in and
stops when your last session ends, **unless linger is on**, and with linger SD
starts at boot and runs while you are signed out. On a machine whose home
opens at sign-in, linger would start SD before the folder can be read. So:

- **leave linger off** (`--no-linger` at install, or
  `sudo loginctl disable-linger $USER` afterward), so that SD starts when you
  sign in and the home is open; and
- remember what that gives up: a scheduled job or an API or ssh client that
  arrives while you are signed out finds nothing running
  ([Scheduled jobs](04-scheduled-jobs.html), [ssh access](08-ssh-access.html)).

If you need SD reachable while nobody is signed in, use method one, where the
disk is open from boot. Try a full restart, and a sign-in, on your own machine
before you rely on either arrangement.

## Method three: a volume for the folder alone

Use this when the disk and the home directory cannot be encrypted. You make an
encrypted volume (LUKS2), move `~/SDCoreSolo` onto it and mount it where the
folder was.

### Install the tools

```bash
sudo apt-get install -y cryptsetup
```

`cryptsetup` is in `/usr/sbin`, which is not on an ordinary user's path, so run
it as `sudo /usr/sbin/cryptsetup`.

### Create the container

```bash
truncate -s 2G ~/sdsolo-volume.img
sudo /usr/sbin/cryptsetup luksFormat --type luks2 ~/sdsolo-volume.img
sudo /usr/sbin/cryptsetup open ~/sdsolo-volume.img sdsolo
sudo mkfs.ext4 -q -L sdsolo /dev/mapper/sdsolo
sudo mkdir -p /mnt/sdsolo
sudo mount /dev/mapper/sdsolo /mnt/sdsolo
sudo chown "$USER": /mnt/sdsolo
chmod 700 /mnt/sdsolo
```

`luksFormat` asks for the passphrase, and `open` asks for it again; typed
at the prompt, it is in no script and no history file. The size is only an
example: size the image to your data and keep it on a disk you choose. The
image is an ordinary file in your home, so keep it out of anything you back up
unencrypted.

### Move the data, in order

Stop SD first. Do not copy the folder while SD is running.

```bash
systemctl --user stop sd-solo.service
rsync -aHAX ~/SDCoreSolo/ /mnt/sdsolo/
```

If you use the API or ssh, stop their sockets too:
`systemctl --user stop sd-solo-api.socket sd-solo-ssh.socket`.
`rsync -aHAX` keeps permissions, hard links, ACLs and extended attributes, so
the folder comes across private to you as it was.

**The old plaintext is still there.** Once the copy is checked, remove the old
folder and put the volume in its place:

```bash
rm -r ~/SDCoreSolo
mkdir -m 700 ~/SDCoreSolo
sudo mount --bind /mnt/sdsolo ~/SDCoreSolo
systemctl --user start sd-solo.service
```

Deleting files does not guarantee that their blocks are erased, on ext4 or on
an SSD. The cleanest order is to encrypt before any data enters the system, which
means an installation into a volume that is already mounted. Otherwise treat
the old blocks of the disk as exposed. The empty `~/SDCoreSolo` that remains
when the volume is closed holds nothing, and it is the reason to remove the
old folder rather than leave it under the mount.

### Open and close

The volume must be open and mounted **before SD starts**. Every sign-in begins
this way:

```bash
sudo /usr/sbin/cryptsetup open ~/sdsolo-volume.img sdsolo
sudo mount /dev/mapper/sdsolo /mnt/sdsolo
sudo mount --bind /mnt/sdsolo ~/SDCoreSolo
systemctl --user start sd-solo.service
```

and ends with the reverse, in order:

```bash
systemctl --user stop sd-solo.service
sudo umount ~/SDCoreSolo
sudo umount /mnt/sdsolo
sudo /usr/sbin/cryptsetup close sdsolo
```

Opening and mounting are done by hand here, and the installer does not do them.
A start while the volume is closed would find the folder empty, so leave linger
off on a machine arranged this way: with linger on, your user manager starts at
boot, before anyone can open the volume. Whether to automate the unlock at boot,
and with what (a typed passphrase, or a keyfile), is a choice for your own
machine. **A keyfile kept on the same computer as the container gives no
protection against anyone who can read that computer's disk**; it protects only
a container that is taken away without the machine.

Unlocking a passphrase keyslot needs about 1 GiB of free memory, which is the
`cryptsetup` default.

### Checking a closed volume

With the volume closed, the image should show no text of yours. Search it for a
word you know is in a record, and for the record id and the account path; each
count should be 0:

```bash
sudo grep -a -c "a text you wrote into a record" ~/sdsolo-volume.img
sudo strings -a ~/sdsolo-volume.img | grep -c "a text you wrote into a record"
```

`file` describes the image as "LUKS encrypted file, ver 2". A search is
evidence about the bytes that were searched, not a proof about every byte.

## What this does not cover

- **A running machine.** Once the volume is unlocked, everything is plain text to
  anyone signed in as you, and to `root`.
- **Copies you make.** A record exported to a file, a report printed to a file, or a
  file copied off the computer is outside the encrypted volume once it leaves it.
- **Backups.** A backup directory protects nothing unless it is on the volume or
  the backups are encrypted another way.
- **The API and ssh.** Nothing here changes how data travels over a connection;
  that is the job of TLS ([API access](09-api-access.html)) and ssh
  ([ssh access](08-ssh-access.html)).
- **The prompt at boot.** The passphrase prompt at start-up is the operating
  system's.

## What to do, in order

1. Prefer an encrypted disk, chosen when the operating system is installed.
2. Failing that, an encrypted home directory with linger off, or a volume for
   `~/SDCoreSolo` as above.
3. Set `SORTWORK` to a directory on the encrypted volume if the disk as a whole
   is not encrypted.
4. Encrypt or switch off swap, and decide where the backup directory lives.
5. Try a full restart and a sign-in before trusting the arrangement.
