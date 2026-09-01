# Part A4 — Recommendation Memo

## What changed after checking the results

The original report says that Hindi has about 6x the fertility of English with
the GPT-2 tokenizer. That result is true for the test we ran, but I would not
use it directly to estimate serving cost.

Using the larger FLORES-derived corpus, the GPT-2 results were:

| Language | GPT-2 tok/word |
|---|---:|
| English | 1.28 |
| Hindi | 7.83 |
| Kannada | 22.95 |
| Tamil | 24.87 |

I then checked the same languages with XLM-RoBERTa:

| Language | XLM-R tok/word |
|---|---:|
| English | 1.38 |
| Hindi | 1.49 |
| Kannada | 2.57 |
| Tamil | 2.42 |

The difference is large. Hindi, Kannada, and Tamil are much closer to English
with XLM-RoBERTa than they are with GPT-2. This means the original result
depends heavily on the tokenizer being used.

## Recommendation

I would not route Indic traffic just because the GPT-2 numbers are high.

Before making a production routing decision, I would test the candidate
model/tokenizer combinations with requests that are similar to real traffic.
The main thing I would compare is the actual number of model tokens processed
per request. I would also check quality and latency before choosing a model.

XLM-RoBERTa gives a good indication that using a tokenizer designed for
multiple languages can reduce the tokenization gap. However, the fertility
numbers alone are not enough to say that XLM-RoBERTa is the best production
choice.

## Main limitation

The FLORES-derived corpus is much larger than the original small test, so it
gives us a better measurement. But it is still not the same as production
traffic.

The sentences and domain may be different from the requests the system will
actually receive. Because of this, these results cannot by themselves tell us
the production cost or the quality of responses.

## Metric I would monitor in production

I would monitor:

**actual model tokens processed per request, separated into prompt and
generation tokens by language.**

This is more directly connected to serving cost than tokens per whitespace
word.

If the number of tokens per request for an Indic language stays much higher
than expected, that would be a useful signal to investigate the tokenizer,
model choice, or routing decision.