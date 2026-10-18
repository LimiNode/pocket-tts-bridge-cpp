# Text preparation profiles

Last reviewed: 2026-10-18

Text preparation is part of the model contract. It must be explicit, testable,
and independent from the worker transport.

## Why a profile is required

The candidate Russian models use different conventions:

- xVibe material accepts forms such as з+амок and a combining acute accent;
- Shtorm material recommends U+0301 after the stressed vowel and documents
  restrictions around Latin text and numbers;
- the amekhrishvili example expects prepared text with accents and expanded
  numbers.

These conventions cannot be merged into one silent normalization pass. A
profile tells the backend exactly which transformations were applied and prevents
a second accenting stage from rewriting an already prepared string.

## Profile shape

The implementation may use a C++ struct, a protocol object, or another
equivalent representation, but it must preserve these concepts:

| Field | Meaning |
| --- | --- |
| profile ID and revision | stable identity of the preparation rules |
| tokenizer family | tokenizer assumptions and vocabulary |
| accent mode | preserve, accept plus notation, accept U+0301, or delegate |
| number policy | keep digits, expand numbers, or reject unsupported forms |
| Latin policy | preserve, transliterate, spell out, or reject |
| punctuation and pause policy | symbols that become pauses or are removed |
| maximum input policy | hard limit, chunking, or sentence segmentation |
| provenance | whether text was raw, prepared, or supplied by the caller |
| locale | language and script expectations |

## Pipeline

A deterministic preparation pipeline should have visible stages:

1. validate UTF-8 and reject malformed input;
2. normalize only the Unicode forms allowed by the selected profile;
3. protect technical tokens, URLs, identifiers, and explicit user accents;
4. apply the model-specific number, abbreviation, and Latin policy;
5. apply or validate accents according to the profile;
6. normalize punctuation and pauses;
7. enforce length and segmentation limits;
8. emit the prepared text plus profile ID and transformation diagnostics.

The bridge must not claim that a model understood a pronunciation because a
preprocessor merely accepted the input. Pronunciation behavior belongs in the
acceptance corpus.

## Required test categories

For every supported profile, include cases for:

- ambiguous Russian homographs;
- ё versus е;
- U+0301 combining accents and plus-notation accents;
- cardinal and ordinal numbers, years, decimals, percentages, and units;
- Latin words, product names, and mixed-script identifiers;
- abbreviations and initials;
- punctuation, repeated punctuation, quotes, and brackets;
- explicit pauses and long sentences;
- already prepared input;
- malformed UTF-8 or unsupported control characters.

At least one test must prove that an explicitly prepared accent is not silently
replaced by an automatic accenting pass.

## Interface rule

The bridge should accept either raw text plus a requested profile or already
prepared text plus an explicit prepared marker. It must never infer that a
string is safe to reprocess from the presence of a combining mark alone.
Profile mismatch is a diagnostic error or an explicit conversion step.
