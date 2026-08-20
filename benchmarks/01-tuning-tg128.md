# 01 - Tune: thread-count sweep

Model `gemma-4-E2B-it-UD-Q4_K_XL.gguf` · host `Windows-AMD64` · llama.cpp `b10488`
CPU: **10 physical · 16 logical** cores · `ngl=99` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 98.4 | 100% |
| 5 | 97.3 | 99% |
| 10 | 98.0 | 100% |
| 16 | 95.9 | 97% |
| 32 | 93.6 | 95% |

**Best**: `-t 1` at 98.4 tok/s
**Slowest tested**: `-t 32` at 93.6 tok/s (1.05x spread)
**Against the physical-core default** (`-t 10`, 98.0 tok/s): 1.00x

Use this in your run:

```bash
LAB_N_THREADS=1 make bench
```

## Your explanation

The knee sits at `-t 10` (the physical core count), with 98.0 tok/s. Threads above 10 result in a performance drop because the extra logical threads compete for the same physical execution units and memory bandwidth without adding real compute power. `tg128` (decode) is heavily memory-bandwidth bound, so adding more threads just adds scheduling overhead. The absolute peak at `-t 1` is interesting and indicates that a single core is already saturating the memory bandwidth for decode on this machine.

_Where is the knee, and why there? If the peak sits at your physical core count
and drops above it, say what the extra threads are competing for. If your curve
does something else -- flat, or still climbing at 2x logical cores -- say that
instead and reason about why. A result that contradicts the expected shape is
worth more than one that matches it, as long as you explain it._
