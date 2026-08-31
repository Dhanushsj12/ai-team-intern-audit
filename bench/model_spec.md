# Serving setup for `bench_log.csv`

## Model: FLM-4B-Instruct (dense)

| Property | Value |
|---|---|
| Parameters | 4.2 B |
| Layers | 28 |
| d_model | 3072 |
| Attention heads (Q) | 24 |
| KV heads (GQA) | 8 |
| Head dimension | 128 |
| Vocabulary | 128k |
| Weights precision | fp16 |
| KV cache precision | fp16 |

## Hardware & serving config

| Property | Value |
|---|---|
| GPU | 1× NVIDIA L4 (24 GB) |
| Memory bandwidth (peak) | 300 GB/s |
| FP16 dense compute (peak) | ~121 TFLOPS |
| `max_model_len` | 4096 |
| `gpu_memory_utilization` | 0.92 |
| Non-KV runtime overhead (activations, CUDA graphs, etc.) | assume ~1.6 GB |

## How the load test was run

Each row of `bench_log.csv` is one run: `num_requests` identical requests
submitted simultaneously with the given `prompt_len` and `gen_len`.
All requests generate exactly `gen_len` tokens, with no early stopping.

Column notes:

- `reported_tok_s` — the harness's built-in throughput counter
- `ttft_ms_p50` — median time to first token
- `itl_ms_p50` — median inter-token latency during decode
- `e2e_ms_p95` — p95 end-to-end request latency
- `preempted_seqs` — sequences the scheduler preempted at least once
- `kv_cache_util` — peak KV cache block utilization