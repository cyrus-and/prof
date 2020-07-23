# Prof

Self-contained C/C++ profiler library for Linux.

Prof offers a quick way to measure performance events (CPU clock cycles,
cache misses, branch mispredictions, etc.) of C/C++ code snippets. Prof is
just a wrapper around the `perf_event_open` system call, its main goal is to
be easy to setup and painless to use for targeted optimizations, namely, when
the hot spot has already been identified. In no way Prof is a replacement for
a fully-fledged profiler like perf, gprof, callgrind, etc.

Please be aware that Prof uses `__attribute__((constructor))` to be as more
straightforward to setup as possible, so it cannot be included more than
once.

## Examples

### Minimal

The following snippet prints the rough number of CPU clock cycles spent in
executing the code between the two Prof calls:

```c
#include "prof.h"

int main()
{
    PROF_START();
    // slow code goes here...
    PROF_STDOUT();
}
```

### Custom options

The following snippet instead counts both read and write faults of the level
1 data cache that occur in the userland code between the two Prof calls:

```c
#include <stdio.h>

#define PROF_USER_EVENTS_ONLY
#define PROF_EVENT_LIST \
    PROF_EVENT_CACHE(L1D, READ, MISS) \
    PROF_EVENT_CACHE(L1D, WRITE, MISS)
#include "prof.h"

int main()
{
    uint64_t faults[2] = { 0 };

    PROF_START();
    // slow code goes here...
    PROF_DO(faults[index] += counter);

    // fast or uninteresting code goes here...

    PROF_START();
    // slow code goes here...
    PROF_DO(faults[index] += counter);

    printf("Total L1 faults: R = %lu; W = %lu\n", faults[0], faults[1]);
}
```

## Installation

Just include `prof.h`. Here is a quick way to fetch the latest version:

    wget -q https://raw.githubusercontent.com/cyrus-and/prof/master/prof.h

## Setup

Since Prof uses `perf_event_open` make sure to have the permission to access
the performance counters: either run the program as superuser (discouraged)
or set the value of `perf_event_paranoid` appropriately, for example:

```console
$ echo 1 | sudo tee /proc/sys/kernel/perf_event_paranoid
```

Optionally make it permanent with:

```console
$ echo 'kernel.perf_event_paranoid=1' | sudo tee /etc/sysctl.d/local.conf
```

See `man perf_event_open` for more information.

## API

### PROF_START()

Reset the counters and (re)start counting the events.

The events to be monitored are specified by setting the `PROF_EVENT_LIST`
macro before including this file to a list of `PROF_EVENT_*` invocations;
defaults to counting the number CPU clock cycles.

If the `PROF_USER_EVENTS_ONLY` macro is defined before including this file
then kernel and hypervisor events are excluded from the count.

### PROF_EVENT(type, config)

Specify an event to be monitored, `type` and `config` are defined in the
documentation of the `perf_event_open` system call.

### PROF_EVENT_HW(config)

Same as `PROF_EVENT` but for hardware events; prefix `PERF_COUNT_HW_` must be
omitted from `config`.

### PROF_EVENT_SW(config)

Same as `PROF_EVENT` but for software events; prefix `PERF_COUNT_SW_` must be
omitted from `config`.

### PROF_EVENT_CACHE(cache, op, result)

Same as `PROF_EVENT` but for cache events; prefixes `PERF_COUNT_HW_CACHE_`,
`PERF_COUNT_HW_CACHE_OP_` and `PERF_COUNT_HW_CACHE_RESULT_` must be omitted
from `cache`, `op` and `result`, respectively. Again `cache`, `op` and
`result` are defined in the documentation of the `perf_event_open` system
call.

### PROF_STOP()

Stop counting the events. The counter array can then be accessed with
`PROF_COUNTERS`.

### PROF_COUNTERS

Access the counter array. The order of counters is the same of the events
defined in `PROF_EVENT_LIST`. Elements of this array are 64 bit unsigned
integers.

### PROF_DO(block)

Stop counting the events and execute the code provided by `block` for each
event. Within `code`: `index` refers to the event position index in the
counter array defined by `PROF_COUNTERS`; `counter` is the actual value of
the counter. `index` is a 64 bit unsigned integer.

### PROF_CALL(callback)

Same as `PROF_DO` except that `callback` is the name of a *callable* object
(e.g. a function) which, for each event, is be called with the two parameters
`index` and `counter`.

### PROF_FILE(file)

Stop counting the events and write to `file` (a stdio.h `FILE *`) as many
lines as are events in `PROF_EVENT_LIST`. Each line contains `index` and
`counter` (as defined by `PROF_DO`) separated by a tabulation character. If
there is only one event then `index` is omitted.

### PROF_STDOUT()

Same as `PROF_LOG_FILE` except that `file` is `stdout`.

### PROF_STDERR()

Same as `PROF_LOG_FILE` except that `file` is `stderr`.

## License

Copyright (c) 2020 Andrea Cardaci <cyrus.and@gmail.com>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

<!-- autogenerated from prof.h -->
