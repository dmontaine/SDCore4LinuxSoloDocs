Title: The AI Helper
Subtitle: One provider and one model, a key that SD never shows or keeps, and one gate - and what a helper that runs as you can and cannot promise.

SD can ask an AI model a question for you (the `ai` verb, see *SD TCL - Asking the AI* in the User set). This page is for the administrator (`admin` first): it says what is set up, what leaves the computer, how to choose the provider and give it a model and a key, how to turn the gate on and off, and **what is weaker here than in SD Core for Linux**.

> **Nothing is on as delivered.** There is no provider, no key, and the gate is off (each named provider comes with a default model, so the model is the one thing you need not supply). The installer asks nothing about the AI. Until you have done the other three, the `ai` verb says so in words and nothing is sent anywhere. Turning one of them on is your act, and each is one command (or one key on the AI choice page) that can be undone.

## What leaves the computer

When you type `ai some text`, SD shows you the provider, the model and the size of all that will be sent, then your text, and asks whether to send it. Only if you answer `y` or `yes` does anything go out. What goes out is your standing text (below), that text, one fixed sentence that asks for a short plain answer, and a limit on the length of the answer. It is sent over **https**, with the certificate checked, to the address of the provider you chose, and to nowhere else. The key goes only to that address. No proxy is used: a computer that reaches the internet only through an HTTP proxy cannot use the AI.

SD adds nothing of the account, the session or the data. You can of course type or paste whatever you like into the question, which is why the preview is not optional.

**The standing text.** Every request, from the `ai` verb and from every AI line, begins with a few lines of yours: what SD is, how its commands are written, which gates there are, and where SD's help is. SD ships a default; you can replace it (see [The standing text](24a-setting-up-the-ai-helper.html#the-standing-text)). It is not in the preview and it does not change the question. The preview's size counts it.

**Passages of the documentation.** The AI line (the home screen, the pages of the toolkit, the IDE, SED, the collection editor and the debugger; not the `ai` verb) also adds up to four sections of SD's own documentation that match the request, at most 2,400 bytes each and 6,000 in all. Like the standing text and SD's fixed instruction they are not shown in the preview; its size counts them. The installer puts the markdown of the User document in `docs/user` in your Solo directory (755, files 644) and the markdown of this Getting Started document in `docs/admin` (700, files 600). **`docs/admin` is read only for a session that has typed `admin`**: the routine asks SD for the administrator before it looks there. The directory is replaced whole at each install and is not a place to put your own text. If the installer could not reach the documentation it prints a note, installs nothing and the AI line works without passages. This is documentation, not source: the rule below that the AI is never given SD's own code is unchanged.

## What the AI is never given, and what is not taken to it

**The AI in SD helps you use SD and write your own programs, dictionaries and files. It is never given SD's own code or any other system information, whichever provider you chose, and not even after `admin` can a question about changing SD be taken to it from inside SD.** The source is open, so if you want help changing SD you can ask an AI outside SD; inside SD that help is not offered. Two things enforce this, and they are not equally strong.

1. **Where the text comes from (enforced).** Every program that puts the text of a record or file into a request (the IDE, SED, the collection editor and the debugger) tells the AI routine where the text comes from. Before anything is shown or sent, the routine refuses text that is part of SD itself, and text whose origin it cannot tell. SD's own files are everything in your Solo directory **except the account's own directory** (`user_accounts/sduser`): the programs, headers, messages, help text, dictionaries and files of the system account, the account register, the audit trail and the error log, and the AI helper's directory `ai/`; and `/run/sd`, `/etc/sd.conf` and any directory called `gplsrc`, `gplbld` or `gplobj`. It is decided by where a file is, with links and `..` followed, never by what the text says. The headers in `syscom` are in the Solo directory, so they are not sent either; your own programs are, `$include` lines and all. If you want the AI to see one of those headers for a program of yours, copy it into the account: a copy there is the account's own file, and nothing is opened up.
2. **What is typed (a screen).** A request that names the system's code or build is refused in words before it is shown, and nothing is sent: `gpl.bp`, `gplsrc`, `gplbld`, `$internal`, a kernel file such as `op_sort.c`, a kernel key such as `k$audit`, the compiler's own programs, the makefile, SD's source or kernel code. **This is a screen, not a wall.** A question that names none of those is not caught, and a question that does reach an AI may be answered from whatever that AI learned of the open source. SD's gate is here, in SD.

Nothing the AI returns is run, and the output of a command is never given back to it. A refusal writes one line to the audit trail (`ai refused why=request ...` or `ai refused why=origin ...`) and never the text. What you type or paste yourself is yours, and is in the preview.

## What is weaker than in SD Core for Linux, said plainly

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
2. **A model** for that provider. Each of the four named providers comes with a default model, shown on the page and used until you choose another (in the setup dialog, or `set.ai model PROVIDER NAME`; a dash goes back to the default). Model names change, so check the one you will pay for; the setup dialog offers the provider's own list once a key is set, and `set.ai models PROVIDER` prints it at the prompt. Model names are case-sensitive. Custom has no default.
3. **A key** for that provider, from the provider's own account. It is typed or pasted in the setup dialog (shown as `*`) or at the helper's own prompt, and kept for the helper only.
4. **The gate**: `set.ai gate on`. It is a setting of the helper (the line `gate=yes` in `ai/ai.conf`); anything else, or no line, is off.

## Continued in

[Setting Up the AI Helper](24a-setting-up-the-ai-helper.html) — the AI choice
page, the set.ai verb, the standing text, the key, a custom provider, the
audit trail and troubleshooting
