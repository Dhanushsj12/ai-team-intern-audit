# \# Tokenizer \& Serving Findings (v0) — for the leadership deck

#

# \*Status: revised after audit. Numbers are based on the available corpus and

# benchmark evidence; production conclusions should be validated on

# representative traffic.\*

#

# \## 1. Tokenizer fertility

#

# The original `fertility.py` analysis used the GPT-2 tokenizer and measured

# tokens per whitespace-separated word. The script averages the per-line

# fertility values rather than calculating total tokens divided by total words

# over the entire corpus.

#

# Using the same GPT-2 tokenizer and corpus files, the corpus-level results are:

#

# | lang | lines | total tokens | total words | corpus tok/word |

# |---|---:|---:|---:|---:|

# | eng | 997 | 26,696 | 20,954 | 1.274 |

# | hin | 997 | 191,842 | 24,607 | 7.796 |

# | kan | 997 | 349,802 | 15,430 | 22.670 |

# | tam | 997 | 397,189 | 16,134 | 24.618 |

#

# The original GPT-2 result therefore shows a large tokenization difference

# between English and the Indic languages in this corpus.

#

# However, this should not be interpreted directly as a production serving-cost

# multiplier. The denominator is whitespace-separated words, which is not a

# consistent unit of linguistic content across languages and scripts.

#

# I also checked the result with XLM-RoBERTa on the larger FLORES-derived

# development data:

#

# | Language | GPT-2 tok/word | XLM-R tok/word |

# |---|---:|---:|

# | English | 1.28 | 1.42 |

# | Hindi | 7.83 | 1.50 |

# | Kannada | 22.95 | 2.60 |

# | Tamil | 24.87 | 2.45 |

#

# The tokenizer choice substantially changes the observed language gap. With

# XLM-RoBERTa, the ratios relative to English are approximately:

#

# \- Hindi: 1.06×

# \- Kannada: 1.83×

# \- Tamil: 1.73×

#

# This means the large GPT-2 difference should be treated as evidence about the

# specific tokenizer, not as a universal property of the languages.

#

# \### Routing and cost implication

#

# I would not route Indic traffic to a different model solely because of the

# GPT-2 fertility result.

#

# For production cost and capacity planning, the primary measurement should be

# the actual model tokens processed per request, with prompt and generated

# tokens tracked separately when possible.

#

# For cross-language evaluation, the benchmark should use equivalent parallel

# requests or otherwise hold the underlying task and information content

# approximately constant. Tokens per whitespace-separated word or tokens per

# grapheme can be useful diagnostic metrics, but neither should be treated as

# the final production cost number.

#

# The tokenizer result should therefore inform further testing rather than

# determine a routing policy by itself.

#

# \## 2. Serving throughput and capacity

#

# The serving benchmark uses:

#

# \- FLM-4B-Instruct (4.2B parameters)

# \- 1× NVIDIA L4 with 24 GB VRAM

# \- fp16 weights

# \- fp16 KV cache

# \- `max\_model\_len=4096`

# \- `gpu\_memory\_utilization=0.92`

# \- approximately 1.6 GB non-KV runtime overhead

#

# The benchmark shows that throughput and latency depend on request shape and

# batch size. The results should therefore not be summarized as a single

# throughput number that scales linearly with batch size.

#

# In particular, higher throughput in a particular benchmark row does not by

# itself mean that clients should send longer prompts. Longer prompts increase

# token processing and can increase KV-cache pressure and latency.

#

# \### Capacity implication

#

# The usable GPU memory under the configured utilization is approximately:

#

# 22.08 GB = 24 GB × 0.92

#

# The fp16 model weights require approximately:

#

# 7.8 GB ≈ 4.2B parameters × 2 bytes

#

# After accounting for the approximately 1.6 GB non-KV runtime overhead, the

# remaining memory available for KV cache is approximately:

#

# 22.08 − 7.8 − 1.6 = 12.68 GB

#

# Using the model's GQA configuration and fp16 KV cache, this gives an

# approximate KV-cache capacity of about 29 full-length 4096-token sequences.

#

# This is consistent with the benchmark behavior: batches below that region can

# avoid preemption, while a batch of 32 can exceed the available KV capacity and

# show preemptions.

#

# The benchmark therefore supports a capacity interpretation based on KV-cache

# pressure rather than a simple linear throughput model.

#

# \### Throughput interpretation

#

# The harness's `reported\_tok\_s` should not be treated blindly as honest

# end-to-end goodput. Throughput should be reconciled against prompt tokens,

# generated tokens, request count, and wall-clock time.

#

# For the benchmark, a useful cross-check is:

#

# `reported\_tok\_s ≈ (prompt\_tokens + generated\_tokens) × requests / wall\_clock`

#

# The benchmark evidence should be used to identify safe operating regions,

# preemption behavior, latency trade-offs, and realistic capacity rather than

# assuming that the best observed throughput will scale linearly to larger

# batches.

#

# \## 3. Overall recommendation

#

# The tokenizer analysis supports further measurement rather than an immediate

# language-specific routing decision.

#

# The serving analysis supports capacity planning around model weights,

# non-KV memory, KV-cache pressure, preemption, and request shape.

#

# Before making production routing or cost decisions, I would evaluate

# candidate tokenizer/model combinations and serving configurations using

# traffic representative of the intended workload and measure:

#

# 1\. actual prompt tokens per request;

# 2\. actual generated tokens per request;

# 3\. end-to-end latency;

# 4\. throughput/goodput;

# 5\. KV-cache utilization and preemptions; and

# 6\. quality on each target language.

#

# These measurements provide a stronger basis for production routing and cost

# decisions than the original fertility or best-row throughput claims alone.

