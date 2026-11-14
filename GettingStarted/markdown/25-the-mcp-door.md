Title: The MCP Door
Subtitle: Letting an AI agent's MCP client use your SD account through four read-only tools, and one more if you allow it - two switches, both off, and what a door like this does and does not add.

The Model Context Protocol (MCP) is how an AI agent - Claude Code, an IDE's assistant, any program that speaks it - is given tools. SD Core Solo for Linux can offer such a client a small set of tools that use **your own account**: look at the files, read a record, list some records, and, only if you allow it, run a sentence. This page is for the administrator (`admin` first). It says what is offered, what the two switches do, what is written down about every call, and what the door is not.

> **Nothing is on as delivered.** Both switches are off. Until the administrator runs `set.mcp on`, the launcher answers an agent's first message with an error that says so, and a request written by hand is refused inside SD as well.

## What the door is

The door is a small program, `solo-mcp.py`, that is part of the installation (in its `tools` folder). **Nothing runs it until an MCP client starts it** as a child process; it has no port, no network connection and no service. It talks to the client in JSON lines on its standard input and output, and for each call it runs one `sd-solo mcp.call ...` command in your account. **Every decision is made inside SD**, by the program `mcp.call`: whether the door is open, whether a word is allowed, the work itself, and the line in the audit trail. A client that did not use `solo-mcp.py` would meet the same gate.

Print the snippet an MCP client needs with:

```
python3 ~/SDCoreSolo/tools/solo-mcp.py --config
```

It shows the command and its full path in the form MCP clients read (`mcpServers`). With Claude Code the same thing is `claude mcp add sd-solo -- python3 <that path>`.

## The two switches

| Command | What it does |
|---|---|
| `set.mcp` | Says whether the door is open and whether the write tool is. Changes nothing. |
| `set.mcp on` / `set.mcp off` | Opens or shuts the door. With it shut, no tool works. |
| `set.mcp write on` / `set.mcp write off` | Gives the agent `sd_run`, or takes it back. |

`set.mcp` is an administrator command (`admin` first), and each command changes one thing. **Both are read at every call**, so `set.mcp off` stops an agent that is already working at its next call. The change is written to the audit trail (`set.mcp on account=sduser`).

## The tools

| Tool | What it does |
|---|---|
| `sd_files` | Lists the files of the account. |
| `sd_describe` | Lists the dictionary items - the fields - of one file. |
| `sd_read` | Reads one record: a field to a line, value marks shown as `]`. |
| `sd_query` | Lists records: the fields to show, conditions (`=`, `#`, `<`, `>`, `<=`, `>=` and a plain value), one sort field, a limit of at most 200 rows. |
| `sd_run` | Runs ONE sentence as you could type it. **Only with `set.mcp write on`.** |

The first four are marked read-only for the client. **`sd_query` does not take a sentence.** The client gives names and values, and SD builds the `list` itself after checking that every field, condition field and sort field is an item of that file's dictionary; a keyword such as `lptr` is not an item, a value may hold only plain characters, and no quote reaches the query processor. `sd_run` is marked destructive, so a client can ask you before it runs.

**No tool can change a setting of SD, make or remove an account, or reach the administrator.** `sd_run` runs in the account's own session, so the account's own switches apply: while `sh` and `os` are off (as delivered), `sh` and the commands that leave the account are refused through the door as they are at your prompt. See [Operating system access](06b-operating-system-access.html).

## What is written down

Every call - refused ones too - is one line in the audit trail:

```
mcp call agent=claude-code tool=read target=customers c1 status=ok
```

`agent` is the name the client gave when it connected, cut to lower case letters, digits, dot, underscore and hyphen and 32 characters; the user is still you. `status` is `ok`, `refused` (the door or the write tool is shut, or a word is not allowed) or `error` (a file or record that is not there, a request that is not valid). For `sd_run` the target is the sentence.

## Limits

One call at a time; at most 30 seconds; an answer is cut at 64 KB (and says `[output cut]`); control characters other than a new line and a tab are turned into spaces, so a terminal escape inside a record cannot reach the agent as text; a message of more than 1 MB is refused.

## How it signs in, and what that means

A one-shot `sd-solo` command never asks for a password. It uses the account password that the installation **keeps** for exactly this (see [Scheduled jobs](04-scheduled-jobs.html)), and it proves the account password only - never the administrator's. The launcher itself holds, reads and passes no password. If no password is kept, the launcher says so and the door stays shut; `set.mcp on` tells you when that is the case.

## What the door does not add

Be clear about this before you open it. **An agent that can run commands as you can already run the same one-shot command itself** - the kept password is a file in your own installation, readable by whoever can read your files. The two switches and the read-only tools are therefore the rules of *this* door and a record for you of what came through it; they are not a wall against an agent that also has a shell.

What the door adds is structure (a tool call instead of free text), a narrow read-only default, the agent's name in every audit line, your client's own permission questions, and a switch that is off until you turn it on. Decide what you hand an agent as you would decide what you hand any program that runs as you.

## Turning it off again

`set.mcp off` shuts the door at once. Removing the MCP client's configuration stops the client starting the launcher; the launcher is a file, and nothing else in the installation depends on it.
