# A2 — Tokenizer Script and Metric Audit

## Summary

I checked the fertility script and compared its results with some additional
calculations.

I found two things worth noting:

1. The script takes the fertility value for each line and then averages those
   values. This is slightly different from calculating total tokens divided by
   total words for the whole corpus.

2. The script uses whitespace-separated words as the denominator. This makes
   comparisons between languages less straightforward because the number of
   whitespace words is different across the languages.

I also checked the lowercasing step. It changes the text before tokenization,
but it is done for every language, so I did not treat it as a bug.

---

## Finding 1 — Per-line averaging

In `fertility.py`, the script does this:

```python
per_line_fertility.append(len(tokens) / len(words))

...

return sum(per_line_fertility) / n