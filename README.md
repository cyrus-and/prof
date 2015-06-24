Prof
====

Self-contained C/C++ profiler library for Linux.

Prof offers a quick way to measure performance events (CPU clock cycles, cache
misses, branch mispredictions, etc.) of C/C++ code snippets. Prof is just a
quick wrapper around the `perf_event_open` system call, its main goal is to be
easy to setup and painless to use for targeted optimizations, namely, when the
hot spot has already been identified. In no way Prof is a replacement for a
fully-fledged profiler like perf, gprof, callgrind, etc.

Please be aware that Prof uses `__attribute__((constructor))` to be as more
straightforward to setup as possible, so it cannot be included more than once.

Minimal example
---------------

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

See the `API` section in [`prof.h`](prof.h) for the documentation.
