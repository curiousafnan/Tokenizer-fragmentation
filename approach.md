# Day 50 — Approach

## Metrics

| Metric | Definition | Note |
|---|---|---|
| **Fertility** | tokens ÷ whitespace words, averaged over sentences | Depends on how words are counted |
| **Parity** | tokens(language) ÷ tokens(English), per sentence, averaged | **No word counting at all** — the cleaner metric |
| **Tax** | fertility(language) ÷ fertility(English), same tokenizer | What the paper reports |
| **`[UNK]` rate** | `[UNK]` ÷ all tokens (mBERT) | Whether the language can be represented at all |

Both fertility and parity are **means of per-sentence ratios**. This is what
reproduces the published numbers, so it is kept, and stated here.

**Why parity matters:** whitespace word counts are unreliable for Urdu, which has
both space omission and space insertion. Parity compares token counts for the same
sentence directly, so the word-counting decision disappears.

**Counting rule:** content tokens only — `add_special_tokens=False` for Hugging Face
tokenizers. tiktoken's `encode` adds none for ordinary text.

## Validation

**English fertility on cl100k: 1.24** — inside the expected 1.2–1.3.

**Urdu tax reproduces the published numbers:**

| Tokenizer | Ours | Published |
|---|---|---|
| cl100k | 3.45× | 3.45× |
| o200k | 1.32× | 1.31× |
| GPT-2 | 4.90× | 4.93× |
| Qwen2.5 | 2.50× | 2.50× |
| mBERT | 1.34× | 1.34× |
| XLM-R | 0.99× | 0.98× |

All six within about 1%. The pipeline is sound, so the Pashto numbers can be trusted.

## Results

### Parity (tokens vs English) with 95% bootstrap intervals

2,000 paired resamples — the same sentence ids for Urdu and Pashto in every resample.

| Tokenizer | Urdu | Pashto | Urdu − Pashto (95% CI) |
|---|---|---|---|
| GPT-2 | 6.35 | 5.49 | +0.87 (0.81 to 0.93) |
| cl100k | 4.46 | 3.90 | +0.56 (0.51 to 0.61) |
| Qwen2.5 | 3.24 | 2.93 | +0.30 (0.27 to 0.34) |
| mBERT ⚠️ | 1.73 | 1.68 | +0.05 (0.03 to 0.07) |
| o200k | 1.68 | 1.78 | −0.10 (−0.11 to −0.08) |
| XLM-R | 1.26 | 1.42 | −0.16 (−0.18 to −0.15) |

Every gap is statistically real — no interval crosses zero.

⚠️ **mBERT's Pashto number is not valid:** 16.68% of its Pashto tokens are `[UNK]`
(English 0.14%, Urdu 0.13%). Letters such as ځ, ښ, ږ are missing from its vocabulary,
so whole words collapse into one `[UNK]`, which *lowers* the token count. The
bootstrap shows the number is stable, not that it measures anything.

## Why is Pashto cheaper than Urdu on English-centric tokenizers?

Two hypotheses, each tested.

### B — English words left in Latin script: ruled out as the main cause

Pashto translations keep far more English words in Latin letters (`Stanford`,
`Inkject Printer`):

| | Latin letters | % of characters | Words with Latin letters |
|---|---|---|---|
| Urdu | 364 | 0.29% | 86 |
| Pashto | 3,850 | 3.15% | 637 |

**Test:** keep only the 726 sentences where neither Urdu nor Pashto contains a Latin
letter. On cl100k, Pashto ÷ Urdu was **0.88** — the gap did not close.

### A — Urdu-specific letters: confirmed

Urdu writes four common letters with its own code points, where Arabic, Persian and
Pashto use shared ones. **Test:** on the 726 sentences, replace them (10,071 letters):

| Urdu | → | Shared form |
|---|---|---|
| ہ, ھ | → | ه |
| ے | → | ی |
| ں | → | ن |

| | Urdu original | Urdu swapped | Pashto |
|---|---|---|---|
| **cl100k** | 4.57× | **3.83×** | 3.95× |
| **o200k** | 1.69× | **1.82×** | 1.76× |

- **cl100k:** swapping four letters makes Urdu cheaper than Pashto. cl100k never
  learned Urdu's own letters and reads Urdu as misspelled Arabic/Persian.
- **o200k:** the same swap makes Urdu **more expensive**. o200k learned real Urdu
  words with those letters; swapping breaks the matches.

The same change moving the two tokenizers in **opposite directions** is what rules
out coincidence.

Scope: supported on FLORES dev, two tokenizers. Not yet tested on other corpora or on
GPT-2/Qwen.