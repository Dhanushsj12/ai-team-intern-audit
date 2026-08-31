# Corpus Construction

## Dataset

I used the FLORES-200 dataset and selected the development files for four
languages:

- English (`eng_Latn`)
- Hindi (`hin_Deva`)
- Kannada (`kan_Knda`)
- Tamil (`tam_Taml`)

The four files contain parallel sentences from the same development data.

## Preprocessing

I copied the selected files into:

`partA/corpus/`

and renamed them as:

- `eng.txt`
- `hin.txt`
- `kan.txt`
- `tam.txt`

The existing `fertility.py` script removes empty lines, applies NFC
normalization and converts the text to lowercase before tokenization.

I did not apply any other language-specific preprocessing.

## Initial GPT-2 measurement

I first ran the existing `fertility.py` script using the GPT-2 tokenizer.

The output was:

| Language | Fertility (tok/word) | Tok/char |
|---|---:|---:|
| English | 1.28 | 0.215 |
| Hindi | 7.83 | 1.528 |
| Kannada | 22.95 | 2.655 |
| Tamil | 24.87 | 2.717 |

These were the initial numbers I used for the later checks.

## Limitations

The FLORES development data is larger than the small initial test, but it is
still an evaluation dataset.

It does not contain every type of text that could appear in production.
For example, real requests may have different domains, writing styles and
sentence lengths.

Because of this, these results should not be treated as an exact prediction
of production token usage or serving cost.