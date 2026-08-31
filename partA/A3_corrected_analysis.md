# A3 — Corrected Analysis

## What I checked

The original result compared the languages using GPT-2 and tokens per
whitespace-separated word.

I repeated the test on the larger FLORES-derived corpus and also checked the
results using XLM-RoBERTa. I looked at tokens per word and tokens per grapheme.

## Corpus

I used the FLORES-derived development data for:

- English
- Hindi
- Kannada
- Tamil

There are 1,012 lines for each language in the selected development data.

The files were read as UTF-8 text. Empty lines were skipped and the text was
NFC-normalized. The fertility script also converts the text to lowercase before
tokenization.

## GPT-2 and XLM-R results

The GPT-2 results were:

| Language | GPT-2 tok/word |
|---|---:|
| English | 1.28 |
| Hindi | 7.83 |
| Kannada | 22.95 |
| Tamil | 24.87 |

I then ran the same corpus with XLM-RoBERTa:

| Language | XLM-R tok/word |
|---|---:|
| English | 1.42 |
| Hindi | 1.50 |
| Kannada | 2.60 |
| Tamil | 2.45 |

The results changed substantially when the tokenizer was changed. With GPT-2,
the Indic-language values were much higher. With XLM-RoBERTa, the difference
was much smaller.

Compared with GPT-2, the XLM-RoBERTa tok/word value was approximately:

- Hindi: 81% lower
- Kannada: 89% lower
- Tamil: 90% lower

This shows that the original conclusion is highly dependent on the tokenizer
used.

## Tokens per grapheme

I also checked the XLM-RoBERTa token count against grapheme clusters.

| Language | XLM-R tokens | Graphemes | XLM-R tok/grapheme |
|---|---:|---:|---:|
| English | 28,995 | 125,194 | 0.232 |
| Hindi | 36,639 | 82,404 | 0.445 |
| Kannada | 39,625 | 86,177 | 0.460 |
| Tamil | 39,087 | 94,467 | 0.414 |

This gives another way to compare the tokenization results. It also shows that
the choice of denominator affects the comparison.

## What should be used for cost and routing?

I would not use tokens per whitespace-separated word or tokens per grapheme as
the final production cost metric.

For a production cost and routing decision, the most useful number is:

**actual model tokens processed per request, separated into prompt tokens and
generated tokens.**

This denominator is preferable because it directly measures the tokens that
the serving system actually processes. It therefore connects more directly to
model compute, KV-cache usage, latency, and serving cost.

For a fair cross-language comparison, the evaluation set should also contain
equivalent parallel requests or otherwise hold the underlying task/information
content approximately constant. A whitespace-separated word is not a
consistent unit of information across different languages and scripts.

Tokens per grapheme can be useful as a diagnostic metric for understanding
tokenization behavior, but it should not be treated as the production cost
metric.

## Routing implication

I would not route Indic-language traffic to a different model solely because
the GPT-2 fertility numbers are high.

The XLM-RoBERTa results show why. On this corpus, the XLM-RoBERTa tok/word
ratios relative to English are approximately:

- Hindi: 1.06x English
- Kannada: 1.83x English
- Tamil: 1.73x English

These are substantially different from the GPT-2 comparison.

However, a tokenizer with fewer tokens is not automatically the best production
choice. Model quality, latency, memory usage, and serving behavior also need
to be considered.

Before making a production routing decision, I would evaluate the candidate
model/tokenizer combinations using requests that are representative of actual
production traffic and measure the real prompt and generation token counts per
request.

## Conclusion

The original statement that Hindi is about 6x worse than English is valid for
the specific GPT-2 tok/word measurement that was performed, but it should not
be treated as a general serving-cost number.

The larger FLORES-derived evaluation and the XLM-RoBERTa comparison show that
the observed language gap depends heavily on the tokenizer.

For production cost and routing decisions, I would therefore use actual model
tokens processed per request, with prompt and generated tokens tracked
separately when possible, and evaluate this metric on traffic that is
representative of the intended workload.

The tokenizer result should be treated as evidence about tokenization behavior,
not by itself as evidence that one language requires a particular routing
policy.

