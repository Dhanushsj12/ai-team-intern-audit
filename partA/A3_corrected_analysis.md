# A3 — Corrected Analysis

## Objective

The original report compares languages using GPT-2 tokenizer fertility measured as
tokens per whitespace-delimited word. This section repeats the comparison on the
larger FLORES-derived corpus using two tokenizers and two denominators.

## Corpus

The evaluation uses FLORES-derived development data for four languages:

- English
- Hindi
- Kannada
- Tamil

The corpus contains 1,012 lines per language in the selected development split.
The same parallel sentence positions are used across languages.

Preprocessing follows the existing script where applicable: UTF-8 input, removal
of empty lines, NFC normalization, and lowercasing for the fertility measurements.

## Tokenizer comparison

### Tokens per whitespace word

| Language | GPT-2 | XLM-RoBERTa |
|---|---:|---:|
| English | 1.28 | 1.42 |
| Hindi | 7.83 | 1.50 |
| Kannada | 22.95 | 2.60 |
| Tamil | 24.87 | 2.45 |

GPT-2 shows a very large Indic-language penalty, while XLM-RoBERTa substantially
reduces the token count for Hindi, Kannada, and Tamil.

Relative to GPT-2, the XLM-RoBERTa tokenizer reduces tok/word by approximately:

- Hindi: 7.83 -> 1.50, about 81% lower
- Kannada: 22.95 -> 2.60, about 89% lower
- Tamil: 24.87 -> 2.45, about 90% lower

## Second denominator: grapheme clusters

For XLM-RoBERTa, token counts and grapheme counts were measured independently:

| Language | XLM-R tokens | Graphemes | XLM-R tok/grapheme |
|---|---:|---:|---:|
| English | 28,995 | 125,194 | 0.232 |
| Hindi | 36,639 | 82,404 | 0.445 |
| Kannada | 39,625 | 86,177 | 0.460 |
| Tamil | 39,087 | 94,467 | 0.414 |

Grapheme-normalized measurements confirm that the tokenizer behavior differs
across languages, but they do not directly represent serving cost.

## Which number should drive routing and cost?

The single number that should drive a routing-and-cost decision is:

**model tokens processed per production request**, measured separately for prompt
and generation tokens where possible.

The reason is that serving cost and capacity are determined by the number of
tokens actually processed by the model. Tok/word and tok/grapheme are useful
diagnostic measures for understanding tokenizer behavior, but their denominators
are language-dependent and do not directly correspond to the workload that the
serving system must execute.

For routing decisions, I would therefore measure representative production
requests for each language and compare their actual token counts under the
candidate tokenizer/model. The tokenizer with lower production token counts
should generally have lower token-processing cost, subject to model quality and
latency constraints.

## Corrected conclusion

The original headline that Hindi is approximately 6x worse than English is not
a robust basis for a serving-cost decision. It is strongly dependent on the
choice of tokenizer and denominator.

On the larger corpus, GPT-2 produces extremely high fertility for the Indic
languages, whereas XLM-RoBERTa produces much smaller gaps:

- Hindi: 1.06x English
- Kannada: 1.83x English
- Tamil: 1.73x English

This demonstrates that the large GPT-2 penalty is substantially tokenizer
dependent rather than a universal property of Indic scripts.

The appropriate next step is therefore not to budget a universal 6x Hindi
serving penalty, but to benchmark candidate tokenizers/models using
representative production workloads and actual tokens processed per request.