# Part A4 — Recommendation Memo

## Corrected headline

The original report's claim that Hindi has approximately 6x the tokenizer
fertility of English is not a robust basis for a serving-cost decision.

On the FLORES-derived corpus, GPT-2 produced:

| Language | GPT-2 tok/word |
|---|---:|
| English | 1.28 |
| Hindi | 7.83 |
| Kannada | 22.95 |
| Tamil | 24.87 |

However, XLM-RoBERTa produced substantially lower Indic fertility:

| Language | XLM-R tok/word |
|---|---:|
| English | 1.42 |
| Hindi | 1.50 |
| Kannada | 2.60 |
| Tamil | 2.45 |

Thus the large Indic penalty is strongly tokenizer-dependent. It should not be
converted directly into a universal 6x Hindi serving-cost assumption.

## Routing recommendation

Do not route all Indic traffic based solely on the original GPT-2 fertility
numbers.

Instead, evaluate a multilingual/Indic-aware tokenizer and model using
representative production requests. Routing should ultimately be based on
actual model tokens processed per request, together with quality and latency.

XLM-RoBERTa is strong evidence that an Indic-aware tokenizer can substantially
reduce the tokenization gap, but tokenizer fertility alone does not establish
that it is the best production model.

## Biggest caveat

The FLORES-derived evaluation set is much better than the original ~10-sentence
smoke test, but it is still a limited benchmark. Its domain and sentence
distribution may not represent our production traffic. The analysis therefore
cannot establish production serving cost or conversational quality.

## Production metric

The single metric I would monitor is:

**actual model tokens processed per request, split into prompt and generation
tokens by language.**

This directly connects the tokenizer analysis to serving capacity and cost.
A persistent unexpected increase in tokens/request for an Indic language would
be an early signal that the routing assumption is wrong.