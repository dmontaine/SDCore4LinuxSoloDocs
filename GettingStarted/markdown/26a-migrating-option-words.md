Title: Migrating to SD: option words
Subtitle: Every word of the option command, what it changes, and where else it is described.

This page continues [Migrating to SD](26-migrating-to-sd.html). The words are the
seven that `option` lists. The others that earlier releases had, whose off behavior is
now the only one, are gone; the words that existed only to imitate other products are
in the last section of [Migrating to SD](26-migrating-to-sd.html). **All are off in a
new session.** `option` *name* turns one on, `option` *name* `off` turns it off,
`option` *name* `display` shows it. The last column names the page of the User set that
also describes the word; **here** means this page is where it is described.

## Queries: list, sort, select and the rest of the query processor

| word | what it does | also described in |
|---|---|---|
| `$query.debug` | lets a query take the `debugging` keyword; a development aid, shown by `option` only while it is on. Do not use it in an application | here |

## Errors and diagnostics

| word | what it does | also described in |
|---|---|---|
| `dump.on.error` | a fatal run-time error writes a process dump file (`sddump.`*n*) in the dump directory | here |
| `show.stack.on.error` | a fatal run-time error also prints the call stack: program, line and object address | here |
| `no.user.aborts` | the A (abort) answer is removed from the prompts that offer it: pause, the end-of-page prompt, the break key menu and the query processor's page prompt | here |
| `spool.command` | shows the operating system command that prints a job, just before it runs; a diagnostic aid | *SD Basic - Program Structure* |

## Files

| word | what it does | also described in |
|---|---|---|
| `dir.dtm` | writing a record to a directory file moves the directory's date and time modified, even when the record already existed | *SD Basic - Program Structure* |
| `git` | a directory file keeps `.gitignore` and `.gitattributes` under their own names on disk; without it the leading dot becomes `%D` | *SD Basic - Program Structure* |
