# enterthematrix

a classic matrix rain animation and benchmark tool

<img src="https://enterthematrix.space/img/demo-0.gif" alt="demo gif">

## Documentation

### Dependencies

* python3
* font with Japanese Katakana support (unicode `0xff66`-`0xff9d`)

### Installation

```sh
curl https://raw.githubusercontent.com/stefrush/enterthematrix/master/enterthematrix -o /usr/local/bin/enterthematrix
chmod +x /usr/local/bin/enterthematrix
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

### Key Commands

```
d => toggle_debug
p => toggle_paused
s => trigger_step
c => clear_screen
b => increase_bandwidth
B => decrease_bandwidth
t => increase_throughput
T => decrease_throughput
n => increase_neos_influence
N => decrease_neos_influence
a => increase_animation_interval
A => decrease_animation_interval
l => increase_limiter
L => decrease_limiter
```

_commands are disabled during benchmarks_

### Usage

```
enterthematrix [-h] [-v] [-c] [-B] [-d] [-b INT] [-t INT] [-n FLOAT] [-a FLOAT] [-l FLOAT] [-m INT] [-e KEY [KEY ...]] [--use-async] [--no-use-async]

optional arguments:
  -h, --help            show this help message and exit
  -v, --version         show program's version number and exit
  -c, --commands        show a list of key commands and exit (default: False)
  -B, --benchmark       run the standard enterthematrix benchmark (default: False)
  -d, --debug           show program runtime information during the animation (default: False)
  -b INT, --bandwidth INT
                        set the maximum number of character animation streams per text column [0-inf) (default: 2)
  -t INT, --throughput INT
                        set the number of character animation streams to create per animation frame [0-inf) (default: 4)
  -n FLOAT, --neos-influence FLOAT
                        follow the white rabbit [0-1] (default: 0.004)
  -a FLOAT, --animation-interval FLOAT
                        set the amount of time in seconds between animation frames [0-inf) (default: 0.042)
  -l FLOAT, --limiter FLOAT
                        limit the maximum number of streams by a factor of the limiter value [0-1] (default: 0)
  -m INT, --max-cols INT
                        set the maximum number of text columns to animate [1-inf) (default: 1280)
  -e KEY [KEY ...], --exit-keys KEY [KEY ...]
                        set the keys to initiate exit; should be a space separated list eg. "e E" (default: ('q', 'Q', '\x1b'))
  --use-async           turn on async frame rendering in supported environments (default: True)
  --no-use-async        turn off async frame rendering in supported environments (default: False)
```

### License

[MIT](https://github.com/stefrush/enterthematrix/blob/master/LICENSE)

