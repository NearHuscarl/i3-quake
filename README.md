# i3 quake

Quake mode for terminal in [i3](https://github.com/i3/i3) window manager

![quake](https://github.com/NearHuscarl/i3-quake/blob/master/screen/quake.png)

## Requirements
* i3wm
* i3ipc (pip install i3ipc)
* terminal supported:
	* alacritty
	* gnome-terminal
	* roxterm
	* st
	* termite
	* urxvt
	* xfce4-terminal
	* xterm

## Usage
Open terminal dropdown in quake style (or bottom up). Each workspace can
have one terminal in quake mode. If the focused workspace already have
quake terminal. The next command will toggle visiblity state of said
console regardless of the input arguments

## Options

```bash
$ i3_quake -h
usage: i3_quake [-h] [-i] [-p POS] [-H RATIO] [-t TERM] [CMD]

positional arguments:
  CMD                   command to execute in new terminal

optional arguments:
  -h, --help            show this help message and exit
  -i, --in-place        set current terminal at top down position and execute
                        cmd
  -p POS, --pos POS     open terminal at top or bottom (default is top)
  -H RATIO, --height RATIO
                        height of terminal in percentage of monitor height
  -t TERM, --term TERM  terminal name (default is xterm)
```

## Examples:
```bash
$ i3_quake # default use xterm and open $SHELL
```
```bash
$ i3_quake -H 0.6 -t termite htop
```
```bash
$ i3_quake -p bottom -H 0.2 -t xfce4-terminal ipython
```

## Flickering:
Add this line to i3 config (~/.config/i3/config) to hide the terminal
immediately when created to prevent flickering issue

```i3
for_window [title="quake_.*"] move window to scratchpad
```

## Credit
Original work is [i3-quickterm](https://github.com/lbonn/i3-quickterm) by [ibonn](https://github.com/lbonn)
which will open terminal dropdown from selected shell in rofi dmenu. i3-quake generalize the idea
with command argument to run when open terminal.

## Licenses
**[BSD 3 Clauses](https://github.com/NearHuscarl/i3-quake/blob/master/LICENSE.md)**
