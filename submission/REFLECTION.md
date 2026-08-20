# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** Nguyễn Ngọc Gia Bảo
**Cohort:** A20-K1
**Ngày submit:** 2026-08-20

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** Windows 11
- **CPU:** 12th Gen Intel(R) Core(TM) i7-12650H
- **Cores:** 10 physical / 16 logical
- **CPU extensions:** AVX2
- **RAM:** 15.7 GB
- **Accelerator:** NVIDIA GeForce RTX 4060 Laptop GPU, 8188 MiB
- **llama.cpp asset đã tải:** b10488
- **Model đã dùng:** Gemma 4 E2B (`LAB_MODEL=gemma4-e2b`)
- **Quantization:** UD-Q4_K_XL + UD-Q2_K_XL (từ `models/active.json`)

**Chạy ở đâu:** laptop của tôi
_(Nếu dùng cloud fallback: nói rõ vì sao — RAM < 8 GB, setup fail, v.v. Không mất điểm.)_

**Setup story** (≤ 80 chữ): Bài lab chạy khá mượt mà trên máy local. Đã gặp lỗi mã hoá file `lab.ps1` ở bản PowerShell 5.1 do chứa ký tự em-dash không có BOM, nhưng đã xử lý nhanh chóng bằng cách đổi thành dấu gạch ngang chuẩn.

---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|---|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 14235 | 115 / 446 | 11.6 / 13.2 | 805 / 1180 / 1180 | 86.5 |
| UD-Q2_K_XL | 2.24 | 4724 | 78 / 271 | 10.3 / 11.0 | 734 / 937 / 937 | 96.6 |

**Quan sát** (≤ 60 chữ): Bản 2-bit (Q2) decode nhanh hơn khoảng 1.12x và tiết kiệm 0.73 GB RAM. Mức đánh đổi này là hoàn toàn xứng đáng vì chất lượng câu trả lời thử nghiệm thực tế không chênh lệch đáng kể đối với các tác vụ thông thường.

---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 1.87 | 4100 | 4100 | 4100 | 7.6 | 100.0% |
| 50 | 3.34 | 13000 | 15000 | 16000 | 41.9 | 0.0% |

- **Offered load tăng 5×, throughput thực tăng:** 1.79×
- **P95 tăng:** 3.66×
- **Effective concurrency ở 50 users:** 41.9 so với `--parallel` = 4 slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): 4.00 / 4 slots

**Saturation reading** (≤ 80 chữ): Server bão hoà ở mức dưới 50 users. Bằng chứng là số RPS chỉ tăng nhẹ (1.79x) nhưng P95 vọt lên rất cao (3.66x). Sự chênh lệch này thể hiện latency tăng lên phần lớn là do queue time. Để tăng goodput@SLO, tôi sẽ ưu tiên tăng `--parallel` (nếu VRAM cho phép) để tận dụng batching tốt hơn.

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | stubbed | stubbed |
| N17 Data pipeline | stubbed | stubbed |
| N18 Lakehouse | stubbed | stubbed |
| N19 Vector + features | stubbed | stubbed |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: 0.0 ms
- retrieve: 0.9 ms
- llm: 2701.9 ms
- **stage chiếm nhiều nhất:** llm (100% của total)

**Reflection** (≤ 60 chữ): Bottleneck nằm hoàn toàn ở bước gọi LLM, hoàn toàn đúng với kỳ vọng do retrieve chỉ là tìm chuỗi cục bộ đơn giản. Nếu cần giảm latency 2x, tôi sẽ tập trung tối ưu LLM: dùng model nhỏ hơn hoặc giảm quantization.

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** Giảm số luồng `-t` từ 16 xuống 10 (bằng đúng số physical cores).

```
before:  95.9 tok/s
after:   98.0 tok/s
speedup: 1.02×
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

Giai đoạn decode của LLM (sinh từng token) là một tác vụ memory-bandwidth bound (giới hạn bởi băng thông bộ nhớ), không phải compute bound. Khi ta tăng số luồng (threads) vượt qua số lượng physical cores (từ 10 lên 16), các logical threads bổ sung không hề tăng thêm sức mạnh tính toán thực tế mà chỉ tranh giành chung một lượng băng thông bộ nhớ và tài nguyên execution.

Sự tranh chấp tài nguyên này tạo ra scheduling overhead, khiến CPU phải tốn thời gian context switch thay vì thực sự di chuyển dữ liệu. Do đó, việc giới hạn số luồng bằng đúng số lượng core vật lý (10 cores) giúp băng thông bộ nhớ được luân chuyển mượt mà nhất, tối ưu hoá TPOT và mang lại tok/s cao hơn. Đáng ngạc nhiên là ở -t 1 tốc độ lại rất cao (98.4 tok/s), cho thấy chỉ cần 1 luồng cũng đã đủ bão hoà băng thông memory trên máy này.

---

## 6. Bonus  *(optional — tối đa 20 điểm)*

> Bỏ trống nếu không làm. Xem `bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** 

**Numbers:**

```
before:  
after:   
speedup: 
```

**Điều này nói lên gì mà deck chưa nói:**

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

Tốc độ decode đạt đỉnh ở cấu hình 1 thread (-t 1), phản ánh rõ ràng tính chất nghẽn băng thông bộ nhớ (memory-bandwidth bound) hơn là compute bound trong pha decode.

---

## 8. Self-check trước khi push

- [x] `hardware.json` committed
- [x] `models/active.json` committed
- [x] `benchmarks/01-quickstart-results.md` committed (`make bench`)
- [x] `benchmarks/01-tuning-tg128.md` committed (`make tune`)
- [x] `benchmarks/02-server-results.md` committed (`make load-report`)
- [x] `benchmarks/02-server-batching-u50.md` hoặc `-metrics-u50.csv` committed (`make metrics`)
- [x] `benchmarks/locust-10_stats.csv` + `locust-50_stats.csv` committed (`make load-10` / `load-50`)
- [x] `benchmarks/03-integration-results.md` committed (`make pipeline`)
- [x] Mọi section **"required — replace this line"** trong các file `benchmarks/*.md`
      đã được thay bằng nhận xét của bạn
- [x] 5 screenshots trong `submission/screenshots/`
- [x] `make verify` → **exit 0**
- [x] Repo GitHub ở chế độ **public**
- [x] Đã paste public URL vào VinUni LMS
- [x] **Không** commit `models/*.gguf` hay `runtime/` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.
