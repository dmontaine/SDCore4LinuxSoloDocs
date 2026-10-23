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

It needs a window of at least **140 columns by 43 lines** (the same as the other
full-screen tools) and a real terminal. In a smaller window it says so, with the
size it found, and leaves you at the prompt; in a session with no terminal (a
pipe, a phantom, a script) it says `home needs a terminal` and reads nothing, so
a script is never taken for someone typing.

## What is on it

```
 File  Go  View  AI  Help                                                       home  don@sduser

   don @ sduser                                                                      -> Sign out
   Thursday 22 Oct 2026  -  18:52

   My tools
     [=]                  </>                    [ ]
     Editor               IDE                    Record editor
     edit a record        programs and files     fields of a record

   Everyone
     >_                                          ^K
     Terminal                                    Keys
     type exit to come back                      test your keyboard
```

| tile | what it does |
|---|---|
| **Editor** | runs `sed` (it asks which file and record) |
| **IDE** | runs `ide` |
| **Record editor** | runs `ced` |
| **Terminal** | leaves the screen for the ordinary command prompt (below) |
| **Keys** | runs `tool.keys`, the terminal and keyboard diagnostic |
| **Sign out** | asks `Sign out of SD?` and ends the session |

**A tile starts a command exactly as you would have typed it.** It adds no
access: if the command needs a switch your account does not have, you get the same
refusal you would at the prompt (`set.os.access`). A command that takes the screen
(the editors, `tool.keys`) gives it back when you leave it, and the home screen is
drawn again; a command that only prints waits for Enter before the screen returns.

After **`admin`** the page also has a group called *System*, with the tiles
**Menu management** (`tool.menus`) and **Bypass programs** (`tool.viewers`); it shows
`administrator` at the top and at the right end of the menu bar. Type `admin`, then
`home`, to see it. (SD Core, the multi-user product, also has a *Switch account*
tile; Solo has one account, so it has none.)

**The page scrolls.** It is laid out at its full height and only the rows that fit are
drawn, with a scroll bar in the last column when there is more than fits, so the
number of tiles can pass the height of the screen. The header (your name, the date
and time, and Sign out) stays where it is.

## Keys and the mouse

| key | does |
|---|---|
| the arrow keys | move to the nearest tile in that direction (a tile in the same row or column comes first) |
| Tab, Back-tab | the next and previous tile, in reading order |
| Home, End | the first and the last tile |
| Page Down, Page Up | the first tile of the next and of the previous group |
| a letter | the next tile whose label begins with it |
| Enter, the space bar | start the tile |
| F5 | build the page again |
| F10 | open the menu bar (File, Go, View, AI, Help) |
| Ctrl-Q | sign out (asks first; **Cancel** is the button Enter chooses, so a stray Enter signs nobody out) |

A **click** starts a tile; the **wheel** scrolls the page three rows; a click on the
menu bar opens the menu under it. The foot of the screen is the same as the
other full-screen tools: a hint line, the status line, a border and the row of
function keys.

On a terminal with colour the tiles are filled in the colour of their group; on one
without, each tile is a box and the one with the focus is in reverse video, so
nothing depends on colour.

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

**Sign out** (the tile, the File menu, or Ctrl-Q) asks first and then ends the
session as `quit` does.

## Where a session starts

**Menu is the delivered start**: a session on a terminal opens the home screen, and
the command prompt is what is left when it returns. The administrator can change it
with **`set.start.default menu`** or **`set.start.default terminal`** (after `admin`;
without a word the verb says what it is now). It is read when you sign on. **After an
upgrade, if you want the old start (the command prompt), type `admin` and then
`set.start.default terminal`.** Starting in the menu confines nobody: the Terminal tile
is there unless it is turned off (below).

The screen is never given to a session that cannot use it: a window under 140 by 43,
a pipe, a phantom or an API server.

## Tiles the administrator turns off

Every tile is on as delivered, and the administrator can turn a tile off for the account
with **`set.tile sduser tile off`** (or on the *Menu management* page): it is then not
on the page, and its menu entry cannot be chosen. The tile ids are `editor`, `ide`, `ced`,
`terminal`, `keys` and `signout`; **sign out cannot be turned off**. `set.tile` alone
says what is off.

If the **Terminal** tile is off and the session starts in the home screen, you are never
given the command prompt: when the screen ends (a window too small, an error, the cursor
lock) you are signed out. If the account starts at the prompt it keeps it.
**It is a wall only for the prompt the screen gives**: the programs behind the other
tiles (`sed`, `ide`, `ced`) are the account's own, so to keep the account to the tiles you
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
| F2 | add a program (it asks for the label, the command, the glyph and the wait) |
| Enter | change the selected one |
| F3 | remove it (asks first; Cancel is the default) |
| F4, F6 | move it up or down the list |
| Ctrl-Q | leave; the home screen is built again and shows the list |

**It manages only your own list.** The standard tiles are not touched. A program of
yours runs the command exactly as you would type it, so it adds no access: the
switches apply as at the prompt. The list is the record `home` of the file `&menus&`
in your account. `set.pins` alone says whether it is allowed; `set.pins off` hides the
group and `my.menu` says it is not allowed (the list stays in your file).

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

## Not yet

Programs the administrator assigns to the account, sub-menus that open and return, a
group of the last commands you ran, and a live line on a tile are planned and not
built.
