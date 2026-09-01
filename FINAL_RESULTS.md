````markdown

\# Part A — Final Results



\## A1 — GPT-2 Tokenizer Fertility



The fertility benchmark was run using `fertility.py` with the GPT-2 tokenizer on the English, Hindi, Kannada and Tamil corpora.



\### Command



```powershell

python fertility.py --corpus eng=partA/corpus/eng.txt --corpus hin=partA/corpus/hin.txt --corpus kan=partA/corpus/kan.txt --corpus tam=partA/corpus/tam.txt --tokenizer gpt2

````



\### Results



| Language | Fertility (tok/word) | Tokens/char |

| -------- | -------------------: | ----------: |

| English  |                 1.28 |       0.215 |

| Hindi    |                 7.83 |       1.528 |

| Kannada  |                22.95 |       2.655 |

| Tamil    |                24.87 |       2.717 |



Relative to English:



| Language | Relative fertility |

| -------- | -----------------: |

| Hindi    |              6.10x |

| Kannada  |             17.89x |

| Tamil    |             19.39x |



The GPT-2 results show substantially higher tokenization cost for the three Indic-language corpora.



\---



\## A2 — Script Average vs Corpus Ratio



The script averages fertility across individual lines. I compared this with total tokens divided by total whitespace-separated words.



| Language | Script average | Corpus ratio | Delta |

| -------- | -------------: | -----------: | ----: |

| English  |          1.283 |        1.274 | 0.009 |

| Hindi    |          7.826 |        7.796 | 0.030 |

| Kannada  |         22.946 |       22.670 | 0.275 |

| Tamil    |         24.867 |       24.618 | 0.249 |



The difference is small compared with the large English/Indic gap, so per-line averaging does not explain the large Indic results.



\---



\## A3 — Lowercasing Check



GPT-2 token counts before and after lowercasing:



| Language |  Before |   After | Delta | Percentage |

| -------- | ------: | ------: | ----: | ---------: |

| English  |  25,741 |  26,696 |  +955 |     +3.71% |

| Hindi    | 191,828 | 191,842 |   +14 |     +0.01% |

| Kannada  | 349,772 | 349,802 |   +30 |     +0.01% |

| Tamil    | 397,163 | 397,189 |   +26 |     +0.01% |



Lowercasing has a noticeable effect on English but a negligible effect on the three Indic languages. Therefore, lowercasing does not explain the large fertility gap.



\---



\## A4 — Tokens per Character Check



| Language | Script TPC | Corpus TPC |   Delta |

| -------- | ---------: | ---------: | ------: |

| English  |     0.2152 |     0.2132 | +0.0019 |

| Hindi    |     1.5276 |     1.5287 | -0.0011 |

| Kannada  |     2.6555 |     2.6551 | +0.0004 |

| Tamil    |     2.7171 |     2.7181 | -0.0011 |



The differences are very small, confirming that the large language differences are not caused by the per-line averaging method.



\---



\## A5 — XLM-RoBERTa Cross-Check



The same corpus was also evaluated using XLM-RoBERTa.



| Language | Fertility (tok/word) |

| -------- | -------------------: |

| English  |                 1.42 |

| Hindi    |                 1.50 |

| Kannada  |                 2.60 |

| Tamil    |                 2.45 |



\### XLM-R Tokens per Grapheme



| Language | Tokens/grapheme |

| -------- | --------------: |

| English  |           0.232 |

| Hindi    |           0.445 |

| Kannada  |           0.460 |

| Tamil    |           0.414 |



The XLM-R results are substantially different from GPT-2, especially for Hindi, Kannada and Tamil. This indicates that tokenizer choice has a major effect on measured fertility.



\---



\## A6 — Corpus Integrity Check



The corpus files were checked directly using Python UTF-8 reading.



```python

from pathlib import Path



print(Path('partA/corpus/hin.txt').read\_text(encoding='utf-8')\[:300])

print(Path('partA/corpus/kan.txt').read\_text(encoding='utf-8')\[:300])

print(Path('partA/corpus/tam.txt').read\_text(encoding='utf-8')\[:300])

```



The Python UTF-8 check confirmed that the Hindi, Kannada and Tamil files contain the expected Unicode text.



The Indic text appearing as mojibake in PowerShell is a terminal display/encoding issue and does not by itself indicate corrupted files.



\---



\## A7 — Reproducibility Check



The script contains:



```python

random.seed(1337)

```



However, the current fertility calculation does not use random sampling. Changing the seed therefore does not change the result.



For English:



```text

with seed: (1.2825789256765674, 0.21515885298271298)

after changing seed: (1.2825789256765674, 0.21515885298271298)

```



The result is unchanged, confirming that the current calculation is deterministic for a given corpus and tokenizer.



\---



\# Final Conclusion



GPT-2 produces substantially higher fertility for Hindi, Kannada and Tamil than for English:



\* English: 1.28 tok/word

\* Hindi: 7.83 tok/word

\* Kannada: 22.95 tok/word

\* Tamil: 24.87 tok/word



However, the XLM-RoBERTa cross-check produces much lower values:



\* English: 1.42 tok/word

\* Hindi: 1.50 tok/word

\* Kannada: 2.60 tok/word

\* Tamil: 2.45 tok/word



Therefore, the GPT-2 fertility numbers should not be used directly as a production inference-cost multiplier.



For a real routing or cost decision, the next step is to measure actual tokens per request using representative production traffic and the tokenizer/model that will actually serve that traffic.



\---



\# Reproduction



From the repository root:



```powershell

python fertility.py --corpus eng=partA/corpus/eng.txt --corpus hin=partA/corpus/hin.txt --corpus kan=partA/corpus/kan.txt --corpus tam=partA/corpus/tam.txt --tokenizer gpt2

```



Expected output:



```text

tokenizer: gpt2

lang      fertility (tok/word)    tok/char

\------------------------------------------

eng                       1.28 0.215

hin                       7.83 1.528

kan                      22.95 2.655

tam                      24.87 2.717



hin is 6.10x the fertility of eng (worse tokenization)

kan is 17.89x the fertility of eng (worse tokenization)

tam is 19.39x the fertility of eng (worse tokenization)

```



\## Related Files



\* `partA/A2\_audit.md`

\* `partA/A3\_corrected\_analysis.md`

\* `partA/corpus\_prep.md`

\* `partA/memo.md`

\* `NOTEBOOK.md`

\* `REPORT\_v0.md`



```

```



