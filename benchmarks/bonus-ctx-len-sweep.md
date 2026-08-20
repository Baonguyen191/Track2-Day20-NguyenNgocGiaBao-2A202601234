# Bonus - Context-length sweep (prefill cost)

Host `Windows-AMD64` · llama.cpp `b10488` ·
`threads=10` `ngl=99` · RAM 15.7 GB

| Prompt tokens | Prefill (tok/s) | TTFT contribution (ms) | vs linear scaling |
|:--|--:|--:|--:|
| 256 | 2946.1 | 86.9 | 1.00x |
| 1024 | 2533.9 | 404.1 | 1.16x |
| 2048 | 3722.4 | 550.2 | 0.79x |
| 4096 | 4629.4 | 884.8 | 0.64x |
| 8192 | 4308.5 | 1901.4 | 0.68x |

At 8192 tokens, prefill costs **1901 ms** --
0.68x what linear scaling from the smallest point would predict. That excess
is attention's O(N^2) term becoming visible, and every millisecond of it lands in TTFT
before the user sees a single token.

Either way, this is the number to remember when someone proposes stuffing more retrieved
context into a RAG prompt "because the context window allows it". Prefill is paid in full,
on every request, before the first token appears.

## Your finding (required -- replace this line)

_At what prompt length does prefill start to dominate your end-to-end latency? Did you see
the quadratic bend, or is your range still linear -- and what does that tell you about how
many retrieved chunks your RAG pipeline can afford?_
