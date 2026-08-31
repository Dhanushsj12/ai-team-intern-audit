# Corpus Construction

## Dataset

I used the FLORES-200 dataset and selected the English, Hindi,
Kannada, and Tamil development sets.

Languages:
- English (eng_Latn)
- Hindi (hin_Deva)
- Kannada (kan_Knda)
- Tamil (tam_Taml)

The selected corpus contains parallel multilingual sentences.

## Preprocessing

The selected language files were copied into `partA/corpus/` and
renamed to `eng.txt`, `hin.txt`, `kan.txt`, and `tam.txt`.

The existing fertility script performs NFC normalization and
lowercasing before tokenization.

No additional linguistic preprocessing was applied.

## Initial measurement

Using the existing `fertility.py` with the GPT-2 tokenizer:

| Language | Fertility (tok/word) | Tok/char |
|---|---:|---:|
| English | 1.28 | 0.215 |
| Hindi | 7.83 | 1.528 |
| Kannada | 22.95 | 2.655 |
| Tamil | 24.87 | 2.717 |

These numbers are an initial measurement, not the final conclusion.

## Limitations

The FLORES development corpus is still a limited evaluation set and
does not represent all real production traffic. Its sentences are
primarily translated/curated evaluation text rather than naturally
occurring conversational requests. Therefore these measurements may
not predict fertility for every domain, writing style, or production
workload.