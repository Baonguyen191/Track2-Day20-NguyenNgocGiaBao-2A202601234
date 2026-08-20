# 03 - Integrate: RAG pipeline run

Host `Windows-AMD64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 2.8 | 2838.0 | 2840.8 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.0 | 2648.4 | 2648.5 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.0 | 2619.2 | 2619.3 |

Mean per stage (ms): embed **0.0** · retrieve **0.9** ·
llm **2701.9** · total **2702.9**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Goodput@SLO counts only the requests per second that met the TTFT and TPOT targets. Throughput at saturation ignores SLOs.

**What problem does PagedAttention actually solve?**

> PagedAttention stores the KV cache in non-contiguous pages, which removes the internal fragmentation that wasted most GPU memory.

**When does splitting prefill and decode help?**

> Splitting prefill and decode helps because prefill is compute-bound and decode is memory-bandwidth-bound.


## Which N16-N19 pieces are real

- N16 Cloud/IaC: stubbed (running locally)
- N17 Data pipeline: stubbed
- N18 Lakehouse: stubbed
- N19 Vector + features: stubbed (using keyword overlap)
- N20 Serving: real (`llama-server`)

The dominant stage is `llm` (100% of latency), which is expected since retrieval is a simple local string match and LLM generation is computationally heavy. To halve the latency, I would attack the `llm` stage by using a smaller model, lower quantization, or increasing batch size for better throughput.

_List each of N16, N17, N18, N19 as real or stubbed. Stubbing costs no points;
misrepresenting it does. Then answer: is the dominant stage above what you expected?
If you had to halve this pipeline's latency, which stage would you attack and why?_
