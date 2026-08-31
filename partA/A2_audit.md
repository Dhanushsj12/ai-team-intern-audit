# A2 — Tokenizer Script and Metric Audit

## Summary

I audited `fertility.py` and checked its fertility calculation against
corpus-level calculations using the same GPT-2 tokenizer and the same corpus
files.

I found two metric issues that affect how the original result should be
interpreted:

1. Fertility is calculated separately for each line and then averaged. This
   is different from calculating total tokens divided by total words across
   the complete corpus.
2. The denominator is based on whitespace-separated words. This is easy to
   compute, but a whitespace-separated word is not a consistent unit of
   linguistic content across different languages and scripts.

I also checked the lowercasing step. It is applied consistently to every
language before tokenization, so I do not treat it as a demonstrated bug.

---

## Finding 1 — Per-line averaging

In `fertility.py`, the script calculates fertility independently for every
non-empty line:

```python
per_line_fertility.append(len(tokens) / len(words))
```

It then returns the arithmetic mean of the per-line fertility values:

```python
return sum(per_line_fertility) / n
```

This gives every non-empty line equal weight, regardless of how many words
that line contains.

The correct corpus-level alternative is:

```text
total tokens / total words
```

These two calculations are not generally identical because the first averages
ratios while the second calculates one ratio from the complete corpus.

### Direction and impact

The effect of per-line averaging depends on the distribution of line lengths
and token/word ratios. Therefore, it cannot be assumed that this method always
overestimates or always underestimates corpus-level fertility.

The important correction is that the script's result must be described as a
**per-line average**, not as a corpus-level tokens-per-word ratio.

---

## Finding 2 — Whitespace-separated word denominator

The script defines words using:

```python
words = line.split()
```

Therefore the denominator is the number of whitespace-separated words.

This is reproducible and simple, but whitespace-separated words are not
necessarily a consistent unit of linguistic content across languages and
scripts.

### Direction and impact

The denominator choice can change the apparent cross-language fertility gap.
A higher tokens-per-word value therefore does not by itself prove that a
language will cost the same multiple more to serve.

The metric is useful as a tokenizer diagnostic, but it should not be treated
as a direct production serving-cost estimate.

---

## Lowercasing check — not a demonstrated bug

The script contains:

```python
line = line.lower()
```

This transformation is applied consistently to every language before
tokenization.

Therefore, although lowercasing can affect tokenization, it is not an
inconsistent language-specific operation in this script.

I do not count lowercasing as a demonstrated flaw.

---

## Conclusion

The original GPT-2 fertility result is useful for describing tokenizer
behavior, but it has important metric limitations:

1. Per-line fertility values are averaged rather than calculating one
   corpus-level tokens-per-word ratio.
2. Whitespace-separated words are not a consistent cross-language unit of
   linguistic content.

The lowercasing step was checked and is applied consistently, so it is not
treated as a demonstrated bug.

For production decisions, actual model tokens processed per request should be
measured, with prompt and generated tokens tracked separately where possible.
Cross-language comparisons should use representative and approximately
equivalent workloads rather than relying on whitespace-word fertility alone.