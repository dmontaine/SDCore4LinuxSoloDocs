Title: Scheduled jobs
Subtitle: Running an SD command on a timer with cron or a systemd timer.

**A scheduled job is `sd-solo <command>`, run by cron or a systemd timer as you.**
There is no permit list to fill in and no administrator rights to give it: a
command on the `sd-solo` command line signs in with the copy of the account password
kept for you, and runs. Unlike the multiuser SD Core for Linux there is no
`batch.jobs` file, and the command may carry arguments — `sd list customers
with city = "Leeds"` is a job.

## Setting one up

**1. Write the work as a paragraph** — a `PA` record in the VOC, with the
commands on the lines after the type. Editing the VOC needs `ADMIN` first:

```
:admin
:ed voc my.report
```

(A job that is one command needs no paragraph.)

**2. Check it runs when you type it**, at the `:` prompt, before putting it on a
timer. Then check it from a terminal, the way the timer will run it:

```
sd-solo my.report
```

**3. Create the timer.** Either of these, as your own user:

```
# crontab -e
0 6 * * *  /home/you/.local/bin/sd-solo my.report
```

or a systemd **user** timer that runs `~/.local/bin/sd-solo my.report` from a
`.service` unit. Use the full path to `sd-solo`: cron's PATH does not include
`~/.local/bin`.

**Run it as your own Linux user**, the one SD is installed for. Another user has
no copy of the password and no `~/SDCoreSolo` of its own. **Do not run it with
`sudo`**: SD refuses to run as root.

## Signed in or not

**The SD daemon has to be running when the job fires.** SD is your systemd user
service, and your systemd user manager stops when your last session ends —
**unless linger is on** (see [Running SD](03-running-sd.html)). A cron job that
fires while you are signed out, on a computer without linger, finds no daemon,
and `sd-solo` says so and does nothing. If jobs must run while you are signed out,
enable linger: `sudo loginctl enable-linger $USER`.

## Supplying the password on the input

**If the kept copy is missing or no longer matches** — you changed the password
without `SET.PASSWORD` — a command whose input is piped takes the first line of
that input as the account password, once:

```
cat ~/.sd-job-password | sd-solo my.report
```

**That file holds the password in clear text.** Keep it mode 0600, or prefer the
kept copy, which is already in a private file: `~/SDCoreSolo/$cred/$stored`.

## When it does not run

| | |
|---|---|
| *A command given on the sd command line needs the account password on its input* | the kept copy did not work and the command was run at a terminal, with nothing piped in |
| *Wrong password* | the piped password was wrong |
| *SD has not been started* | the daemon is not running: no linger, or SD was stopped |
| the job reports success and nothing happened | look in `~/SDCoreSolo/audit`: a sign-in with the kept copy is recorded as `login password account=sduser via=stored` |

**A job's output goes where cron sends it** — mail, or nowhere — unless the
paragraph sends it somewhere itself: a file, or a printer.
