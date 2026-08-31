# Notebook

## 31 Aug 2026

### Part A

- Checked the starter kit files.
- Created the Part A corpus folder.
- Downloaded and extracted the FLORES200 dataset.
- Took the English, Hindi, Kannada and Tamil dev files.
- Ran `fertility.py` with GPT-2.

GPT-2 output:

| Language | tok/word |
|---|---:|
| eng | 1.28 |
| hin | 7.83 |
| kan | 22.95 |
| tam | 24.87 |

The difference between English and the three Indic languages was very large.

### Checking the numbers

I checked the number of whitespace words, graphemes and tokens for each
language.

I also checked whether lowercasing and NFC normalization changed the token
counts.

For lowercasing, English changed from 25,741 tokens to 26,696 tokens. The
Indic languages changed only by a very small amount.

I also compared the script's fertility calculation with total tokens divided
by total words. The difference was small:

- English: 0.671%
- Hindi: 0.381%
- Kannada: 1.215%
- Tamil: 1.010%

So the per-line averaging is a real difference, but it does not explain the
large Indic results.

### XLM-R check

Ran the same corpus with XLM-RoBERTa.

| Language | tok/word |
|---|---:|
| eng | 1.42 |
| hin | 1.50 |
| kan | 2.60 |
| tam | 2.45 |

This was much different from the GPT-2 results, especially for Hindi,
Kannada and Tamil.

I also checked tokens per grapheme:

| Language | tok/grapheme |
|---|---:|
| eng | 0.232 |
| hin | 0.445 |
| kan | 0.460 |
| tam | 0.414 |

This showed that the result changes depending on both the tokenizer and the
denominator.

### Problems while testing

One command failed because I passed the tiktoken `Encoding` object instead of
its `encode` method. I corrected the command and ran it again.

Another one-line command gave a division-by-zero error. I replaced it with a
simpler command and got the required word/grapheme counts.

### Current conclusion

I would not use the original GPT-2 Hindi 6x number as a production cost
multiplier.

The next step for a real routing decision would be to check actual model
tokens per request using representative production traffic.