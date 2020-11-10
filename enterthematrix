#!/usr/bin/env python3

NAME        = 'enterthematrix'
DESCRIPTION = 'controllable matrix rain animation and benchmarking tool'
VERSION     = '3.1.0'

DEBUG_DESCRIPTION              = 'show program runtime information during the animation'
BANDWIDTH_DESCRIPTION          = 'set the maximum number of character animation streams per text column'
THROUGHPUT_DESCRIPTION         = 'set the number of character animation streams to create per animation frame'
NEOS_INFLUENCE_DESCRIPTION     = 'follow the white rabbit'
ANIMATION_INTERVAL_DESCRIPTION = 'set the amount of time in seconds between animation frames'
LIMITER_DESCRIPTION            = 'limit the maximum number of streams by a factor of the limiter value'
STREAM_SIZE_MULT_DESCRIPTION   = 'set the average stream size to the number of rows times the multiplier value'
MAX_COLS_DESCRIPTION           = 'set the maximum number of text columns to animate'
MAX_ROWS_DESCRIPTION           = 'set the maximum number of text rows to animate'
MAX_FRAMES_DESCRIPTION         = 'set the maximum number of frames to animate'
EXIT_KEYS_DESCRIPTION          = 'set the list of keys to initiate exit'
USE_ASYNC_DESCRIPTION          = 'turn on async frame rendering in supported environments'
NO_USE_ASYNC_DESCRIPTION       = 'turn off async frame rendering in supported environments'
COMMANDS_DESCRIPTION           = 'show the list of key commands and exit'
BENCHMARK_DESCRIPTION          = f'run the standard {NAME} benchmark'

# DEFAULTS

DEBUG              = False
BANDWIDTH          = 2
THROUGHPUT         = 4
NEOS_INFLUENCE     = 0.004
ANIMATION_INTERVAL = 0.042
LIMITER            = 0
STREAM_SIZE_MULT   = 0.8
MAX_COLS           = 1280
MAX_ROWS           = float('inf')
MAX_FRAMES         = float('inf')
EXIT_KEYS          = ('q', 'Q', chr(0x1b))
USE_ASYNC          = True
MIN_STREAM_SIZE    = 2
COMMANDS_ENABLED   = True

KEY_COMMANDS = {
    'd': 'toggle_debug',
    'p': 'toggle_paused',
    's': 'trigger_step',
    'c': 'clear_screen',
    'b': 'increase_bandwidth',
    'B': 'decrease_bandwidth',
    't': 'increase_throughput',
    'T': 'decrease_throughput',
    'n': 'increase_neos_influence',
    'N': 'decrease_neos_influence',
    'a': 'increase_animation_interval',
    'A': 'decrease_animation_interval',
    'l': 'increase_limiter',
    'L': 'decrease_limiter',
    'm': 'increase_stream_size_mult',
    'M': 'decrease_stream_size_mult',
}

BANDWIDTH_INCREMENT          = 1
THROUGHPUT_INCREMENT         = 1
NEOS_INFLUENCE_INCREMENT     = 1e-3
ANIMATION_INTERVAL_INCREMENT = 1e-3
LIMITER_INCREMENT            = 1e-2
STREAM_SIZE_MULT_INCREMENT   = 1e-2

BANDWIDTH_RANGE          = (0, float('inf'))
THROUGHPUT_RANGE         = (0, float('inf'))
NEOS_INFLUENCE_RANGE     = (0, 1)
ANIMATION_INTERVAL_RANGE = (0, float('inf'))
LIMITER_RANGE            = (0, 1)
STREAM_SIZE_MULT_RANGE   = (0, float('inf'))
MAX_COLS_RANGE           = (1, float('inf'))
MAX_ROWS_RANGE           = (1, float('inf'))
MAX_FRAMES_RANGE         = (1, float('inf'))
MIN_STREAM_SIZE_RANGE    = (MIN_STREAM_SIZE, float('inf'))

CLEANABLE_PARAMS = (
    'throughput',
    'bandwidth',
    'neos_influence',
    'animation_interval',
    'limiter',
    'stream_size_mult',
    'max_cols',
    'max_rows',
    'max_frames',
    'min_stream_size',
)

FIRST_KATA,  NUM_KATAS  = 0xff66,   56
FIRST_ALPHA, NUM_ALPHAS = ord('A'), 26
FIRST_NUM,   NUM_NUMS   = ord('0'), 10
ALPHABET = ( # Cypher: "All I see is blonde, brunette, redhead..."
    *range(FIRST_KATA, FIRST_KATA + NUM_KATAS),
    *range(FIRST_ALPHA, FIRST_ALPHA + NUM_ALPHAS),
    *range(FIRST_NUM, FIRST_NUM + NUM_NUMS),
)

BENCHMARK_CONFIG = {
    'max_frames':         2 ** 12,
    'debug':              True,
    'bandwidth':          8,
    'throughput':         16,
    'neos_influence':     0.04,
    'animation_interval': 0,
    'max_cols':           80,
    'max_rows':           40,
    'use_async':          False,
    'commands_enabled':   False,
}

# IMPORTS

from sys import argv
from argparse import ArgumentParser
from collections import deque
from asyncio import run
from time import sleep
from random import choice, random
from math import log
from curses import (
    wrapper,
    curs_set,
    use_default_colors,
    init_pair,
    color_pair,
    COLOR_GREEN,
    A_BOLD,
)
PAIR_HIGHLIGHTED = 0
PAIR_NORMAL      = 1

# SIMULATION

class MatrixSimulation:
    def __init__(self, stdscr, **kwargs):
        self.stdscr = stdscr
        self.stdscr.nodelay(1)
        curs_set(0)
        use_default_colors()
        init_pair(PAIR_NORMAL, COLOR_GREEN, -1)
        self.pair_highlighted = color_pair(PAIR_HIGHLIGHTED) | A_BOLD
        self.pair_normal      = color_pair(PAIR_NORMAL)

        self.debug              = kwargs.get('debug', DEBUG)
        self.bandwidth          = kwargs.get('bandwidth', BANDWIDTH)
        self.throughput         = kwargs.get('throughput', THROUGHPUT)
        self.neos_influence     = kwargs.get('neos_influence', NEOS_INFLUENCE)
        self.animation_interval = kwargs.get('animation_interval', ANIMATION_INTERVAL)
        self.limiter            = kwargs.get('limiter', LIMITER)
        self.stream_size_mult   = kwargs.get('stream_size_mult', STREAM_SIZE_MULT)
        self.max_cols           = kwargs.get('max_cols', MAX_COLS)
        self.max_rows           = kwargs.get('max_rows', MAX_ROWS)
        self.max_frames         = kwargs.get('max_frames', MAX_FRAMES)
        self.exit_keys          = kwargs.get('exit_keys', EXIT_KEYS)
        self.use_async          = kwargs.get('use_async', USE_ASYNC) and self.can_use_async()
        self.min_stream_size    = kwargs.get('min_stream_size', MIN_STREAM_SIZE)
        self.commands_enabled   = kwargs.get('commands_enabled', COMMANDS_ENABLED)
        self.key_commands       = kwargs.get('key_commands', KEY_COMMANDS)

        self.bandwidth_increment          = BANDWIDTH_INCREMENT
        self.throughput_increment         = THROUGHPUT_INCREMENT
        self.neos_influence_increment     = NEOS_INFLUENCE_INCREMENT
        self.animation_interval_increment = ANIMATION_INTERVAL_INCREMENT
        self.limiter_increment            = LIMITER_INCREMENT
        self.stream_size_mult_increment   = STREAM_SIZE_MULT_INCREMENT

        self.bandwidth_range          = BANDWIDTH_RANGE
        self.throughput_range         = THROUGHPUT_RANGE
        self.neos_influence_range     = NEOS_INFLUENCE_RANGE
        self.animation_interval_range = ANIMATION_INTERVAL_RANGE
        self.limiter_range            = LIMITER_RANGE
        self.stream_size_mult_range   = STREAM_SIZE_MULT_RANGE
        self.max_cols_range           = MAX_COLS_RANGE
        self.max_rows_range           = MAX_ROWS_RANGE
        self.max_frames_range         = MAX_FRAMES_RANGE
        self.min_stream_size_range    = MIN_STREAM_SIZE_RANGE

        self.cleanable_params = CLEANABLE_PARAMS
        self.clean_params()

        self.char_streams = [ deque() for _ in range(self.max_cols) ]
        self.num_streams  = 0
        self.num_frames   = 0
        self.is_paused    = False
        self.is_stepping  = False

        if self.use_async:
            run(self.run_async())
        else:
            self.run()

    def clean_params(self):
        for param in self.cleanable_params:
            self.limit_param(param)
            self.round_param(param)

    def limit_param(self, param):
        param_val = getattr(self, param)
        param_min, param_max = getattr(self, f'{param}_range')
        if param_val < param_min:
            setattr(self, param, param_min)
        if param_val > param_max:
            setattr(self, param, param_max)

    def round_param(self, param):
        param_val = getattr(self, param)
        param_increment = getattr(self, f'{param}_increment', None)
        if param_increment is not None:
            param_precision = 0 if type(param_increment) is int else -int(log(param_increment, 10)) + 1
            rounded_param = round(param_val, param_precision) if param_precision > 0 else int(param_val)
            setattr(self, param, rounded_param)

    def can_use_async(self):
        try:
            from asyncio import to_thread
        except ImportError:
            return False
        return True

    @property
    def max_streams(self):
        return int(min(self.max_cols, self.num_cols) * self.bandwidth * (1 - self.limiter))

    @property
    def num_cols(self):
        return min(self.stdscr.getmaxyx()[1], self.max_cols)

    @property
    def num_rows(self):
        return min(self.stdscr.getmaxyx()[0], self.max_rows)

    async def run_async(self):
        from asyncio import to_thread, sleep as async_sleep, gather
        key = -1
        while self.can_run(key):
            async_next_frame = to_thread(self.next_frame)
            async_animation_interval = async_sleep(self.animation_interval)
            await gather(async_next_frame, async_animation_interval)
            key = self.stdscr.getch()
            self.handle_keypress(key)

    def run(self):
        key = -1
        while self.can_run(key):
            self.next_frame()
            sleep(self.animation_interval)
            key = self.stdscr.getch()
            self.handle_keypress(key)

    def can_run(self, key):
        return not self.is_exit_key(key) and not self.at_max_frames()

    def is_exit_key(self, key):
        return key >= 0 and chr(key) in self.exit_keys

    def at_max_frames(self):
        return self.num_frames >= self.max_frames

    def next_frame(self):
        self.num_frames += 1
        if self.is_paused and not self.is_stepping:
            return
        self.add_streams()
        self.step_streams()
        self.render_streams()
        self.remove_streams()
        if self.debug:
            self.render_debug_message()
        self.stdscr.refresh()

    def add_streams(self):
        for _ in range(self.throughput):
            if self.at_max_throughput():
                break
            col = choice(range(self.num_cols))
            if self.can_add_stream(col):
                self.add_stream(col)

    def at_max_throughput(self):
        return self.num_streams >= self.max_streams

    def can_add_stream(self, col):
        return len(self.char_streams[col]) < self.bandwidth

    def add_stream(self, col):
        size = self.get_next_stream_size()
        stream = CharStream(size)
        self.char_streams[col].append(stream)
        self.num_streams += 1

    def get_next_stream_size(self):
        size = choice(range(1, self.num_rows + 1))
        size = int(size * self.stream_size_mult)
        return max(size, self.min_stream_size)

    def step_streams(self):
        for col_streams in self.char_streams:
            for stream in col_streams:
                stream.step()

    def render_streams(self):
        for col, col_streams in enumerate(self.char_streams):
            for stream in col_streams:
                self.render_stream(stream, col)

    def render_stream(self, stream, col):
        if self.can_render_tail(stream):
            self.render_tail(stream, col)
        if self.can_render_before_head(stream):
            self.render_before_head(stream, col)
        if self.can_render_head(stream):
            self.render_head(stream, col)

    def can_render_head(self, stream):
        return stream.head_row < self.num_rows

    def render_head(self, stream, col):
        is_influenced_by_neo = random() <= self.neos_influence
        char = stream.next_char(is_influenced_by_neo)
        self.addch(stream.head_row, col, char, True)

    def can_render_before_head(self, stream):
        return stream.head_row - 1 >= 0 and \
            stream.head_row - 1 < self.num_rows and \
            not stream.latest_is_influenced

    def render_before_head(self, stream, col):
        self.addch(stream.head_row - 1, col, stream.latest_char)

    def can_render_tail(self, stream):
        return stream.tail_row >= 0 and not stream.tail_is_influenced()

    def render_tail(self, stream, col):
        self.addch(stream.tail_row, col, ord(' '))

    def addch(self, row, col, char, is_highlighted=False):
        if self.can_addch(row, col):
            color = self.pair_highlighted if is_highlighted else self.pair_normal
            try:
                self.stdscr.addch(row, col, char, color)
            except Exception:
                pass

    def can_addch(self, row, col):
        return row < self.num_rows and col < self.num_cols

    def remove_streams(self):
        for col_streams in self.char_streams:
            if self.should_remove_stream(col_streams):
                self.remove_oldest_stream(col_streams)

    def should_remove_stream(self, col_streams):
        return col_streams and col_streams[0].tail_row >= (self.num_rows - 1)

    def remove_oldest_stream(self, col_streams):
        col_streams.popleft()
        self.num_streams -= 1

    def remove_all_streams(self):
        for col_streams in self.char_streams:
            while col_streams:
                self.remove_oldest_stream(col_streams)

    def render_debug_message(self):
        row = self.num_rows - 1
        for idx, char in enumerate(self.debug_message):
            col = idx
            self.addch(row, col, char, True)

    @property
    def debug_message(self):
        return ' | '.join((
            f'STR: {self.num_streams}/{self.max_streams}',
            f'DIM: {self.num_cols}x{self.num_rows}',
            f'B: {self.bandwidth}',
            f'T: {self.throughput}',
            f'N: {self.neos_influence}',
            f'A: {self.animation_interval}',
            f'L: {self.limiter}',
            f'M: {self.stream_size_mult}',
        ))

    def handle_keypress(self, key):
        if key < 0 or not self.commands_enabled:
            return
        command = self.key_commands.get(chr(key))
        if command is not None:
            self.exec_key_command(command)
            self.clean_params()

    def exec_key_command(self, command):
        getattr(self, command)()

    def toggle_debug(self):
        self.debug = not self.debug

    def toggle_paused(self):
        self.is_paused = not self.is_paused

    def trigger_step(self):
        self.is_stepping = True
        self.next_frame()
        self.is_stepping = False

    def clear_screen(self):
        self.remove_all_streams()
        self.stdscr.clear()

    def increase_bandwidth(self):
        self.bandwidth += self.bandwidth_increment

    def decrease_bandwidth(self):
        self.bandwidth -= self.bandwidth_increment

    def increase_throughput(self):
        self.throughput += self.throughput_increment

    def decrease_throughput(self):
        self.throughput -= self.throughput_increment

    def increase_neos_influence(self):
        self.neos_influence += self.neos_influence_increment

    def decrease_neos_influence(self):
        self.neos_influence -= self.neos_influence_increment

    def increase_animation_interval(self):
        self.animation_interval += self.animation_interval_increment

    def decrease_animation_interval(self):
        self.animation_interval -= self.animation_interval_increment

    def increase_limiter(self):
        self.limiter += self.limiter_increment

    def decrease_limiter(self):
        self.limiter -= self.limiter_increment

    def increase_stream_size_mult(self):
        self.stream_size_mult += self.stream_size_mult_increment

    def decrease_stream_size_mult(self):
        self.stream_size_mult -= self.stream_size_mult_increment

class CharStream:
    def __init__(self, size):
        self.size                 = size
        self.head_row             = -1
        self.tail_row             = self.head_row - self.size
        self.latest_char          = None
        self.latest_is_influenced = False
        self.influenced_chars     = {}

    def step(self):
        self.head_row += 1
        self.tail_row += 1

    def next_char(self, is_influenced_by_neo=False):
        if is_influenced_by_neo:
            self.influenced_chars[self.head_row] = True
        self.latest_is_influenced = is_influenced_by_neo
        self.latest_char = chr(choice(ALPHABET))
        return self.latest_char

    def tail_is_influenced(self):
        if self.tail_row in self.influenced_chars:
            del self.influenced_chars[self.tail_row]
            return True
        return False

# DRIVER

def main():
    def run_animation(config):
        wrapper(MatrixSimulation, **config)

    def run_benchmark(overrides):
        from datetime import datetime
        config = BENCHMARK_CONFIG.copy()
        config.update(overrides)
        benchmark_start = datetime.now()
        run_animation(config)
        benchmark_end = datetime.now()
        print(benchmark_end - benchmark_start)

    def print_commands():
        for key, command, in KEY_COMMANDS.items():
            print(f'{key} => {command}')

    arg_parser = ArgumentParser(prog=NAME, description=DESCRIPTION)

    arg_parser.add_argument('-v', '--version', \
        action='version', version=f'{NAME} v{VERSION}')

    arg_parser.add_argument('-c', '--commands', \
        help=COMMANDS_DESCRIPTION, \
        action='store_true', default=None)

    arg_parser.add_argument('-B', '--benchmark', \
        help=BENCHMARK_DESCRIPTION, \
        action='store_true', default=None)

    arg_parser.add_argument('-d', '--debug', \
        help=DEBUG_DESCRIPTION, \
        action='store_true', default=None)

    arg_parser.add_argument('-b', '--bandwidth', \
        help=f'{BANDWIDTH_DESCRIPTION}; default: {BANDWIDTH}; range: {BANDWIDTH_RANGE}', \
        type=int, metavar='INT')

    arg_parser.add_argument('-t', '--throughput', \
        help=f'{THROUGHPUT_DESCRIPTION}; default: {THROUGHPUT}; range: {THROUGHPUT_RANGE}', \
        type=int, metavar='INT')

    arg_parser.add_argument('-n', '--neos-influence', \
        help=f'{NEOS_INFLUENCE_DESCRIPTION}; default: {NEOS_INFLUENCE}; range: {NEOS_INFLUENCE_RANGE}', \
        type=float, metavar='FLOAT')

    arg_parser.add_argument('-a', '--animation-interval', \
        help=f'{ANIMATION_INTERVAL_DESCRIPTION}; default: {ANIMATION_INTERVAL}; range: {ANIMATION_INTERVAL_RANGE}', \
        type=float, metavar='FLOAT')

    arg_parser.add_argument('-l', '--limiter', \
        help=f'{LIMITER_DESCRIPTION}; default: {LIMITER}; range: {LIMITER_RANGE}', \
        type=float, metavar='FLOAT')

    arg_parser.add_argument('-m', '--stream-size-mult', \
        help=f'{STREAM_SIZE_MULT_DESCRIPTION}; default: {STREAM_SIZE_MULT}; range: {STREAM_SIZE_MULT_RANGE}', \
        type=float, metavar='FLOAT')

    arg_parser.add_argument('-C', '--max-cols', \
        help=f'{MAX_COLS_DESCRIPTION}; default: {MAX_COLS}; range: {MAX_COLS_RANGE}', \
        type=int, metavar='INT')

    arg_parser.add_argument('-R', '--max-rows', \
        help=f'{MAX_ROWS_DESCRIPTION}; default: {MAX_ROWS}; range: {MAX_ROWS_RANGE}', \
        type=int, metavar='INT')

    arg_parser.add_argument('-F', '--max-frames', \
        help=f'{MAX_FRAMES_DESCRIPTION}; default: {MAX_FRAMES}; range: {MAX_FRAMES_RANGE}', \
        type=int, metavar='INT')

    arg_parser.add_argument('-e', '--exit-keys', \
        help=f'{EXIT_KEYS_DESCRIPTION}; default: {EXIT_KEYS}', \
        nargs='+', metavar='KEY')

    arg_parser.add_argument('--use-async', \
        help=f'{USE_ASYNC_DESCRIPTION}; default: {USE_ASYNC}', \
        action='store_true', default=None)

    arg_parser.add_argument('--no-use-async', \
        help=f'{NO_USE_ASYNC_DESCRIPTION}; default: {not USE_ASYNC}', \
        action='store_false', dest='use_async', default=None)

    args = vars(arg_parser.parse_args(argv[1:]))
    args = { k: v for k, v in args.items() if v is not None }

    if args.get('commands'):
        print_commands()
    elif args.get('benchmark'):
        run_benchmark(args)
    else:
        run_animation(args)

if __name__ == '__main__':
    main()

