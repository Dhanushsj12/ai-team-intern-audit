# A2 — Tokenizer Script and Metric Audit

## Summary

I audited `fertility.py` and checked its fertility calculation against corpus-level calculations using the same GPT-2 tokenizer and the same corpus files.

I found two metric issues that affect how the original result should be interpreted:

1. Fertility is calculated separately for each line and then averaged. This is different from calculating total tokens divided by total words across the complete corpus.
2. The denominator is based on whitespace-separated words. This is easy to compute, but a whitespace-separated word is not a consistent unit of linguistic content across different languages and scripts.

I also checked the lowercasing step. It is applied consistently to every language before tokenization, so I do not treat it as a demonstrated bug.

---

## Finding 1 — Per-line averaging

In `fertility.py`, the script calculates fertility independently for every non-empty line:

```python
per_line_fertility.append(len(tokens) / len(words))
```

It then returns the arithmetic mean of the per-line fertility values:

```python
return sum(per_line_fertility) / n
```

This gives every non-empty line equal weight, regardless of how many words that line contains.

The correct corpus-level alternative is:

```text
total tokens / total words
```

These two calculations are not generally identical because the first averages ratios while the second calculates one ratio from the complete corpus.

### Direction and impact

The effect of per-line averaging depends on the distribution of line lengths and token/word ratios. Therefore, it cannot be assumed that this method always overestimates or always underestimates corpus-level fertility.

The important correction is that the script's result must be described as a **per-line average**, not as a corpus-level tokens-per-word ratio.

### Measured effect

I compared the script's per-line arithmetic mean against the corpus-level calculation `total tokens / total words` on the same four corpus files and using the same GPT-2 tokenizer.

| Language | Script result | Corpus-level result | Difference |
| -------- | ------------: | ------------------: | ---------: |
| English  |      1.282579 |            1.274029 |     0.671% |
| Hindi    |      7.825973 |            7.796237 |     0.381% |
| Kannada  |     22.945631 |           22.670253 |     1.215% |
| Tamil    |     24.866899 |           24.618136 |     1.010% |

The corpus-level calculation was performed as:

```text
total tokens / total whitespace-separated words
```

The corpus-level values are lower than the script's per-line averages for all four languages in this experiment, by **0.381% to 1.215%**.

Therefore, for this corpus, the per-line averaging produces a small upward distortion. However, the direction and magnitude are not guaranteed to be the same for every corpus because they depend on line length and the distribution of fertility values across lines.

---

## Finding 2 — Whitespace-separated word denominator

The script defines words using:

```python
words = line.split()
```

Therefore, the denominator is the number of whitespace-separated words.

This is reproducible and simple, but whitespace-separated words are not necessarily a consistent unit of linguistic content across languages and scripts.

### Direction and impact

The denominator choice can change the apparent cross-language fertility gap.

A higher tokens-per-word value therefore does not by itself prove that a language will cost the same multiple more to serve.

For example, using the same corpus, the relative comparison between languages changes depending on whether the denominator is whitespace-separated words or characters.

For Kannada:

```text
Token/word relative to English:
22.6703 / 1.2740 = 17.79x

Token/character relative to English:
2.6551 / 0.2132 = 12.45x
```

For Tamil:

```text
Token/word relative to English:
24.6181 / 1.2740 = 19.32x

Token/character relative to English:
2.7181 / 0.2132 = 12.75x
```

This demonstrates that the choice of denominator materially affects the reported cross-language comparison.

The metric is therefore useful as a tokenizer diagnostic, but it should not be treated as a direct production serving-cost estimate.

### Measured corpus counts

Using the same GPT-2 tokenizer and corpus:
| Language | Tokens | Words | Tok/word | Characters | Tok/character |
|---|---:|---:|---:|---:|---:|
| English | 26,696 | 20,954 | 1.2740 | 125,194 | 0.2132 |
| Hindi | 191,842 | 24,607 | 7.7962 | 125,495 | 1.5287 |
| Kannada | 349,802 | 15,430 | 22.6703 | 131,749 | 2.6551 |
| Tamil | 397,189 | 16,134 | 24.6181 | 146,126 | 2.7181 |

These counts show that the underlying token count is fixed for a given tokenizer and corpus. Changing the denominator from whitespace-separated words to characters changes the reported ratio without changing the number of tokenizer output tokens.

---

## Lowercasing check — not a demonstrated bug

The script contains:

```python
line = line.lower()
```

This transformation is applied consistently to every language before tokenization.

Therefore, although lowercasing can affect tokenization, it is not an inconsistent language-specific operation in this script.

I do not count lowercasing as a demonstrated flaw.

### Measured effect

For English, the token count changed from **25,741 before lowercasing** to **26,696 after lowercasing**:

```text
(26,696 - 25,741) / 25,741 × 100 = 3.71%
```

This means lowercasing changed the English GPT-2 token count by approximately **3.71%** in this corpus.

This confirms that lowercasing can affect tokenizer output. However, because the transformation is applied consistently across all languages in the script, this observation alone does not establish a methodological bug.

---

## Conclusion

The original GPT-2 fertility result is useful for describing tokenizer behavior, but it has important metric limitations:

1. **Per-line fertility values are averaged** rather than calculating one corpus-level tokens-per-word ratio.
2. **Whitespace-separated words are not a consistent cross-language unit of linguistic content.**
3. **Changing the denominator can materially change the apparent cross-language fertility gap.**
4. **Lowercasing affects tokenizer output**, but it is applied consistently and therefore is not treated as a demonstrated bug.

For this corpus, replacing the per-line arithmetic mean with the corpus-level calculation changes the reported result by **0.381% to 1.215%**, depending on language. The per-line method produces a small upward distortion in this particular experiment.

The denominator experiment also shows that cross-language comparisons depend substantially on how the denominator is defined. For example, Kannada appears **17.79×** English on a token/word basis but **12.45×** on a token/character basis. Tamil appears **19.32×** English on a token/word basis but **12.75×** on a token/character basis.

Therefore, fertility should be treated primarily as a **tokenizer diagnostic**, rather than as a direct estimate of production serving cost.

For production decisions, actual model tokens processed per request should be measured, with prompt and generated tokens tracked separately where possible.

Cross-language comparisons should use representative and approximately equivalent workloads rather than relying on whitespace-word fertility alone.
