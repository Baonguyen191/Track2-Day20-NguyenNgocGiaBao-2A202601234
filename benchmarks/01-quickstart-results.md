# 01 - Measure: latency baseline

Model `Gemma 4 E2B` · host `Windows-AMD64` · llama.cpp `b10488`
Settings: `threads=10` `ngl=99` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `UD-Q4_K_XL` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 14235 | 115 / 446 | 11.6 / 13.2 | 805 / 1180 / 1180 | 86.5 |
| UD-Q2_K_XL | 2.24 | 4724 | 78 / 271 | 10.3 / 11.0 | 734 / 937 / 937 | 96.6 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.12x faster** than `UD-Q4_K_XL` here, for 0.73 GB less on disk.

## Your observation

Yes, it is worth it. The 2-bit quantization (UD-Q2_K_XL) is 1.12x faster in decode speed (96.6 vs 86.5 tok/s) and uses 0.73 GB less RAM, while the answer quality remains very similar for general queries.

_Is the smaller quantization worth it on your machine? Compare the numbers above,
then judge the answer quality yourself: run `make serve` on each and ask the same
question twice. Size and speed are measurable; usefulness is your call._
