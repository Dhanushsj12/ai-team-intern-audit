# A2 — Tokenizer Script and Metric Audit

## Summary

I checked `fertility.py` and compared its line-level fertility calculation
with a corpus-level calculation using the same GPT-2 tokenizer and the same
corpus files.

There are two important metric issues:

1. The script calculates fertility separately for each line and then takes
   the arithmetic mean. This is not the same as total tokens divided by total
   words across the corpus.
2. The script uses whitespace-separated words as the denominator. This is a
   simple metric, but it is not a consistent measure of linguistic content
   across different languages and scripts.

I also checked the lowercasing step. The script lowercases every language
before tokenization, so the transformation is applied consistently. I
therefore do not treat lowercasing as a demonstrated bug.

---

## Finding 1 — Per-line averaging

In `fertility.py`, fertility is calculated separately for every non-empty
line:

```python
per_line_fertility.append(len(tokens) / len(words))