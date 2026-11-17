Title: Scheduled jobs
Subtitle: Running an SD command on a timer: SD's own jobs (set.job), or cron and a timer of your own.

There are two ways: **SD's own scheduled jobs**, which SD lists, starts and
stops for you, and **a cron line or a systemd timer you write yourself**. Both
run `sd-solo <command>` as you, signed in with the kept copy of your account
password.

## SD's own scheduled jobs

**Off as delivered.** The administrator turns them on once:

```
:admin
:set.jobs.access on
```

(or on the gates page, `tool.gates`, the row *Scheduled jobs*). While they are
off, no job can be made or started, and every job is refused at its next run;
`set.jobs.access off` stops them all that way. Their timers stay until you
delete the jobs.

**Make a job** with its name, its schedule and the command, as you would type
the command at the prompt:

```
:set.job add nightly daily 02:30 my.report
Password:
Job nightly is scheduled: daily 02:30
```

**It asks for your account password, every time** you make or change a job,
even in a session that has signed in; it cannot be done from a script or a
one-shot command. So a job runs only a command that you, knowing the password,
gave it.

| Command | Does |
|---|---|
| `set.job` | lists your jobs, each with its schedule, its command, when it runs next and how its last run ended |
| `set.job add` *name* *schedule* *command* | makes a job (refused if one of that name exists) |
| `set.job change` *name* *schedule* *command* | changes one |
| `set.job delete` *name* | stops and removes one |
| `set.job run` *name* | starts one now, as its timer would |
| `tool.jobs` | the same on a full-screen page: N, C, D and R; its form has a masked field for the password |

*name* is 1 to 32 characters: a lower-case letter first, then lower-case
letters, digits and dashes. *schedule* is one of `every 5 minutes` (or 10, 15,
20, 30), `hourly`, `daily hh:mm`, `weekly mon hh:mm` (mon to sun), `monthly 1
hh:mm` (day 1 to 28), in the computer's time. A run missed while the computer
was off is not made up.

**What a job is.** Two systemd user units of yours,
`~/.config/systemd/user/sd-solo-job-<name>.timer` and `.service`. The service
runs `sd-solo "run.job <name>"`, which signs in as every one-shot command does
and runs the command the job was given; SD keeps the command, not the unit. A
job's output goes to your journal (`journalctl --user -u sd-solo-job-<name>`),
and the audit trail has a `run.job start` and a `run.job end` line for every
run. Change a job with `set.job`, not by editing its units.

**This is not a wall against you.** The definitions (`~/SDCoreSolo/$jobs`) and
the units are yours, as the programs are: a job adds no way in that a one-shot
`sd-solo` command does not already have. The password is asked so that a job
is set deliberately, by its owner.

**Uninstalling** removes the timers. When you keep your data, the job
definitions are kept with it, and a new install that reloads the data gives
the jobs their timers back. The register's switch is not kept: turn jobs on
again with `set.jobs.access on`.

## Cron or a timer of your own

**A scheduled job is `sd-solo <command>`, run by cron or a systemd timer as you.**
There is no permit list to fill in and no administrator rights to give it: a
command on the `sd-solo` command line signs in with the copy of the account password
kept for you, and runs. Unlike the multiuser SD Core for Linux there is no
`batch.jobs` file, and the command may carry arguments — `sd-solo list customers
with city = "Leeds"` is a job.

## Setting one up

**1. Write the work as a paragraph** — a `PA` record in the VOC, with the
commands on the lines after the type. Editing the VOC needs `admin` first:

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
without `set.password` — a command whose input is piped takes the first line of
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
