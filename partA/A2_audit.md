# A2 — fertility.py Audit

## 1. Conceptual problem: tokens per whitespace word

The script reports tokenizer fertility as tokens per whitespace-delimited word.
This denominator is not language-neutral because whitespace word counts depend on
the writing system and tokenization conventions.

On the FLORES-derived corpus, the results were:

| Language | Tokens | Whitespace words | Graphemes | Tok/word | Tok/grapheme |
|---|---:|---:|---:|---:|---:|
| English | 26,696 | 20,954 | 125,194 | 1.274 | 0.213 |
| Hindi | 191,842 | 24,607 | 82,404 | 7.796 | 2.328 |
| Kannada | 349,802 | 15,430 | 86,177 | 22.670 | 4.059 |
| Tamil | 397,189 | 16,134 | 94,467 | 24.618 | 4.205 |

The denominator therefore changes the apparent cross-language comparison
substantially. Grapheme-normalized measurements are useful as a diagnostic,
but neither metric alone directly represents serving cost.

## 2. Code issue: split(" ")

The script uses `line.split(" ")` rather than `line.split()`.

Measured totals:

| Language | split(" ") | split() | Difference |
|---|---:|---:|---:|
| English | 20,955 | 20,954 | +1 |
| Hindi | 24,616 | 24,607 | +9 |

This is a real but small distortion on this corpus. It should not be treated as
the main source of the reported cross-language gap.

## 3. Language-dependent effect of lowercasing

The script lowercases every line before tokenization.

| Language | Original tokens | Lowercase tokens | Change |
|---|---:|---:|---:|
| English | 25,741 | 26,696 | +955 (+3.71%) |
| Hindi | 191,589 | 191,603 | +14 (+0.007%) |
| Kannada | 349,823 | 349,853 | +30 (+0.009%) |
| Tamil | 397,169 | 397,195 | +26 (+0.007%) |

Lowercasing therefore changes GPT-2 token counts substantially for English but
only minimally for the Indic languages tested. It introduces a language-dependent
preprocessing effect.

## 4. NFC normalization

NFC normalization also changes token counts in the corpus:

| Language | Raw tokens | NFC tokens | Change |
|---|---:|---:|---:|
| English | 25,741 | 25,741 | 0 |
| Hindi | 191,589 | 191,828 | +239 (+0.125%) |
| Kannada | 349,823 | 349,772 | -51 (-0.015%) |
| Tamil | 397,169 | 397,163 | -6 (-0.002%) |

The effect is measurable but small relative to the large cross-language
differences.

## 5. Suspicious but harmless

`random.seed(1337)` looks suspicious because the script contains no random
sampling or other use of the `random` module. It therefore has no effect on
the reported results. It is unnecessary/dead code, but it is not a measurement
bug.