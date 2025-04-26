# benchmark-python-script-performance

## After-run stats:
### Determine which functions are taking the longest time with Built-in Deterministic Profiling

1.	cProfile + pstats
•	Instrument entire script:

python -m cProfile -o profile.out your_script.py


•	Inspect results in an interactive shell:

import pstats
stats = pstats.Stats('profile.out')
stats.strip_dirs().sort_stats('cumtime').print_stats(20)


•	Pros: No code changes; function-level breakdown.
•	Cons: Overhead can slow your program; no line-level detail.

### Determine which line is taking the longest with line_profiler
•	Install and add the @profile decorator to functions you want to drill into:

from line_profiler import LineProfiler

lp = LineProfiler()
lp.add_function(my_heavy_function)
lp_wrapper = lp(my_heavy_function)
lp_wrapper(...)
lp.print_stats()


	•	Or use the kernprof script:

pip install line_profiler
kernprof -l -v your_script.py


	•	Pros: Exact time per source line.
	•	Cons: You must decorate or kernprof-wrap specific functions.

## Real time monitoring

2. Sampling Profilers (Low-overhead, Real-time)
	1.	Py-Spy

# Live “top” view:
py-spy top --pid <your-process-pid>
# Record to a flamegraph:
py-spy record -o profile.svg --pid <pid>

	•	Pros: Works without modifying code, very low overhead, great for production.
	•	Cons: Sampling means it may miss very short-lived functions.

	2.	Scalene

pip install scalene
scalene your_script.py

	•	Provides CPU-, memory-, and GPU-profiling in one report, with per-line breakdown.
	•	Pros: Distinguishes Python vs native time, shows memory hotspots.
	•	Cons: Requires an extra dependency; report format may take a minute to learn.

⸻

3. Visualization Tools
	1.	SnakeViz

python -m cProfile -o profile.out your_script.py
snakeviz profile.out

	•	Spins up a web UI to explore call graphs and hot paths.

	2.	gprof2dot + Graphviz

python -m cProfile -o profile.out your_script.py
gprof2dot -f pstats profile.out | dot -Tpng -o callgraph.png

	•	Renders call graph images showing call counts and times.

⸻

4. Manual Instrumentation
	1.	Time-stamped logging / decorators

import time, functools

def timeit(fn):
    @functools.wraps(fn)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = fn(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{fn.__name__!r} took {elapsed:.3f}s")
        return result
    return wrapper

@timeit
def my_heavy_func(...):
    ...


	2.	Context managers

from contextlib import contextmanager
import time

@contextmanager
def log_block(name):
    start = time.perf_counter()
    yield
    print(f"[{name}] {time.perf_counter() - start:.3f}s")

# usage
with log_block("data load"):
    load_data()



⸻

5. System-Level Monitoring
	•	CPU / I/O usage:
	•	top / htop to see overall CPU load (process vs threads).
	•	iotop to detect slow disk I/O.
	•	strace -T -p <pid> to see syscalls and their timing.
	•	Memory profiling (if you suspect memory overhead):
	•	tracemalloc built-in module.
	•	memory_profiler (mprof run your_script.py; mprof plot).

⸻

Recommended Workflow
	1.	Start coarse with cProfile (or py-spy top) to find “big” hot functions.
	2.	Zoom in with line_profiler or Scalene on the worst offenders.
	3.	Visualize with SnakeViz or flamegraphs for patterns you might miss in tables.
	4.	Iterate: refactor, then re-profile to verify gains.

⸻

Let me know if you’d like a deeper dive on any of these tools—code examples, installation tips, or how to integrate into CI pipelines!
