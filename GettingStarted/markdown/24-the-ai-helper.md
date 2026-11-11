Title: The AI Helper
Subtitle: One provider and one model, a key that SD never shows or keeps, and one gate - and what a helper that runs as you can and cannot promise.

SD can ask an AI model a question for you (the `ai` verb, see [SD TCL - Asking the AI](29b-sd-tcl-asking-the-ai.html)). This page is for the administrator (`admin` first): it says what is set up, what leaves the machine, how to choose the provider and give it a model and a key, how to turn the gate on and off, and **what is weaker here than in SD Core**.

> **Nothing is on as delivered.** There is no provider, no key, and the gate is off (each named provider comes with a default model, so the model is the one thing you need not supply). Until you have done the other three, the `ai` verb says so in words and nothing is sent anywhere. Turning one of them on is your act, and each is one command (or one key on the AI choice page) that can be undone.

## What leaves the machine

When you type `ai some text`, SD shows you the text, the provider and the model, and asks whether to send it. Only if you answer `y` or `yes` does anything go out. What goes out is that text, one fixed sentence that asks for a short plain answer, and a limit on the length of the answer. It is sent over **https**, with the certificate checked, to the address of the provider you chose, and to nowhere else. The key goes only to that address.

SD adds nothing of the account, the session or the data. You can of course type or paste whatever you like into the question, which is why the preview is not optional.

## What is weaker than in SD Core, said plainly

SD Core runs the helper as a separate operating system user, `sdai`, in files only root can change, and asks the operating system who is calling. **Solo makes no operating system user and no root-owned file of its own** (that is what makes it a Solo), so none of that is possible. The helper runs **as you**, from your own systemd user manager, with its settings and keys in a private directory of your Solo directory (`ai/`, mode 0700; every file in it mode 0600).

What follows from that, and what does not:

- **SD itself keeps its side.** `set.ai` and `tool.ai` are the administrator's. Nothing in SD ever shows a key, and none is ever written to the audit trail; `set.ai` reads back only whether it is *set*. A key typed at the helper's own prompt (`set.ai key PROVIDER`) does not pass through SD at all; one typed in the page's setup dialog does pass through your SD session and a hand-over file on its way (see "The key", below).
- **Another program of yours is not stopped.** Any program you run - a shell, a script, a program of the account that has been given `os.execute` - can talk to the helper's socket, change its settings, turn the gate on, or read the key file, because they are all you. The gate is the one switch SD keeps; it is not a wall against a program that is not SD. If that matters, do not give the account `sh` or `os` access (see [Operating system access](06b-operating-system-access.html)) and do not run untrusted programs as this user.
- **Someone else's account is stopped.** The socket is mode 0600 in your runtime directory, and the helper refuses any connection that is not from your own user, so another operating system user on the machine cannot use it.

If you need the key out of reach of your own programs, use SD Core, where the helper is a different user.

## The pieces

| Piece | What it is |
|---|---|
| `tools/sd-ai` (in your Solo directory) | The helper. A small Python program, started by your systemd user manager for each request, as you. It is the only part of SD that connects outward and the only part that reads a key. |
| `$XDG_RUNTIME_DIR/sd-solo-ai.sock` | The socket the helper listens on (`sd-solo-ai.socket`, mode 0600, no network address). |
| `ai/` (in your Solo directory) | The helper's own directory: the settings (`ai.conf`, which holds the gate line), one key file per provider and the counters of the limits. The helper makes it on the first change. |
| `tools/ai-providers` | The connection details that come with SD for Anthropic, OpenAI, Google and DeepSeek, and each one's default model. |
| `tools/sd-ai-ctl` | The client for the helper. It is what asks for a key at the terminal, without echo, or reads it from the page's hand-over file. |
| `$aikey/` (in your Solo directory) | The setup dialog's hand-over directory, mode 0700. A key typed in the dialog is in a file here only until `set.ai key PROVIDER handover` has given it to the helper; the file is then removed. |

The installer writes the socket and the service unit (and an upgrade writes them for an install from before this release). They do nothing until the AI is set up.

## The four things that turn it on

1. **A provider.** One: Anthropic, OpenAI, Google, DeepSeek or Custom.
2. **A model** for that provider. Each of the four named providers comes with a default model, shown on the page and used until you choose another (in the setup dialog, or `set.ai model PROVIDER NAME`; a dash goes back to the default). The defaults were read from the providers' own pages and have not been tried against a real key; model names change, and the setup dialog offers the provider's own list once a key is set. Custom has no default.
3. **A key** for that provider, from the provider's own account. It is typed or pasted in the setup dialog (shown as `*`) or at the helper's own prompt, and kept for the helper only.
4. **The gate**: `set.ai gate on`. It is a setting of the helper (the line `gate=yes` in `ai/ai.conf`); anything else, or no line, is off.

## The AI choice page: `tool.ai`

```
tool.ai
```

An administrator-only full-screen page (140 columns by 43 lines at least). It has two boxes.

**Provider** lists Anthropic, OpenAI, Google and DeepSeek, then Custom, one to a row. A check box marks the provider in use; none marked means the AI is off. The four named providers come with their address and a default model filled in (the Model column says `(default)` beside one you have not changed); you supply the key, and S (setup) sets the key and the model together. **Custom** is the fifth choice, where you fill in everything yourself: a name, an `https://` address, the request layout (`anthropic` or `openai`), the authentication (`bearer` or `x-api-key`), a version header (or none) and the name of the field that limits the length of an answer. Below the list, the row under the cursor is shown in full, with the file of certificate authorities and the limits.

**May this account use the AI?** is the one row for your account, with the state of the gate.

| Key | In the Provider box | In the Account box |
|---|---|---|
| Enter or space | check the provider under the cursor (you type its name to confirm), or uncheck the one in use | turn the gate on (you type `ai`) or off |
| S | the setup dialog for the provider under the cursor: the key and the model, the model chosen from the provider's own list (below) | |
| X | remove the key | |
| C | fill in Custom | |
| D | remove the Custom connection (the helper refuses while it is in use) | |
| A | the file of certificate authorities (you type `ca` to confirm) | |
| L | change a limit | |
| F6, or Left and Right | change box | change box |
| F1, `?` or a click on `[?]` | explain the box | explain the box |
| F5 | read the machine again | read the machine again |
| Ctrl-Q | leave | leave |

**The page keeps nothing itself and never shows a key.** Every change is a `set.ai` sentence, shown with its effect in a box that you answer Run or Cancel (Enter chooses Cancel). The page then runs exactly that sentence. The exception is the setup dialog, whose Save is the confirmation.

**The setup dialog (S).** It looks the same every time, whatever is already set:

- **Key**, a password field. What you type or paste is shown as `*`. When a key is already set the field shows a row of twelve stars, which keeps it; replace them to give a new key.
- **Model**, a list of the provider's own model names, with the one in use chosen. With no key yet the list says the key comes first: enter the key and press Tab, and the list is filled from the provider with that key. A model in use that the list does not hold is pointed out, and when it differs from a listed one only in capitals the listed one is chosen for you.
- **Other name**, only for a model the list does not hold, such as on a Custom server that gives no list. A name typed here is still checked against the list when there is one.
- **Save** sets what you chose; **Cancel** or Esc changes nothing more (a key already handed over by Tab stays set, and the page says so).

**What the dialog does with the key.** When you leave the Key field with a new key (Tab, or Enter, which then shows the list rather than saving), the page writes it to a file in a directory only you can enter (`$aikey` in your Solo directory, mode 0700), runs `set.ai key PROVIDER handover`, which gives that file to the helper's client as its input, and removes the file whether or not that worked. The key is never on a command line and never in the audit trail. It does pass through your SD session's memory while the dialog is open, and through that file for a moment. If you would rather it never touched SD, use `set.ai key PROVIDER` at the prompt, which hands the terminal to the helper's own prompt.

Opening the page changes nothing and writes one audit line, `tool start app=tool.ai`.

## The verb: `set.ai`

```
set.ai
set.ai gate on | off
set.ai provider anthropic | openai | google | deepseek | custom | none
set.ai model PROVIDER NAME | -
set.ai models PROVIDER
set.ai key PROVIDER
set.ai key PROVIDER handover
set.ai key PROVIDER clear
set.ai custom NAME URL FORMAT AUTH VERSION MAXPARAM
set.ai custom clear
set.ai ca PATH | -
set.ai limit NAME NUMBER
```

For the administrator only. Alone it says what is set: the provider in use, the gate, each provider's address, model and whether its key is set, the certificate file and the limits. It never shows a key. `provider none` turns the AI off. `gate off` stops every request while leaving the provider, the model and the key as they are. `models` asks the provider, with its key, for the names of its models and prints them, one to a line. `model` keeps the name you give, and says so when the provider does not list it (`deepseek does not list a model named Deepseek-V4-Pro, so requests may be refused`, then `It lists deepseek-v4-pro`); a dash goes back to the provider's default. `key` hands the terminal to the helper's prompt; nothing typed there reaches SD. `key PROVIDER handover` is what the page runs: it reads the page's hand-over file and removes it. When a provider refuses a request, its own explanation is shown after the HTTP status (`the provider refused the request (HTTP 400): Model Not Exist`), cut to one line, with any copy of the key taken out; a refused key (401 or 403) shows only that the key was not accepted. A word may hold letters, digits and `. _ : / @ + , -` and nothing else. The helper checks every result whole before it writes it: one provider selected means its details are complete, a Custom address is `https://` with no query string, a limit is a whole number in range.

The limits, and what they are as delivered:

| Limit | Delivered | Meaning |
|---|---|---|
| `requests-hour` | 20 | requests you may make in an hour |
| `tokens-day` | 100000 | tokens (in plus out) you may use in a day |
| `timeout` | 30 | seconds to wait for the provider (at most 300) |
| `bytes-request` | 65536 | the most that may be sent in one request |
| `bytes-reply` | 65536 | the most that may come back |
| `max-tokens` | 1024 | the longest answer asked for |

## The key

The key is typed at the helper's own prompt (`set.ai key anthropic`) or in the setup dialog on the page. At the prompt it goes from your terminal straight to the helper and SD never sees it. From the dialog it passes through your SD session's memory and a hand-over file in a directory only you can enter, which is removed at once (see above). Either way it is written to `ai/keys/anthropic`, mode 0600; SD does not keep it, print it or put it in the audit trail, and the page and the verb read back only whether it is **set**. `set.ai key anthropic clear` removes it. A key is never an argument of a command, so it is in no process list and no history. Deleting your Solo directory (`deletesdsolo.sh`) deletes the keys with it.

If a key is lost or shown to someone, clear it and ask the provider for a new one: SD cannot tell you who has seen it.

## Custom: say it plainly

Custom lets you point the helper at any `https://` address. The text you send, and the key, then go to that host. That is your deliberate act and it is the widest thing on this page. What stays fixed: https only (a local model server on `http://` is not allowed), the certificate and the host name are always checked (there is no switch to turn that off), the helper never follows a redirect to another host, and the key is sent only to the address you gave. A private certificate authority is the file you name with `set.ai ca`; **it replaces the system store for the helper**, so whoever holds one of its keys can read what you send and the key. Use a file you trust, and `set.ai ca -` to go back to the system store.

If you want only the four named providers, leave Custom unfilled.

## The audit trail

All in lower case, none with a key or the text:

```
ai request account=sduser program=ai bytes.out=154
set.ai gate=on
set.ai provider=anthropic
set.ai model provider=anthropic model=claude-...
set.ai key set provider=anthropic
set.ai key set provider=anthropic from=page
set.ai key cleared provider=anthropic
set.ai custom name=MyLLM host=api.example.com
set.ai limit name=tokens-day value=50000
tool start app=tool.ai
```

The `ai request` line is written by the SD kernel **before** the connection is made, with the account and the program that called, so a crash or a refusal still leaves the record. The Custom line names the host and never the whole address.

## Troubleshooting

| What you see | What it means |
|---|---|
| `The AI helper could not be reached: it is not running, or it is not installed` | the socket is not there: `systemctl --user status sd-solo-ai.socket`; an install from before this release gets it from the upgrade, or `bash tools/solo-service.sh ai <your Solo directory>` |
| `The AI is off: the administrator has not chosen a provider` | nothing is checked on the page (`set.ai provider ...`) |
| `The AI is not turned on: ... set.ai gate on` | the gate is off |
| `The AI is not set up for anthropic: the administrator has not set its key` (or, for Custom, `model`) | the provider is chosen but its key (or Custom's model) is missing |
| `The AI did not answer: refused (the provider refused the request (HTTP 400): ...)` | the provider's own words follow the status: often a model it does not have; choose one from its list with S |
| `The AI did not answer: limit (...)` | one of the limits above is used up; wait, or raise it |
| `The AI did not answer: net (...)` or `timeout` | the provider could not be reached; check the address, the certificate and the clock |

## What has not been shown

The helper, the verb and the page are tested against a stand-in provider on this machine, in a private copy of SD. They have **not been run against a real provider**: the request layouts follow each provider's published documentation, the one for OpenAI (`max_completion_tokens`) and Google's (its OpenAI-compatible address) have not been checked against a real key, and neither has any default model nor any provider's list of models. A real install and the real systemd user socket are a separate check.

## See also

[Administrator commands](06-administrator-commands.html) ·
[Operating system access](06b-operating-system-access.html) ·
[Giving an AI agent an installation of its own](23-giving-an-ai-agent-its-own-installation.html).
