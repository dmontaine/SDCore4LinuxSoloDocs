Title: Giving an AI Agent an Installation of Its Own
Subtitle: A recipe with no code: a computer of the agent's own with its own SD Core for Linux Solo, so that the operating system is the wall, and why it cannot be a second Linux user on yours.

An AI agent that works on an SD project needs a place to work and a limit on what it can reach. Prompts, tool scopes and promises inside the agent's own software protect against mistakes and against instructions hidden in the data it reads. **They do not stop an agent that can run as the Linux user who owns the installation**: it can read the kept copy of the account password, open an interactive `sd-solo` as you and do whatever you can. The wall that holds is the operating system's, and in this product it can only be a computer.

> **The agent is never you.** Do not give it your passwords, your ssh key or a terminal that is already signed in, and do not point it at your own installation.

## Why a computer, and not a Linux user

SD Core for Linux Solo has **one account**, `sduser`, and no per-account switches ([The account and its passwords](05-account-types.html)). There is no second account to make for an agent and nothing inside the account that scopes it: whoever has the account password has all of it. **Only one SD can run on a computer** ([Running SD](03-running-sd.html)): the shared-memory name is machine-wide, so a second Solo for another Linux user would attach to the first one's segment. So the agent cannot have an installation of its own beside yours, and the two ssh and API ports (4251 and 4249) are fixed as well.

And **the shell is not a wall here.** `sh`, `!`, `os.execute` and `openseq` of any path run with the rights of the Linux user ([Operating system access](06b-operating-system-access.html)); no switch turns them off. What an agent's installation can reach is what its Linux user can reach, so the agent's computer should hold nothing you want to keep from it.

## The recipe, in outline

The full recipe is `agent-pack/recipe/agent-account.md` in the SD Core for Linux Solo source repository, with a template for the project's `CLAUDE.md` and a Claude Code skill that carries the SD knowledge (see the page *SD and AI Agents* in the User set).

1. **Get the computer.** A separate machine, a virtual machine, or a container with its own IPC namespace and its own network address. Not a second Linux user on yours.
2. **Install Solo there** as an ordinary Linux user of its own ([Installing](01-installation.html)), choosing **new** account and administrator passwords. The agent is given the account password. It is never given the administrator password, and it never asks for it.
3. **Give it a workspace** with copies of what it needs to read. Do not mount your real data.
4. **Give it a way in:** the agent pipes commands to `sd-solo` as that installation's Linux user (the first line is the account password, then `TERM 200,9999`, and the last is `OFF`), or it comes in over ssh to port 4251 with a key you added to Solo's own key file ([ssh access](08-ssh-access.html)) and types the account password at the SD prompt.
5. **Prove the walls** before you trust them. As the agent, try what the guide says is refused: `config`, a `copy` into the VOC, `catalog ... global`, a program's write to the VOC, a restricted statement. Each must be refused in the words the guide quotes. Then read what is **not** refused (`sh`, `os.execute`, a path outside the account): if that is not acceptable, the computer is the wrong place for the agent.
6. **Keep the hand-over human.** The agent proposes; a person reads the program on the agent's computer and copies it to yours. The agent has no route to your computer, so it cannot promote its own work.
7. **Take it back** by stopping or deleting the virtual machine or container, or by removing the key and changing the account password on that installation.

## What this does not give you

There are no scopes inside the account: the agent can read, change and delete everything in `sduser` and everything its Linux user can reach, which is why your data is somewhere else. The audit trail names what the agent did, not what it saw. There is no MCP door yet (`sd-mcp` is planned and not built), and the project-scope `.mcp.json` in the source repository is a shape, not something to use: Solo's own ssh server runs `sd-solo` for every sign-in and discards the command the client asked for. And no setting protects you from a prompt that persuades the agent to ask *you* for more: the answer to a request for the administrator password is yours, and the safe default is no.

## See also

[Operating system access](06b-operating-system-access.html) ·
[Security](12-security.html) ·
and, in the User set, the page *SD and AI Agents*.
