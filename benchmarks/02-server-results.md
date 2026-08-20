# 02 - Serve: load test + saturation reading

Host `Windows-AMD64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=10` ·
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 62 | 1.87 | 4100 | 4100 | 4100 | 7.6 | 100.0% |
| 50 | 198 | 3.34 | 13000 | 15000 | 16000 | 41.9 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **1.79x** (36% of linear) |
| P95 latency | **3.66x** |
| Effective concurrency at 50 users | 41.9 vs `--parallel 4` slots (occupancy/slot ratio 10.47) |

**Saturated.** Throughput delivered only 1.79x for 5x the offered load, and effective concurrency (41.9) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 1.79x while P95 moved 3.66x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

## Your reading

The server saturates between 10 and 50 users. The evidence is that offered load increased 5x (10 -> 50), but throughput only increased 1.79x (1.87 RPS -> 3.34 RPS). At 50 users, the effective concurrency is 41.9, massively exceeding the 4 available decode slots, causing P95 latency to jump 3.66x (to 15000ms) mostly due to queue time.
To raise goodput at a fixed SLO, I would first increase `--parallel` (batch size) if VRAM allows, to process more requests concurrently, converting queue time into compute time.

_Where does your server saturate, and what is the evidence? Name the number that
convinced you. Then say what you would change first to raise goodput at your SLO --
and why that knob and not another._
