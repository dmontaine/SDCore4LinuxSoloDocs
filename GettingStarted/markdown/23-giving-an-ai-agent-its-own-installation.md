Title: Giving an AI Agent an Installation of Its Own
Subtitle: A recipe with no code: two shapes - a client of your installation with both OS switches off, or a computer of the agent's own - and why it cannot be a second Linux user on yours.

An AI agent that works on an SD project needs a place to work and a limit on what it can reach. Prompts, tool scopes and promises inside the agent's own software protect against mistakes and against instructions hidden in the data it reads. **They do not stop an agent that can run as the Linux user who owns the installation**: it can read the kept copy of the account password, open an interactive `sd-solo` as you and do whatever you can. What decides how far an agent can reach is **who it runs as** and **whether the two OS switches are on**.

> **The agent is never you.** Do not give it your passwords, your ssh key or a terminal that is already signed in.

## Why not a second Linux user

SD Core Solo for Linux has **one account**, `sduser`, and its two OS switches belong to that account alone ([The account and its passwords](05-account-types.html)). There is no second account to make for an agent and nothing inside the account that scopes it: whoever has the account password has all of it, less what the two switches withhold. **Only one SD can run on a computer** ([Running SD](03-running-sd.html)): the shared-memory name is machine-wide, so a second Solo for another Linux user would attach to the first one's segment. So the agent cannot have an installation of its own beside yours, and the two ssh and API ports (4251 and 4249) are fixed as well.

## The two OS switches

**The shell and OS access are two switches, `sh` and `os`, both off as delivered** ([Operating system access](06b-operating-system-access.html)). With them off, `sh`, `os.execute`, the embedded Python, the editors that run outside SD and a path outside the account are refused to a session that has not typed `admin`. **Switched on, they run with the rights of the Linux user, and then they are not a wall**: what an agent's installation can reach is what its Linux user can reach. The switches are the administrator's to turn on or off (`set.os.access`), and the installer asks about them once.

## Two shapes

The full recipe is `agent-pack/recipe/agent-account.md` in the SD Core Solo for Linux source repository, with a template for the project's `CLAUDE.md` and a Claude Code skill that carries the SD knowledge (see the page *SD and AI Agents* in the User set). The installer puts only the guide and the knowledge pack of the installed version under `~/SDCoreSolo/agent-pack`; the recipe, the template and the skill are not installed with the product: take them from the source of the version you run.

**Shape 1: the agent is a client of your installation, with both switches off.** It gets the account and nothing outside it: it cannot run a Linux command, read or write a file outside the account's directory, or change the VOC. It can read, change and delete everything the account holds, **including your production data**, so use it only where the agent may touch that data.

1. **Check the switches are off.** Type `admin`, then `set.os.access` with no argument: it says `Account sduser has SH no, OS.EXECUTE no`. If it does not, `set.os.access sh off` and `set.os.access os off` put it right, and the next sign-in sees it.
2. **Give it a way in that is not your Linux login.** Over ssh, give it a key, never your Linux password: add the agent's public key as one line of Solo's own key file, `~/SDCoreSolo/sshd/authorized_keys` ([ssh access](08-ssh-access.html)); it signs in on port 4251 with the key and types the account password at the SD prompt, and the sign-in runs `sd-solo`, never a shell. Or the API on port 4249 ([API access](09-api-access.html)), which takes the account password. The agent runs on another computer, as a client.
3. **Give it the account password and nothing else**, never the administrator password, the global password or your Linux password. Change the account password when the work ends (`set.password`).
4. **Prove the walls** before you trust them. As the agent, try what the guide says is refused: `sh`, `os.execute`, a path outside the account, `config`, a `copy` into the VOC, `catalog ... global`. Each must be refused in the words the guide quotes. If one succeeds, a switch is on that you did not mean to turn on.
5. **Keep the hand-over human.** The agent proposes; a person reads the program and puts it where it belongs.
6. **Take it back:** delete the agent's line from the key file and change the account password.

**Shape 2: the agent has a computer of its own.** Use it when the agent needs the shell or OS access, or must not see your account. The wall is the operating system, and the computer holds nothing you want to keep from the agent.

1. **Get the computer.** A separate machine, a virtual machine, or a container with its own IPC namespace and its own network address. Not a second Linux user on yours.
2. **Install Solo there** as an ordinary Linux user of its own ([Installing](01-installation.html)), choosing **new** account and administrator passwords, and turning on the shell or OS access only for what the project needs.
3. **Give it a workspace** with copies of what it needs to read. Do not mount your real data.
4. **Give it a way in:** the agent pipes commands to `sd-solo` as that installation's Linux user (the first line is the account password, then `TERM 200,9999`, and the last is `OFF`), or it comes in over ssh to port 4251.
5. **Prove the walls**, as above; with a switch on, the matching probe is allowed, and that is what you chose.
6. **Take it back** by stopping or deleting the virtual machine or container.

## What this does not give you

There are no scopes inside the account: the agent can read, change and delete everything in `sduser`, and with a switch on everything its Linux user can reach, which is why your data is somewhere else or behind shape 2. A switch turned on later takes the refusal away from then on. The audit trail names what the agent did, not what it saw. The MCP door ([The MCP door](25-the-mcp-door.html)) is another way in, off as delivered, run by an MCP client on this computer; this recipe does not use it, and the project-scope `.mcp.json` example in the source repository is the shape of a remote door that is not built, not something to use: Solo's own ssh server runs `sd-solo` for every sign-in and discards the command the client asked for. And no setting protects you from a prompt that persuades the agent to ask *you* for more: the answer to a request for the administrator password, or for a switch, is yours, and the safe default is no.

## See also

[Operating system access](06b-operating-system-access.html) ·
[Security](12-security.html) ·
[The MCP door](25-the-mcp-door.html) ·
and, in the User set, the page *SD and AI Agents*.
