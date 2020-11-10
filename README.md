# enterthematrix

controllable matrix rain animation and benchmarking tool

<img src="https://enterthematrix.space/img/demo-0.gif" alt="demo gif">

## Documentation

### Dependencies

* python3
* Font with Japanese Katakana support (unicode `0xff66`-`0xff9d`)

### Installation

```sh
sudo curl https://raw.githubusercontent.com/stefrush/enterthematrix/master/enterthematrix -o /usr/local/bin/enterthematrix
sudo chmod a+rx /usr/local/bin/enterthematrix
enterthematrix --version
```

### Running Animation

```sh
enterthematrix
```

### Running Benchmark

```sh
enterthematrix -B
```

The standard benchmark as requested by Neo is 4096 rendered frames with the following config:

* 16 bandwidth
* 16 throughput
* 80 columns
* 40 rows
* No animation interval
* Async frame rendering off
* Neo's influence at 4%

This means a standard benchmark requires a terminal display with 80x40 text cells in view to achieve maximum stress.

The amount of time elapsed during the benchmark will print once every frame has been rendered.

### Key Commands

`d` => `toggle_debug`

`p` => `toggle_paused`

`s` => `trigger_step`

`c` => `clear_screen`

`b` => `increase_bandwidth`

`B` => `decrease_bandwidth`

`t` => `increase_throughput`

`T` => `decrease_throughput`

`n` => `increase_neos_influence`

`N` => `decrease_neos_influence`

`a` => `increase_animation_interval`

`A` => `decrease_animation_interval`

`l` => `increase_limiter`

`L` => `decrease_limiter`

`m` => `increase_stream_size_mult`

`M` => `decrease_stream_size_mult`

_Commands are disabled during benchmarks._

### Usage

```
enterthematrix [-h] [-v] [-c] [-B] [-d] [-b INT] [-t INT] [-n FLOAT] [-a FLOAT] [-l FLOAT] [-m FLOAT] [-C INT] [-R INT] [-F INT] [-e KEY [KEY ...]] [--use-async] [--no-use-async]


optional arguments:
  -h, --help            show this help message and exit
  -v, --version         show program's version number and exit
  -c, --commands        show the list of key commands and exit
  -B, --benchmark       run the standard enterthematrix benchmark
  -d, --debug           show program runtime information during the animation
  -b INT, --bandwidth INT
                        set the maximum number of character animation streams per text column; default: 2; range: (0, inf)
  -t INT, --throughput INT
                        set the number of character animation streams to create per animation frame; default: 4; range: (0, inf)
  -n FLOAT, --neos-influence FLOAT
                        follow the white rabbit; default: 0.004; range: (0, 1)
  -a FLOAT, --animation-interval FLOAT
                        set the amount of time in seconds between animation frames; default: 0.042; range: (0, inf)
  -l FLOAT, --limiter FLOAT
                        limit the maximum number of streams by a factor of the limiter value; default: 0; range: (0, 1)
  -m FLOAT, --stream-size-mult FLOAT
                        set the average stream size to the number of rows times the multiplier value; default: 0.8; range: (0, inf)
  -C INT, --max-cols INT
                        set the maximum number of text columns to animate; default: 1280; range: (1, inf)
  -R INT, --max-rows INT
                        set the maximum number of text rows to animate; default: inf; range: (1, inf)
  -F INT, --max-frames INT
                        set the maximum number of frames to animate; default: inf; range: (1, inf)
  -e KEY [KEY ...], --exit-keys KEY [KEY ...]
                        set the list of keys to initiate exit
  --use-async           turn on async frame rendering in supported environments; default: True
  --no-use-async        turn off async frame rendering in supported environments; default: False
```

### License

[MIT](https://github.com/stefrush/enterthematrix/blob/master/LICENSE)

