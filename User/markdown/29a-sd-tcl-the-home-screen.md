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
| **Receive files** | runs `receive` (*Moving files in and out*), then waits for Enter so you can read what came; it is there only when a transfer can be opened: `XFER=1` in `sd.conf` and the account's switch on (`set.xfer.access on`) |
| **Terminal** | leaves the screen for the ordinary command prompt (below) |
| **Keys** | runs `tool.keys`, the terminal and keyboard diagnostic |
| **Files** · **Vocabulary** · **Dictionary** · **Queries** · **Bulk update** · **Programs** · **Tests** | the group *Develop*: each opens a development page (`tool.files`, `tool.voc`, `tool.dict`, `tool.query`, `tool.bulk`, `tool.catalog`, `tool.test`; see [The Development Tools](29c-sd-tcl-the-development-tools.html)) |
| **Sign out** | asks `Sign out of SD?` and ends the session |

**A tile for one of SD's own pages (a `tool.*` command) is shown only when that command is in
your account's VOC**, so a tile never offers a command you cannot run; it appears once the
command is there. A pinned tile is not asked.

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

## Continued in

[SD TCL - The Home Screen: my.menu and the AI
Line](29a-sd-tcl-the-home-screen-2-my-menu-and-the-ai-line.html) — my.menu,
sub-menus and the AI line.
