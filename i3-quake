#!/bin/env python

""" quake it """

import argparse
import os
import sys

import shlex
import i3ipc

# pylint: disable=invalid-name

i3 = i3ipc.Connection()

def get_args():
	""" get script args """
	def position(pos):
		""" position argument handling """
		if pos != 'top' and pos != 'bottom':
			raise ValueError('argument should be top or bottom, not both')
		return pos

	parser = argparse.ArgumentParser()
	parser.add_argument('-i', '--in-place', action='store_true',
			help='set current terminal at top down position and execute cmd')
	parser.add_argument('-p', '--pos', type=position, default='top',
			help='open terminal at top or bottom (default is top)')
	parser.add_argument('-H', '--height', dest='ratio', type=float, default=0.25,
			help='height of terminal in percentage of monitor height')
	parser.add_argument('-t', '--term', default='xterm', help='terminal name (default is xterm)')
	parser.add_argument('cmd', metavar='CMD', default='{$SHELL}', nargs='?',
			help='command to execute in new terminal')

	return parser.parse_args()

def TERM(executable, execopt='-e', execfmt='expanded', titleopt='-T'):
	""" Helper to declare a terminal in the hardcoded list """
	if execfmt not in ('expanded', 'string'):
		raise RuntimeError('Invalid execfmt')

	fmt = executable

	if titleopt is not None:
		fmt += ' ' + titleopt + " '{title}'"

	fmt += ' {} {{{}}}'.format(execopt, execfmt)

	return fmt

TERMS = {
		'alacritty': TERM('alacritty', titleopt='-t'),
		'gnome-terminal': TERM('gnome-terminal', execopt='--', titleopt=None),
		'roxterm': TERM('roxterm'),
		'st': TERM('st'),
		'termite': TERM('termite', execfmt='string', titleopt='-t'),
		'urxvt': TERM('urxvt'),
		'xfce4-terminal': TERM('xfce4-terminal', execfmt='string'),
		'xterm': TERM('xterm'),
		}

QUAKE_NAME = 'quake_'

def quoted(string):
	""" quoting string """
	return "'" + string + "'"

def expand_command(cmd, **rplc_map):
	""" return a tuple of command and its options/parameters with environment
	variables expanded """
	replacement_map = {'$' + key: val for key, val in
	os.environ.items()}
	replacement_map.update(rplc_map)

	return shlex.split(cmd.format(**replacement_map))

def get_current_workspace():
	""" get info of focused workspace """
	return [ws for ws in i3.get_workspaces() if ws['focused']][0]

def get_current_workspace_info():
	""" get even more info of focused workspace """
	workspace = get_current_workspace()
	tree = i3.get_tree()
	return [c for c in tree.descendents()
			if c.type == 'workspace' and c.name == workspace['name']][0]

def get_name():
	""" get quake terminal name of current workspace """
	ws_num = get_current_workspace()['num']
	return QUAKE_NAME + str(ws_num)

def move_back(title):
	""" hide container """
	i3.command('[title={}], floating enable, move scratchpad'.format(title))

def toggle_display(title):
	""" show/hide container """
	i3.command('[title={}], scratchpad show'.format(title))

def show(title):
	""" show container """
	i3.command('[title={}], focus'.format(title))

def toggle_quake(term, pos, ratio, cmd):
	""" toggle quake """
	quake_term = i3.get_tree().find_named(get_name())

	# quake terminal not exist yet
	if not quake_term:
		term_cmd = TERMS.get(term, 'xterm')
		quake_cmd = "{} -i -p {pos} -H {height} -t {term} {exec}".format(
				sys.argv[0], pos=pos, height=ratio, term=term, exec=cmd)
		term_cmd = expand_command(term_cmd,
				title=get_name(), expanded=quake_cmd, string=quoted(quake_cmd))

		try:
			# print(term_cmd)
			os.execvp(term_cmd[0], term_cmd)
		except FileNotFoundError:
			print('{} executable not found. Please select another terminal using --term flag'
					''.format(term_cmd[0]))
			sys.exit(1)
	else: # toggling scratchpad
		quake_term = quake_term[0]
		toggle_display(quake_term.name[0])

def quake(title, pos, ratio):
	""" resize terminal to make it looks top down or bottom up """
	workspace = get_current_workspace()
	ws_x = workspace['rect']['x']
	ws_y = workspace['rect']['y']
	ws_width = workspace['rect']['width']
	ws_height = workspace['rect']['height']

	width = ws_width
	height = int(ws_height * ratio)
	posx = ws_x

	if pos == 'bottom':
		posy = ws_y + ws_height - height
	else: # top
		posy = ws_y

	i3.command('[title={title}],'
			'resize set {width} px {height} px,'
			'move absolute position {posx}px {posy}px,'
			'move scratchpad,'
			'scratchpad show'
			''.format(title=title, posx=posx, posy=posy, width=width, height=height))

def launch_inplace(pos, ratio, cmd):
	""" make current terminal top down """
	title = get_name()
	move_back(title)
	quake(title, pos, ratio)
	show(title)
	cmd = expand_command(cmd)
	os.execvp(cmd[0], cmd)

if __name__ == '__main__':
	args = get_args()
	if args.in_place:
		launch_inplace(args.pos, args.ratio, args.cmd)
	else:
		toggle_quake(args.term, args.pos, args.ratio, args.cmd)
