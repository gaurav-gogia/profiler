# Sampling Rate Choice: Scientific Basis and Practical Sweet Spot

## Why a sampling rate is scientifically justified

A fixed sampling interval is not a workaround; it is the standard way to
estimate time-varying process behavior under bounded measurement overhead.

1. Sampling is a signal-measurement problem.

- CPU%, RSS, and thread count are time-series signals.
- If sampling is too slow, short-lived events (bursts/spikes) are missed
  (aliasing risk).
- If sampling is too fast, the measurement process perturbs the system (observer
  effect / measurement bias).

2. There is an unavoidable bias-variance-overhead tradeoff.

- Lower interval (faster sampling): better temporal resolution, higher
  overhead/noise.
- Higher interval (slower sampling): lower overhead, worse peak detection.
- Good profiling practice is to choose the lowest interval that keeps overhead
  acceptably small and stable.

3. Termination detection should be decoupled from sampling.

- Process completion should be captured by OS notification (`wait` semantics),
  not by polling cadence.
- Sampling rate should control only metric granularity, not experiment stop-time
  accuracy.

## What this means for this profiler specifically

Your profiler reads CPU, memory, threads, and I/O counters via psutil in a loop.
For this type of Python userspace monitor:

- CPU measurements are interval-based estimates (not truly instantaneous).
- Memory/thread metrics are snapshots and can miss short spikes if interval is
  too large.
- Read/write byte counters are cumulative, so final totals are robust even with
  slower sampling (you mainly lose short-window rate detail).

## Practical sweet spot (recommended default)

For general process profiling on modern desktop/server systems:

- Recommended default: **0.05 s to 0.10 s** (50-100 ms)
- Use **0.02 s to 0.05 s** only when you explicitly care about very short spikes
  and have confirmed low profiler overhead.
- Avoid **0.00 s** in production measurements; near-busy-loop sampling can
  materially perturb CPU and scheduler behavior.

A robust default for your script is **0.10 s**. If peak fidelity matters (short
bursts), use **0.05 s** after validating overhead.

## Reproducible calibration protocol (scientific workflow)

Use this once per machine/workload class:

1. Run a baseline with coarse sampling (e.g., 0.25 s).
2. Repeat at 0.10 s, 0.05 s, 0.02 s.
3. For each interval, run at least 10 repetitions.
4. Compare:

- Median runtime and its variability.
- Peak RSS and high percentiles (p95/p99) of CPU.
- Consistency of conclusions (ranking between program variants).

5. Choose the smallest interval where:

- Runtime inflation from profiling is small (commonly <= 1-3%).
- Key metrics stabilize (further interval reductions change peaks/percentiles
  only marginally).

This is the same philosophy used in empirical systems research: minimize
instrumentation bias while preserving decision-relevant signal.

## Why this is better than poll-based completion

Polling completion with sleep ties end-time error to sampling interval. Using OS
wait + separate monitor thread gives:

- More accurate process end detection.
- Better-defined execution-time measurement.
- Independent control of measurement granularity.

## References (papers, standards, and docs)

1. C. E. Shannon, "Communication in the Presence of Noise," Proceedings of the
   IRE, 1949. DOI: https://doi.org/10.1109/JRPROC.1949.232969
2. T. Mytkowicz, A. Diwan, M. Hauswirth, P. F. Sweeney, "Producing Wrong Data
   Without Doing Anything Obviously Wrong!," ASPLOS 2009. DOI:
   https://doi.org/10.1145/1508244.1508275
3. Python `time` module docs (`perf_counter`, `sleep` semantics and scheduling
   caveats): https://docs.python.org/3/library/time.html
4. Linux man page `/proc/pid/io` (definition and caveats for cumulative I/O
   counters): https://man7.org/linux/man-pages/man5/proc_pid_io.5.html
5. psutil documentation (process metrics API and performance guidance such as
   `oneshot`): https://psutil.readthedocs.io/
6. Nyquist-Shannon sampling theorem background (sampling/aliasing fundamentals):
   https://en.wikipedia.org/wiki/Nyquist%E2%80%93Shannon_sampling_theorem
7. C. Millsap, "Thinking Clearly about Performance," ACM Queue, 2010
   (measurement and profiling methodology context):
   https://queue.acm.org/detail.cfm?id=1854041

## Suggested policy for this repository

- Default sampling rate: **0.10 s**
- High-fidelity mode: **0.05 s**
- Minimum allowed in normal runs: **0.01 s**
- Record interval in every experiment row (already implemented)
- Recalibrate if hardware, OS, Python version, or workload type changes
  materially
