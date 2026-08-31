@'
# Tokenizer & Serving Findings (v0)

*Status: revised after audit. Numbers are based on the available corpus and benchmark evidence; production conclusions should be validated on representative traffic.*

## 1. Tokenizer Fertility

The original `fertility.py` analysis used the GPT-2 tokenizer and measured tokens per whitespace-separated word. The script averages the per-line fertility values rather than calculating total tokens divided by total words over the entire corpus.

Using the same GPT-2 tokenizer and corpus files, the corpus-level results are:

| Language | Lines | Total Tokens | Total Words | Corpus Tok/Word |
|---|---:|---:|---:|---:|
| English | 997 | 26,696 | 20,954 | 1.274 |
| Hindi | 997 | 191,842 | 24,607 | 7.796 |
| Kannada | 997 | 349,802 | 15,430 | 22.670 |
| Tamil | 997 | 397,189 | 16,134 | 24.618 |

The original GPT-2 result therefore shows a large tokenization difference between English and the Indic languages in this corpus.

However, this should not be interpreted directly as a production serving-cost multiplier. The denominator is whitespace-separated words, which is not a consistent unit of linguistic content across languages and scripts.

I also checked the result with XLM-RoBERTa on the larger FLORES-derived development data:

| Language | GPT-2 Tok/Word | XLM-R Tok/Word |
|---|---:|---:|
| English | 1.28 | 1.42 |
| Hindi | 7.83 | 1.50 |
| Kannada | 22.95 | 2.60 |
| Tamil | 24.87 | 2.45 |

The tokenizer choice substantially changes the observed language gap. With XLM-RoBERTa, the ratios relative to English are approximately:

- Hindi: 1.06x
- Kannada: 1.83x
- Tamil: 1.73x

This means the large GPT-2 difference should be treated as evidence about the specific tokenizer, not as a universal property of the languages.

### Routing and Cost Implication

I would not route Indic traffic to a different model solely because of the GPT-2 fertility result.

For production cost and capacity planning, the primary measurement should be the actual model tokens processed per request, with prompt and generated tokens tracked separately when possible.

For cross-language evaluation, the benchmark should use equivalent parallel requests or otherwise hold the underlying task and information content approximately constant. Tokens per whitespace-separated word or tokens per grapheme can be useful diagnostic metrics, but neither should be treated as the final production cost number.

The tokenizer result should therefore inform further testing rather than determine a routing policy by itself.

## 2. Serving Throughput and Capacity

The serving benchmark uses:

- FLM-4B-Instruct (4.2B parameters)
- 1x NVIDIA L4 with 24 GB VRAM
- fp16 weights
- fp16 KV cache
- `max_model_len=4096`
- `gpu_memory_utilization=0.92`
- approximately 1.6 GB non-KV runtime overhead

The benchmark shows that throughput and latency depend on request shape and batch size. The results should therefore not be summarized as a single throughput number that scales linearly with batch size.

In particular, higher throughput in a particular benchmark row does not by itself mean that clients should send longer prompts. Longer prompts increase token processing and can increase KV-cache pressure and latency.

### Capacity Implication

The usable GPU memory under the configured utilization is approximately:

```text
22.08 GB = 24 GB x 0.92