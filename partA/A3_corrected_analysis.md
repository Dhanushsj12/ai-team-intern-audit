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

The results changed a lot when the tokenizer was changed. With GPT-2, the
Indic-language values were much higher. With XLM-RoBERTa, the difference was
much smaller.

Compared with GPT-2, the XLM-RoBERTa tok/word value was approximately:

- Hindi: 81% lower
- Kannada: 89% lower
- Tamil: 90% lower

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

I would not use tokens per word or tokens per grapheme as the final production
cost number.

For production, I would check the actual number of model tokens processed for
each request. Prompt tokens and generated tokens should be tracked separately
when possible.

This is more useful for cost and capacity planning because it measures the
tokens that the model actually processes.

The tokenizer with fewer tokens is not automatically the best choice. Model
quality, latency and the actual serving setup also need to be checked.

## Conclusion

The original statement that Hindi is about 6x worse than English is based on
the GPT-2 result. It should not be treated as a general serving-cost number.

The XLM-RoBERTa results were much closer:

- Hindi: 1.06x English
- Kannada: 1.83x English
- Tamil: 1.73x English

So the large GPT-2 difference depends heavily on the tokenizer being used.

Before making a production routing decision, I would test the candidate models
with requests that are closer to actual production traffic and compare their
real tokens per request.