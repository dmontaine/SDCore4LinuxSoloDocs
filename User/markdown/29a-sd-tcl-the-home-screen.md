Title: SD TCL - The Home Screen
Subtitle: A start screen of tiles: your tools, a terminal and sign out.

```
home
menu
exit
```

**`home`** opens a full-screen start screen for your account: a page of **tiles**,
each of which starts something. **`menu`** does the same (it is the plain word for
"bring up the full-screen menu", at the prompt or in the terminal the screen gave
you), and **`exit`** is the way back to it from that terminal. **A session starts
here as delivered**, not at the command prompt (see *Where a session starts*, below);
you can always type `home` or `menu` at the prompt to open it.

The keys, menus, boxes and pick lists that every full-screen page shares are described once in
[Using the Full-Screen Pages](29d-sd-tcl-using-the-full-screen-pages.html); this page adds what is
particular to the home screen.

It needs a window of at least **140 columns by 43 lines** (the same as the other
full-screen tools) and a real terminal. In a smaller window it says so, with the
size it found. If you typed `home` at the prompt, you stay at the prompt; if the
session starts in the home screen, you are **signed out** (*This account starts at
the home screen, which could not open, so the session is signed out*), whether or not
the Terminal tile is on: a small window is never a way to the prompt. In a session with no terminal (a
pipe, a phantom, a script) it says `home needs a terminal` and reads nothing, so
a script is never taken for someone typing. **In a bigger window the tiles spread over
the whole window**: as many columns of tiles as fit, each tile a little
wider, and the page follows the window when it is resized; at 140 by 43 it looks as it always did.
If the window is made smaller than 140 by 43 while the home screen is up, it holds the keyboard
behind a warning that gives both sizes until the window is big enough again.

## What is on it

```
 File  Go  View  AI  Help                                                       home  don@sduser

   don @ sduser                                                                      -> Sign out
   Thursday 22 Oct 2026  -  18:52

   My tools
     [=]                  </>
     Editor               IDE
     edit a record        programs and files

   Everyone
     >_                                          ^K
     Terminal                                    Keys
     type exit to come back                      test your keyboard

   Develop
     [F]                  [V]                    [D]                    [Q]
     Files                Vocabulary             Dictionary             Queries
     files and indices    verbs and pointers     items and indices      build a report
     ...
```

| tile | what it does |
|---|---|
| **Editor** | runs `sed` (it asks which file and record) |
| **IDE** | runs `ide` |
| **Terminal** | leaves the screen for the ordinary command prompt (below) |
| **Keys** | runs `tool.keys`, the terminal and keyboard diagnostic |
| **Files** · **Vocabulary** · **Dictionary** · **Queries** · **Bulk update** · **Programs** · **Tests** | the group *Develop*: each opens a development page (`tool.files`, `tool.voc`, `tool.dict`, `tool.query`, `tool.bulk`, `tool.catalog`, `tool.test`; see [The Development Tools](29c-sd-tcl-the-development-tools.html)) |
| **Sign out** | asks `Sign out of SD?` and ends the session |

**A tile starts a command exactly as you would have typed it.** It adds no
access: if the command needs a switch your account does not have, you get the same
refusal you would at the prompt (`set.os.access`). A command that takes the screen
(the editors, `tool.keys`) gives it back when you leave it, and the home screen is
drawn again; a command that only prints waits for Enter before the screen returns.

After **`admin`** the page shows `administrator` at the top and at the right end of
the menu bar, the group *My tools* is called *Tools*, and three groups of administrator's
tiles stand above it. Type `admin`, then `home`, to see it.

| group | tile | runs |
|---|---|---|
| *System* | **Menu management** (tiles and start) | `tool.menus` |
| | **Audit trail** (who did what) | `tool.audit` |
| | **Configuration** (sd.conf and SD) | `tool.config` |
| | **Bypass programs** (readers for pdf) | `tool.viewers` |
| *People and access* | **Sessions and locks** (who is on) | `tool.sessions` |
| | **Gates** (who may do what) | `tool.gates` |
| *Data, network and AI* | **Backups** (save and restore) | `tool.backups` |
| | **Remote servers** (doors and files) | `tool.remote` |
| | **AI** (model and key) | `tool.ai` |

(SD Core, the multi-user product, also has a *Switch account* tile and pages for its
accounts; Solo has one account, so it has none.)

**The page scrolls.** It is laid out at its full height and only the rows that fit are
drawn, with a scroll bar in the last column when there is more than fits, so the
number of tiles can pass the height of the screen. The header (your name, the date
and time, and Sign out) stays where it is.

## Keys and the mouse

| key | does |
|---|---|
| the arrow keys | move to the nearest tile in that direction (a tile in the same row or column comes first) |
| Tab, Back-tab | the next and previous tile, in reading order, round the tiles (they never go to the AI line) |
| Home, End | the first and the last tile |
| Page Down, Page Up | the first tile of the next and of the previous group |
| a letter | the next tile whose label begins with it |
| / | find a tile: a window of the page's tiles; type any part of a tile's name (or of its group) to narrow the list; Enter moves the focus to the tile chosen (Enter on the page then starts it), Esc leaves the focus where it was |
| Enter, the space bar | start the tile |
| Esc, Backspace | back one page, when you are in a sub-menu (below); on the top page Esc opens the menu bar, as F10 does, and Backspace does nothing |
| F1 | help on the home screen |
| F5 | build the page again |
| F6, Shift-F6 | move to the AI line, and from it back to the tile you were on, when the AI is on (when it is not, the status line says so); the same keys move between the boxes of every page |
| F10, Alt and a title's letter, a click on the top row | open the menu bar (File, Go, View, AI, Help). Every entry shows its key and does what the key does: **File** Sign out (Ctrl-Q, or no key when you typed `home` at the prompt) and, in that case, Back to the prompt (Ctrl-Q); **Go** Back (Esc or Backspace, only in a sub-menu), Terminal (Alt-T), Find a tile (/), Start the tile (Enter); **View** Refresh (F5), Recent group on or off (Ctrl-R), Theme (Ctrl-T); **AI** Ask the AI (F6), dimmed when the AI is not on; **Help** Help on this page (F1), Keys (Ctrl-K: the keys of the home screen). An entry whose tile the administrator turned off is dimmed, and its key only says so. The Keys tile still opens the keyboard and terminal page `tool.keys` |
| Ctrl-T | the next color theme, kept for you: every page, the IDE, SED and the debugger start in it from then on (see *Using the full-screen pages*) |
| Ctrl-K | the keys of the home screen, in a box |
| F2 | find an action: every entry of the menus in one list (*Go > Terminal*, with its key); type part of a name to narrow it, Enter chooses. An entry that cannot be used now says *(not now)* and only explains why |
| a right click, Shift-F10 | the menu of the page's actions at the pointer (Shift-F10: in the middle), ending with *Find an action...* (F2). The home screen has one box, so the right click does not move anything first |
| Ctrl-Q | sign out (asks first; **Cancel** is the button Enter chooses, so a stray Enter signs nobody out). **When you typed `home` at the prompt, Ctrl-Q goes back to that prompt instead** and does not ask; the row then reads *^Q prompt* |
| Alt-T | the Terminal tile |

Backspace, Alt-T, Ctrl-R, Ctrl-T and Ctrl-K are not on the row of keys at the foot of
the screen (F1 lists every key). The row shows Enter, F1, F5, Ctrl-Q and /, Esc in a
sub-menu, and F6 when the AI is on.

**Which Ctrl-Q you have.** A screen your session started in (the start setting is
the home screen) is the session's own: Ctrl-Q signs out. `exit` after the Terminal tile
brings back the same kind of screen you left. A screen you typed `home` for at the prompt is a visit: Ctrl-Q
leaves it for the prompt, and the session goes on. When the administrator has turned the
Terminal tile off there is no prompt to go back to, so Ctrl-Q signs out
whichever way the screen was started.

A **click** starts a tile; the **wheel** scrolls the page three rows; a click on the
menu bar opens the menu under it. The foot of the screen is the same as the
other full-screen tools - the status line, a border and the row of function
keys - with a hint line above it for the keys the row does not show: the
arrows, a letter and PgDn.

On a terminal with color the tiles are filled in the color of their group; on one
without, each tile is a box and the one with the focus is in reverse video, so
nothing depends on color.

## The terminal, and `exit`

The **Terminal** tile gives the screen back and shows

```
Terminal - type exit to return to the home screen
:
```

That is the ordinary command prompt, with all of its features (the command stack,
paragraphs, select lists, everything you can type at a prompt that did not come
from the home screen). **`exit`** (or **`menu`**) brings the home screen back. At a
prompt that *did not* come from the home screen, `exit` does what `quit` does and ends
the session. The home screen remembers that you left by the Terminal tile for the rest
of the session, so `exit` goes on coming back to it.

**Sign out** (the tile, the File menu, or Ctrl-Q when your session started at the home
screen) asks first and then ends the session as `quit` does.

## Where a session starts

**Menu is the delivered start**: a session on a terminal opens the home screen, and
the command prompt is what is left when it returns. The administrator can change it
with **`set.start.default menu`** or **`set.start.default terminal`** (after `admin`;
without a word the verb says what it is now). It is read when you sign on. **After an
upgrade, if you want the old start (the command prompt), type `admin` and then
`set.start.default terminal`.** Starting in the menu confines nobody: the Terminal tile
is there unless it is turned off (below).

The screen is never given to a session that cannot use it: a pipe, a phantom or an
API server. A session that starts in the menu in a window under 140 by 43 is signed
out (above).

## Tiles the administrator turns off

Every tile is on as delivered, and the administrator can turn a tile off for the account
with **`set.tile sduser tile off`** (or on the *Menu management* page): it is then not
on the page, and its menu entry cannot be chosen. The tile ids are `editor`, `ide`,
`terminal`, `keys`, `files`, `voc`, `dict`, `query`, `bulk`, `catalog`, `test`, `signout` and
`recent` (the Recent group, below); **sign out cannot be turned off**. `set.tile` alone
says what is off.

If the **Terminal** tile is off and the session starts in the home screen, you are never
given the command prompt: when the screen ends (an error, the cursor
lock) you are signed out. A window too small for the screen signs you out whatever the
tiles say. If the account starts at the prompt it keeps it.
**It is a wall only for the prompt the screen gives**: the programs behind the other
tiles (`sed`, `ide` and the rest) are the account's own, so to keep the account to the tiles you
give it, turn those off too. A shell needs the `sh` switch (`set.os.access`), which is
off as delivered.

## Your own programs: `my.menu`

```
my.menu
set.pins on | off
```

**Only if the administrator has allowed it** (`set.pins on`; it is off as delivered).
The home screen then has a group **My programs**, with a tile **Add a program** that
opens `my.menu`, and a tile for each program you have added. A program is a **label**
(up to 21 characters), a **command** (what you would type at your prompt), an
optional three-character **glyph**, and whether the screen should **wait for Enter**
when the command has finished printing.

| key | does |
|---|---|
| Ctrl-N | add a program (it asks for the label, the command, the glyph and the wait) |
| Enter | change the selected one |
| Del | remove it (asks first; Cancel is the default) |
| U, D | move it up or down the list |
| / | go to the next program whose label or command holds a text you type (any case; round to the top) |
| Esc, F10 | open the menu bar (its menus Program, File and Help): the same actions by name; with no program in the list, Change, Remove and the moves are shown dimmed |
| F1, `?`, a click on the `[?]` | help on this page; the Help menu has the same |
| Ctrl-Q | leave; the home screen is built again and shows the list |

In a window bigger than 140 by 43 the list takes the extra rows and the command column
the extra width.

**It manages only your own list.** The standard tiles are not touched. A program of
yours runs the command exactly as you would type it, so it adds no access: the
switches apply as at the prompt. The list is the record `home` of the file `&menus&`
in your account. `set.pins` alone says whether it is allowed; `set.pins off` hides the
group and `my.menu` says it is not allowed (the list stays in your file).

## The Recent group

```
View > Recent group
```

A group of **the tiles you started last**, newest first, at the top of the page: six at
most. **It is off as delivered.** Turn it on in the **View** menu (*Recent group on or off*,
or Ctrl-R; the same entry turns it off, and the status line says which it is now). The first time it is on, SD makes the
file `&menus&` in your account if it is not there, as `my.menu` does, and keeps the
list in its record `recent`; the list is one word `on`, then one id for each tile.

* **What is kept is an id, never a command.** The group is built from the page as it is
  now: a tile that the administrator has turned off, or that is not on your page, is not
  shown, and a record that someone edited by hand cannot make the page offer a tile it does
  not have. Starting a tile of the group runs the command of that tile, exactly as the tile
  in its own group would, and puts it first.
* **The tiles you added yourself (`my.menu`) are not kept here**: they are your own list
  already.
* **Turning it off forgets the list** (the record is deleted), and **while it is off nothing
  is written**: starting a tile leaves your file as it was.
* **The administrator can turn it off** (`set.tile sduser recent off`, or
  the *Menu management* page): the group is then not on your page, nothing is read or written
  for it, and the View entry cannot be chosen. What you had kept stays in your file.
* If the file `&menus&` cannot be made or written, turning the group on says so (*The
  Recent group could not be turned on*) and leaves nothing behind. `my.menu` makes the file
  the same way and meets the same limit.

## Pages inside pages: sub-menus

A tile can open a **second page of tiles**. The new page has a **Back** tile at the left of
the header (your name moves right to make room), and the status line shows where you are:
`[Home > Develop]`. **Esc**, **Backspace**, the Back tile and the *Back* entry of the Go menu (Esc or Backspace)
each go back one page, with the tile you left selected.
Pages can be nested **eight deep**; a page that is already open above cannot be opened again
(*... is already open above this page*), and a menu none of whose tiles is on your page has no
tile to open it.

**No tile of the delivered page opens a sub-menu yet.** The tiles you start inside a
sub-menu are kept by the Recent group like any other.

## Help on a setting, and the documentation

On the *Menu management* and *Bypass programs* pages every setting has a **`[?]`** at the
end of its row; F1, `?` or a click opens a box that says what it means, with a
*Documentation* button that shows the guide page in the PDF reader (`pdf`, below). The
text is the file `context.help` of the account.

```
pdf
pdf name
```

`pdf` alone lists the documents (the PDF files in the `pdf` directory of the install
directory) and `pdf name` shows one in the PDF reader of the computer you are sitting at.
It works even when the account has no `os.execute` switch, because the reader it starts is
one the administrator allowed for this one purpose (`set.viewer`), and the document is
always one of those files, never a path you type. With no reader allowed (the delivered
state) it says `No viewer has been allowed for documents`; over plain ssh, which has no
display, it says so and does nothing.

## The AI line

When the AI is on (the administrator has chosen a provider, given it a model and a key,
**and turned the AI gate on with `set.ai gate on`**, which is off as delivered; see
[Asking the AI](29b-sd-tcl-asking-the-ai.html) and
[The AI helper](24-the-ai-helper.html)) the home screen has one more row, just above the
status line, that starts **`AI>`**. Without the AI there is no such row and nothing on this
page changes.

**Press F6, choose AI > Ask the AI, or click the row**, type
what you want in your own words, and press Enter:

```
AI> open the ide
AI> show me who I am
```

Enter does not send anything. It shows the provider, the model, the size of all that will be
sent, and **your request**. SD's own text that goes with it (one fixed instruction, the list of
what this page can do, what the page says about itself) is sent and counted, but not shown. **Cancel is the default
button**, so Enter alone sends nothing; choose **Send** (press `s`) to send. While the AI
answers the row says so and **Esc gives the request up**.

The answer comes in a popup. It says what the AI said, and it may *propose* one thing:

| The AI proposes | What you see | What happens |
|---|---|---|
| a **tile** that opens a tool (Editor, IDE, Keys, the Develop tiles, and after `admin` the administrator's tiles) | `The AI proposes: IDE - programs and files`, with **Accept** and Reject | **Accept** starts the tile exactly as a click on it would. Enter alone accepts, because a tile only opens a tool you could open anyway. |
| the **Terminal** tile, or one of **your own programs** on the page | the same, with **Accept** and Reject | **Accept** starts it as a click would. **Enter alone rejects**: the Terminal tile leaves the screen for the command prompt and a program of yours runs a command you wrote, so neither starts unless you choose Accept. This is decided for the proposal: having a program of your own on the page does not change what Enter does for the other tiles. |
| a **command** | the command on a line of its own, with **Run** and Cancel | **Run** runs it as the Terminal tile's prompt would, and waits for Enter. **Enter alone cancels**: a command never runs unless you choose Run. |
| nothing | only what it said, with Close | nothing |

What it cannot do: it is never offered **Sign out**; a tile that is not on your page is
ignored, with a note; a command with a control character in it or of more than 200
characters is not shown; anything that is not one JSON object is shown as words, with a
note, and nothing is proposed. The accepted tile or command runs with **your** rights: the
switches of your account (`set.os.access`) apply as they would if you had typed it.

The audit trail records that you asked (`ai request account=sduser program=tk_ai
bytes.out=...`), never what you asked or what came back.

**What is different from SD Core.** Solo has one account, so there is no *Switch account*
tile to withhold and no per-account permission to ask for: the one switch is the AI gate,
which the administrator turns on with `set.ai gate on` (never `modify.account`). And the
helper that talks to the provider runs **as you**, not as a user of its own, so any program
you run, not only SD, can reach it; [The AI helper](24-the-ai-helper.html) says what that
weakens. The row, the preview and the proposals are the same.

## Not yet

Programs the administrator assigns to the account, a menu builder (grouping your own
programs into sub-menus) and a live line on your own tiles are planned and not
built; today the page is the table above plus your own programs and, if
you turn it on, the Recent group.
