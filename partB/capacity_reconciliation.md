# Part B — Capacity Reconciliation

## B1. KV-cache capacity

The model has:

- Layers = 28
- KV heads = 8
- Head dimension = 128
- KV precision = FP16 = 2 bytes/value

Each token stores both a K and V vector.

Therefore:

KV bytes/token
= layers × 2 (K and V) × KV heads × head_dim × 2 bytes
= 28 × 2 × 8 × 128 × 2
= 114,688 bytes/token

Therefore:

**KV cache = 114,688 bytes/token = 112 KiB/token.**

For a 4096-token sequence:

114,688 × 4096
= 469,762,048 bytes
= 448 MiB

The GPU has 24 GB of memory and the serving configuration reserves:

24 × 0.92 = 22.08 GB

After the assumed 1.6 GB non-KV runtime overhead:

22.08 - 1.6 = 20.48 GB

Using decimal GB for the available-memory calculation:

20.48 × 10^9 / 469,762,048
≈ 43.6

Therefore the approximate maximum number of complete 4096-token sequences is:

**43 concurrent sequences.**

This is an approximate capacity estimate because actual serving systems allocate
KV cache in blocks and have additional implementation overhead.

### Check against the benchmark log

The long-context sweep uses 3584-token prompts and 512 generated tokens,
so each request reaches the configured 4096-token maximum.

Relevant rows:

| Batch | KV utilization | Preempted sequences |
|---:|---:|---:|
| 24 | 0.93 | 0 |
| 32 | 0.97 | 7 |
| 48 | 0.97 | 23 |

At batch 24, there are no preemptions. At batch 32, preemptions appear,
and at batch 48 they increase substantially.

This is consistent with the model-spec estimate of approximately 43 full
4096-token sequences: increasing the number of simultaneously active
long-context sequences pushes the KV cache toward capacity and causes
scheduler preemption.

The estimate should not be interpreted as an exact scheduler threshold,
because KV allocation is block-based and the reported utilization is a
peak utilization metric rather than a direct count of bytes allocated.

## B2. Long-context throughput anomaly

The long-context sweep uses a 3584-token prompt and 512 generated tokens,
so every request reaches 4096 total tokens.

A naive expectation is that increasing batch size should increase aggregate
throughput because more requests are processed concurrently. The log instead
shows a clear throughput peak followed by a decline:

| Batch | Reported tok/s | TTFT p50 (ms) | ITL p50 (ms) | Preempted seqs | KV utilization |
|---:|---:|---:|---:|---:|---:|
| 16 | 1311.4 | 498.3 | 77.2 | 0 | 0.62 |
| 24 | **1607.4** | 500.5 | 96.07 | 0 | 0.93 |
| 32 | 1384.0 | 636.9 | 101.79 | 7 | 0.97 |
| 48 | 1298.5 | 955.4 | 100.0 | 23 | 0.97 |

The anomaly begins after batch 24.

From batch 16 to batch 24, reported throughput increases:

1607.4 / 1311.4 = 1.226

or approximately **22.6% higher throughput**.

However, from batch 24 to batch 32:

1384.0 - 1607.4 = **-223.4 tok/s**

and from batch 24 to batch 48:

1298.5 - 1607.4 = **-308.9 tok/s**.

At the same time, KV-cache utilization rises from 0.93 at batch 24 to
0.97 at batch 32 and remains at 0.97 at batch 48. Scheduler preemptions
also appear:

- Batch 24: 0 preemptions
- Batch 32: 7 preempted sequences
- Batch 48: 23 preempted sequences

Latency also deteriorates. TTFT p50 rises from 500.5 ms at batch 24 to
955.4 ms at batch 48, approximately:

955.4 / 500.5 = 1.91x.

### Mechanism

The evidence is consistent with KV-cache pressure causing scheduler
preemption at high batch sizes. With 3584-token prompts plus 512 generated
tokens, each request can occupy the full 4096-token model context. Increasing
the batch beyond the sustainable KV-cache capacity pushes utilization to
approximately 97%, after which sequences are preempted.

Preemption adds scheduling/recomputation overhead instead of providing useful
additional concurrency. Therefore increasing batch size beyond the throughput
sweet spot can reduce aggregate throughput while substantially increasing
latency.

### Proposed deployment change

Use **batch 24 as the long-context operating point** rather than increasing
directly to batch 32 or 48.

The measured data predicts that this avoids the observed preemptions:

- Batch 24: 0 preemptions
- Batch 32: 7 preemptions
- Batch 48: 23 preemptions

It also preserves the highest measured reported throughput in this sweep:
**1607.4 tok/s**, compared with 1384.0 tok/s at batch 32 and 1298.5 tok/s
at batch 48.

The deployment should therefore use a conservative maximum batch around 24
for this 3584+512 workload, with production monitoring of KV utilization and
preemptions.

## B3. Misreading of `reported_tok_s`

The report's conclusions that longer prompts provide better throughput and that
batch 48 will deliver approximately 3200 tok/s come from treating
`reported_tok_s` as if it represented generation throughput alone.

For the long-prompt batch-24 row:

- prompt length = 3584
- generation length = 512
- requests = 24
- wall clock = 61.16 s
- reported_tok_s = 1607.4

The harness's reported throughput is consistent with counting both prompt and
generation tokens:

24 × (3584 + 512) / 61.16
= 1607.33 tok/s

This matches the logged **1607.4 tok/s**.

Therefore `reported_tok_s` is not generation-only throughput.

### Honest generation goodput

The actual generated tokens are:

24 × 512 = 12,288 generated tokens.

Dividing by wall-clock time:

12,288 / 61.16
= **200.92 generated tok/s**

This can also be derived independently from the reported throughput:

1607.4 × 61.16
= approximately 98,307 total tokens

Subtracting the prompt tokens:

24 × 3584 = 86,016

leaves approximately 12,293 generated tokens.

Dividing by 61.16 seconds:

12,293 / 61.16
≈ **200.99 generated tok/s**

The small difference from 200.92 is due to rounding in the logged
`reported_tok_s` value.

Thus the honest generation goodput for batch 24 is approximately:

**201 generated tokens/s.**

### What the report should have said

The report should not have concluded that longer prompts inherently provide
better generation throughput. The higher `reported_tok_s` for long prompts is
largely explained by the counter including the large prompt-processing token
count.

Likewise, the batch-48 value of 1298.5 tok/s cannot be interpreted as
approximately 3200 generated tok/s. For the 3584+512 workload, the logged
batch-48 value is a combined prompt-plus-generation throughput counter, and
the row also suffers from 23 preempted sequences.

The report should have separated **prefill/prompt processing throughput** from
**generation/decode throughput**, and should have reported generated
tokens/second when making claims about generation performance.

---

## B4. Metric to confirm the B2 mechanism

The single most useful serving-stack metric to confirm the suspected KV-cache
pressure mechanism is **KV-cache allocation/usage together with scheduler
preemptions**; if one counter must be chosen, use the scheduler's
**preempted-sequence count**.

The prediction is that increasing the long-context batch beyond the sustainable
point causes this counter to rise sharply. The existing log already shows:

- batch 24: **0 preemptions**
- batch 32: **7 preemptions**
- batch 48: **23 preemptions**

Therefore, if the proposed mechanism is correct, a live serving run around
the batch-24 to batch-32 transition should show preemptions appearing as KV
cache utilization approaches its limit. A corresponding increase in
recomputation/scheduler work would further confirm that KV pressure, rather
than simply insufficient raw compute, is responsible for the throughput
collapse.